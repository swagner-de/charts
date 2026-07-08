# samba

![Version: 0.1.0](https://img.shields.io/badge/Version-0.1.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 4.22.10](https://img.shields.io/badge/AppVersion-4.22.10-informational?style=flat-square)
Multi-user Samba SMB file server with per-share encryption and Time Machine support

## Features
- Multi-user setup: users declared in `values.users`, passwords injected from a
  Kubernetes Secret at `/run/secrets/<name>`
- One PVC per share, sized and storage-classed per share
- Optional per-share and global SMB encryption (`required` / `desired` / `off`)
- Optional per-share Time Machine support (`fruit:time machine = yes`) with
  configurable max size
- Uses [`ghcr.io/swagner-de/containers/samba`](https://github.com/swagner-de/containers/tree/main/apps/samba) —
  Alpine + `samba-server` with an entrypoint that reads `/config/users.conf` and
  `/run/secrets/<name>` at pod startup

## Install

```bash
helm install samba oci://ghcr.io/swagner-de/charts/samba
```

## Requirements

| Repository | Name | Version |
|------------|------|---------|
| https://bjw-s-labs.github.io/helm-charts/ | common | 5.0.1 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| existingPasswordsSecret | string | `"samba-passwords"` | Name of an existing Secret whose keys are user names from `users` and values are their SMB passwords. Mounted at `/run/secrets`; the entrypoint imports each key into passdb.tdb via `smbpasswd -a` on every pod start. When empty the chart creates an in-cluster Secret from `passwordsData` instead (intended for chart-testing / CI only). |
| passwordsData | object | {} | Data for the chart-managed passwords Secret when `existingPasswordsSecret` is empty. Do not use in production. |
| samba | object | `{"global":["server min protocol = SMB2","vfs objects = fruit streams_xattr","fruit:aapl = yes","fruit:metadata = stream","fruit:model = TimeCapsule8,119","fruit:nfs_aces = no","fruit:posix_rename = yes","fruit:zero_file_id = yes","fruit:wipe_intentionally_left_blank_rfork = yes","fruit:delete_empty_adfiles = yes","usershare allow guests = no","wide links = no","strict locking = yes","pam password change = no"],"hostsAllow":"127.0.0.0/8 10.0.0.0/8 172.16.0.0/12 192.168.0.0/16","logLevel":0,"serverString":"Samba on Kubernetes","workgroup":"WORKGROUP"}` | Global Samba server settings |
| samba.global | list | `["server min protocol = SMB2","vfs objects = fruit streams_xattr","fruit:aapl = yes","fruit:metadata = stream","fruit:model = TimeCapsule8,119","fruit:nfs_aces = no","fruit:posix_rename = yes","fruit:zero_file_id = yes","fruit:wipe_intentionally_left_blank_rfork = yes","fruit:delete_empty_adfiles = yes","usershare allow guests = no","wide links = no","strict locking = yes","pam password change = no"]` | Lines rendered verbatim under `[global]` in `smb.conf`. Overriding this replaces the entire list; copy the defaults before removing entries. Defaults enable macOS/Time Machine interoperability via the `fruit` VFS. |
| samba.hostsAllow | string | `"127.0.0.0/8 10.0.0.0/8 172.16.0.0/12 192.168.0.0/16"` | Space-separated CIDR list of permitted hosts |
| samba.logLevel | int | `0` | Samba log level (0-10) |
| samba.serverString | string | `"Samba on Kubernetes"` | Server description string |
| samba.workgroup | string | `"WORKGROUP"` | NT workgroup name |
| service | object | `{"main":{"externalTrafficPolicy":"Local","ipFamilies":["IPv4"],"ipFamilyPolicy":"SingleStack","type":"LoadBalancer"}}` | Kubernetes Service for smbd. The chart hardcodes only what's required for the pod to be reachable on SMB (`controller: main`, `primary: true`, and the `smb` port on 445) — everything below is a soft default that HelmRelease values can override (`type`, `externalTrafficPolicy`, IP families, and `annotations` for MetalLB IPs, external-dns hostnames, etc.). |
| shares | list | [] | Share definitions |
| users | list | [] | User accounts. Passwords are NOT here — they come from a Secret mounted at /run/secrets/<name> |

## macOS Time Machine setup

Time Machine over SMB on macOS Sequoia (15.x) has two known regressions that
block `backupd` from reading the network credential even when interactive
`smbutil` and Finder mount work correctly. Both need to be worked around on the
Mac before the first backup will run.

Symptoms in `log show --predicate 'process == "backupd" OR process == "NetAuthSysAgent"'`:

```
NetAuthSysAgent: ... Keychain: isKnownServer 0
NetAuthSysAgent: ... MechTypes: There are no user credentials to add to the MechType session
NetAuthSysAgent: ... NetFS: OpenSession failed 80
backupd: ... NAConnectToServerSync failed with error: 80 (Authentication error)
backupd: ... Backup failed: BACKUP_FAILED_AUTHENTICATION_ERROR (29)
```

### 1. Whitelist the server in `serverMarkers.plist`

Without this, `NetAuthSysAgent` reports `isKnownServer 0` and gates the
credential lookup.

```bash
sudo /usr/libexec/PlistBuddy -c 'Add :<samba-host> bool true' \
  '/private/var/root/Library/Group Containers/group.com.apple.NetworkAuthorization.ServerMarkers/serverMarkers.plist'
```

### 2. Register the destination via `tmutil`, not System Settings

The Time Machine settings pane writes a `Time Machine Network Password`
keychain entry whose ACL does not grant `backupd` read access on Sequoia. The
`-T` flag on `security add-internet-password` does not help either — the
runtime signature check refuses path-based ACLs for system daemons.

`tmutil setdestination -ap` writes the credential via TimeMachine's own APIs in
a form `backupd` and `NetAuthSysAgent` accept:

```bash
# If a broken destination was already added, remove it first
sudo tmutil removedestination <destination-id>            # see `tmutil destinationinfo`
sudo security delete-internet-password -s <samba-host> -a <user> \
  /Library/Keychains/System.keychain 2>/dev/null || true

sudo tmutil setdestination -ap 'smb://<user>:<password>@<samba-host>/<share>'
tmutil startbackup
```

Verify in the same `log show` command: `NetAuthSysAgent` should log
`Found MacOS NetAuthSysAgent ACL` and `Reference "ntlm:<user>@..." acquired for
the MechType`, followed by `backupd: ... Mounted 'smb://...' ...`.

### If you rename a share user

Renaming a share user invalidates the entries macOS stored under the old
account. Delete stale entries from **both** keychains and re-register the
destination with the new user:

```bash
sudo security delete-internet-password -s <samba-host> -a <old-user> \
  /Library/Keychains/System.keychain
security delete-internet-password -s <samba-host> -a <old-user> \
  ~/Library/Keychains/login.keychain-db 2>/dev/null || true
```

Then repeat step 2 with the new user. The existing on-disk sparsebundle is
reused as long as the share's owner UID is preserved.

## Server-side notes

- `passdb.tdb` lives on an `emptyDir` under `/var/lib/samba/private`, so the
  entrypoint re-imports users from `/run/secrets/*` on every pod restart.
  Rotating a password in the source secret takes effect on the next pod
  restart.
- The service is `SingleStack: IPv4` by design — dual-stack SMB against a
  ULA-scoped LB IP has interoperability issues with clients that only hold a
  global IPv6 address (macOS 15 `smbutil` will time out on the AAAA reply from
  DNS), and `hostsAllow` becomes harder to reason about.

## Maintainers

| Name | Email | Url |
| ---- | ------ | --- |
| swagner-de | <swagner-de@users.noreply.github.com> |  |
