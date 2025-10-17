# Odelia Viewer Deployment Guide

## Overview 🎯
This repository provides deployment instructions for the Odelia Viewer, a medical imaging platform for Odelia AI integration.

## Quick Start ⚡
1. Clone the repository:
   ```bash
   git clone --recurse-submodules  https://github.com/StratifAI-Research/odelia-deployment
   cd odelia-deployment
   ```

2. Make sure you have Docker installed on your computer
3. Open a terminal in the project directory
4. Run the following command:
   ```bash
   docker-compose up -d
   ```
<details>
<summary>Replace domain/host checklist</summary>

When deploying to a server or custom domain, update these locations:

- `custom/deploy/docker-compose.yml`
  - `KC_HOSTNAME_URL` and `KC_HOSTNAME_ADMIN_URL`: set to `https://YOUR_DOMAIN/keycloak/` (use `http://` if not using TLS).
  - Keycloak healthcheck: replace `YOUR_DOMAIN` and the `host:` header to reflect your domain.

- `custom/deploy/config/app-config.js`
  - `oidc[0].authority`: keep `'/keycloak/realms/ohif'` if Nginx proxies Keycloak at `/keycloak/`; otherwise use `https://YOUR_DOMAIN/keycloak/realms/ohif`.
  - `oidc[0].redirect_uri`: keep `'/callback'` and allow it in Keycloak client settings.
  - `oidc[0].post_logout_redirect_uri`: ensure it is permitted in Keycloak client settings.

- `custom/deploy/config/ohif-keycloak-realm.json`
  - If you rely on realm auto-import, adjust (or update in UI post-import): `rootUrl`, `adminUrl`, `baseUrl`, `redirectUris`, and `webOrigins` to `https://YOUR_DOMAIN`.

- `custom/deploy/config/nginx.conf`
  - Optional: set `server_name` to your domain if desired.
  - Keycloak is reverse-proxied at `/keycloak/`; keeping OHIF `authority` relative simplifies domain moves.
  - Note: `/oauth2/*` blocks are legacy (oauth2-proxy). They are unused with internal OIDC and can be removed later.

- This README
  - Replace any remaining `localhost` URLs if your deployment uses a different host/port.

</details>
5. Open your web browser and navigate to `http://localhost:8081`



## Architecture Overview 🔄
The send to AI pipeline consists of the following components:

1. **Odelia Viewer**: OHIF based viewer, customized with AI routing and feedback features
2. **Local Orthanc Instance**: Acts as the DICOM server, receiving and managing medical images
3. **Orthanc Router**: The traffic controller that:
   - Receives studies from the local Orthanc
   - Routes them to the appropriate AI model
   - Receives AI inference results
   - Wraps the results in DICOM format
   - Sends everything back to the viewer
4. **AI Model**: The AI model that receives the studies and returns the inference results

## Use Cases 💡
<details>
<summary>AI Routing</summary>
[Content to be added]
</details>

<details>
<summary>Uploading Images to Orthanc</summary>
[Content to be added]
</details>

## Directory Structure 📁
The deployment uses the following structure:
```
custom/deploy/
├── docker-compose.yml
├── config/                         # Deployment configuration
│   ├── app-config.js               # OHIF viewer settings (OIDC, DICOMWeb)
│   ├── nginx.conf                  # Nginx reverse proxy
│   ├── ohif-keycloak-realm.json    # Keycloak realm import (ohif)
│   ├── orthanc-router.json         # Orthanc router config
├── grafana/
│   └── provisioning/...
└── volumes/                        # Persistent data volumes
    ├── orthanc-viewer-db/
    └── feedback-db/
```

## Configuration ⚙️
The viewer comes pre-configured for most use cases. If you need to make changes:
1. The main configuration file is located in `config/app-config.js`
2. Web server settings can be modified in `config/nginx.conf`

## Keycloak Configuration 🔐
<details>
<summary>View Keycloak setup</summary>

Odelia Viewer uses OHIF's internal OIDC module to authenticate directly with Keycloak (no oauth2-proxy). Nginx exposes Keycloak at `/keycloak/` so OHIF can use a relative OIDC `authority`.

- Admin Console: `http://localhost:8081/keycloak` (replace with your domain in production)
- Realm import: `custom/deploy/config/ohif-keycloak-realm.json` is auto-imported at container start
- Default admin credentials: `admin` / `admin`

In Keycloak UI (Realm: `ohif` → Client: `ohif_viewer`), set for YOUR_DOMAIN:

- Root URL: `https://YOUR_DOMAIN`
- Home URL: `https://YOUR_DOMAIN`
- Valid Redirect URIs: `https://YOUR_DOMAIN/callback`
- Web Origins: `https://YOUR_DOMAIN`
- Post Logout Redirect URIs: `https://YOUR_DOMAIN/*` (or your exact path)

If you change domains or enable TLS, also set in compose (service `keycloak`):

- `KC_HOSTNAME_URL=https://YOUR_DOMAIN/keycloak/`
- `KC_HOSTNAME_ADMIN_URL=https://YOUR_DOMAIN/keycloak/`

Troubleshooting:

- "invalid parameter: redirect_uri": fix Redirect URIs/Web Origins in the `ohif_viewer` client.
- CORS errors: ensure Web Origins matches your scheme + domain.
- References: [User Account Control](https://docs.ohif.org/deployment/user-account-control/), [Authorization (OIDC)](https://docs.ohif.org/deployment/authorization/)

</details>

## Accessing the Viewer 🌐
- Local deployment: `http://localhost:8081`
- Remote deployment: `http://your-server-address:8081`

## Support 🤝
If you encounter any issues or need assistance:

### Collecting Logs for Support
To help us diagnose issues, please collect logs from all components:

1. **Viewer Logs**
   ```bash
   docker logs odelia-viewer > logs/viewer.log
   ```

2. **Orthanc Logs**
   ```bash
   docker logs odelia-orthanc > logs/orthanc.log
   ```

3. **Router Logs**
   ```bash
   docker logs odelia-router > logs/router.log
   ```

4. **System Logs**
   ```bash
   # Collect Docker system logs
   docker system info > logs/docker-info.log
   docker ps -a > logs/containers.log
   ```

5. **Configuration Files**
   ```bash
   # Copy current configuration
   cp config/* logs/
   ```

Once you've collected all logs, please:
1. Compress the `logs` directory
2. Contact the us at ....
3. Provide a description of the issue
4. Attach the compressed logs
