# joplin

![Version: 0.1.0](https://img.shields.io/badge/Version-0.1.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 3.7.2](https://img.shields.io/badge/AppVersion-3.7.2-informational?style=flat-square)
Self-hosted sync server for the Joplin note-taking app
**Homepage:** <https://joplinapp.org/>

## Features

- Single stateless pod — all data stored in PostgreSQL (no PVC required for app data)
- SAML authentication support (e.g. authentik) via mounted XML files
- Optional SMTP mailer
- Hardened security context: read-only root FS, non-root UID, dropped capabilities

## Database

Joplin Server requires a PostgreSQL database. Provision one externally (e.g. via
CloudNativePG) and supply the connection details via `envFromSecrets`.

The secret must provide: `POSTGRES_USER`, `POSTGRES_PASSWORD`.
Set `postgres.host` and `postgres.database` in values.

## SAML

Set `saml.enabled: true` and provide:

- A Secret named `joplin-saml-idp` (key `idp.xml`) with the IdP metadata XML
- A ConfigMap named `joplin-saml-sp` (key `sp.xml`) with the SP configuration XML

Both are mounted at `/saml/` inside the container.

Set `saml.localAuthEnabled: false` to disable password login and force SSO-only.

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| appBaseUrl | string | `"https://joplin.example.com"` |  |
| cnpg.clusterName | string | `"joplin-db"` |  |
| cnpg.prependReleaseName | bool | `false` |  |
| envFromSecrets | list | `[]` |  |
| mailer.enabled | bool | `false` |  |
| mailer.host | string | `""` |  |
| mailer.noreplyEmail | string | `""` |  |
| mailer.noreplyName | string | `"Joplin"` |  |
| mailer.port | string | `"587"` |  |
| mailer.security | string | `"starttls"` |  |
| saml.enabled | bool | `false` |  |
| saml.idpConfigFile | string | `"/saml/idp.xml"` |  |
| saml.idpSecretName | string | `"joplin-saml-idp"` |  |
| saml.localAuthEnabled | bool | `true` |  |
| saml.spConfigFile | string | `"/saml/sp.xml"` |  |
| saml.spConfigMapName | string | `"joplin-saml-sp"` |  |
