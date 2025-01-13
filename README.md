# Docker Deployment of the PKI Toolkit

This repository provides the required tools for a local docker deployment of the PKI Toolkit provided by the Munich University of Applied Sciences.

## Requirements

- Two valid HARICA Accounts
  - One for certificate requests
  - One for automated request validations (Needs at least the permission to approve SSL Requests)
- Installed Docker 
- OIDC at your IDP or any other OIDC provider
- Overall 5 FQDNs
  - Frontend (e.g. pki.example.edu)
  - Urls for EAB, PKI and Domainmanagement REST-Service
  - ACME


## Shibboleth IDP Configuration

Setting up and configuring an shibboleth IDP to provide an working OIDC endpoint can be pretty cumberstone.

The following configuration must be adapted for the own domains but provides a basic setup. The second client allows the use of OAuth2 beside OIDC. This results in 'readable' JWT for the frontend and the backend.

```.json
{
  "scope":"openid profile email offline_access Certificates Domains EAB",
  "redirect_uris":["https://pki.example.edu/api/auth/callback/oidc"],
  "client_id":"pki.example.edu"
  "subject_type":"pairwise",
  "client_secret":"XXXXXXXXXXXXX",
  "response_types": [
    "code"
  ],
  "grant_types": [
    "authorization_code",
    "refresh_token"
  ],
  "audience": [
    "https://api.example.edu"
  ]
}, {
  "client_id":"https://api.example.edu"
}

```

Please ensure that the following points are also configured: 

- Active `OAUTH2.TokenAudience` profile in the relying-party.xml
- The Client Secret is passed as HTTP-Header and thus should contain any strange special characters. We recommend a long alpha-numeric string


## Configuration

Before building and starting the containers serveral environment variables should be configured. Therefor you should copy the `.env.example` file to `.env` and set the appropriate values.

| Variable                | Description                                                                                                                   | Example Value                                          |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| USER                    | Your HARICA Username for certificate requests                                                                                 | youruser                                               |
| PASSWORD                | Your HARICA Password for certificate requests                                                                                 | yourpassword                                           |
| TOTP_SEED               | Your HARICA TOTP Seed for certificate requests                                                                                | totpseed                                               |
| VALIDATION_USER         | Your HARICA Username for certificate request validations                                                                      | youruser                                               |
| VALIDATION_PASSWORD     | Your HARICA Password for certificate request validations                                                                      | yourpassword                                           |
| VALIDATION_TOTP_SEED    | Your HARICA TOTP Seed for certificate request validations                                                                     | totpseed                                               |
| SMIME_KEY_LENGTH        | The requested keylength of the SMIME certificate.                                                                             | 3072                                                   |
| JWKS_URI                | The URI to your JSON Web Key Set to validate the OIDC/OAuth2 Authentication                                                   | `https://your.idp.example.edu/idp/profile/oidc/keyset` |
| AUDIENCE                | The used adiennce for OAuth2 (must be the same value as `AUTH_RESOURCE`)                                                      | `https://api.example.edu`                              |
| NEXTAUTH_URL            | The canonical URL of your site                                                                                                | `https://pki.example.edu`                              |
| NEXT_PUBLIC_AUTH_IDP    | The IDP that shal be used for OIDC authentication                                                                             | `https://sso.example.edu`                              |
| NEXT_PUBLIC_EAB_HOST    | The API Backend host for EAB operations                                                                                       | `https://eab.api.example.edu`                          |
| NEXT_PUBLIC_PKI_HOST    | The API Backend host for PKI operations                                                                                       | `https://pki.api.example.edu`                          |
| NEXT_PUBLIC_DOMAIN_HOST | The API Backend host for Domain operations                                                                                    | `https://domain.api.example.edu`                       |
| NEXT_PUBLIC_ACME_HOST   | The host for ACME operations                                                                                                  | `https://domain.api.example.edu`                       |
| NEXT_PUBLIC_SENTRY_DSN  | In case of using sentry the sentry DSN to upload error reports                                                                |                                                        |
| AUTH_CLIENT_ID          | The OIDC Client ID                                                                                                            | `pki`                                                  |
| AUTH_CLIENT_SECRET      | The OIDC Client Secret                                                                                                        | `Random String`                                        |
| AUTH_IDP                | The IDP that shal be used for OIDC authentication (Should be the same as during build time)                                   | `https://sso.example.edu`                              |
| AUTH_RESOURCE           | The requested OIDC resource to get OAuth2 working.                                                                            | `https://api.example.edu`                              |
| AUTH_SECRET             | The [Next.JS Auth Secret](https://next-auth.js.org/configuration/options#secret) used to encrypt JWT                          | `Random String`                                        |

## Getting Started

After setting all required values you can build the docker containers. 

```
docker compose build
```

Afterwards you should be able to start the deployment

```
docker compose up -d
```