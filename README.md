# Odelia Viewer

## Overview 🎯
The Odelia Viewer is a cutting-edge medical imaging platform developed under the **EU ODELIA Project**. This platform facilitates the integration and analysis of medical images using advanced AI models, specifically designed for Odelia AI components.

> 🔗 For more information about the project's vision and goals, visit the official **[ODELIA Project Website](https://odelia.ai/)**.

---

## Installation and Setup 🚀
This section guides you through the process of getting the Odelia Viewer up and running on your local machine using Docker.

### Prerequisites
1.  **Git:** Installed to clone the repository.
2.  **Docker:** Installed on your computer (includes Docker Compose).

### Quick Start (Local Deployment)
Follow these steps for a complete local deployment:

<details>
<summary>View Quick Start Steps</summary>

1.  **Clone the Repository** (with submodules for AI models):
    ```bash
    git clone --recurse-submodules https://github.com/StratifAI-Research/odelia-deployment
    cd odelia-deployment
    ```

2.  **Run the Deployment:**
    Open a terminal in the project directory and start all services in detached mode:
    ```bash
    docker compose up -d
    ```
    > **Note:** Initial run warnings about not being able to pull images for `odelia-orthanc-router`, `odelia-orthanc-viewer`, and the AI models (`odelia-breast-cancer-classification`, `odelia-mst-classifier`) are **normal**. These images will be built automatically from source.

3.  **Access the Viewer:**
    Open your web browser and navigate to (use viewer/viewer as default credentials):
    ```
    http://localhost:8081
    ```
    > **Configuration Note:** All settings are pre-set for `http://localhost:8081` for local deployment. No changes are required.
</details>

### Directory Structure
<details>
<summary>View Deployment Directory Structure</summary>

```
├── docker-compose.yml
├── config/                     # Deployment configuration files
│   ├── app-config.js           # OHIF viewer settings (OIDC, DICOMWeb)
│   ├── nginx.conf              # Nginx reverse proxy settings
│   ├── ohif-keycloak-realm.json# Keycloak realm import (ohif)
│   └── orthanc-router.json     # Orthanc router configuration
├── grafana/                    # Grafana provisioning for monitoring
│   └── provisioning/...
└── volumes/                    # Persistent data volumes
    ├── orthanc-viewer-db/      # Local Orthanc DICOM data
    └── feedback-db/            # AI feedback data
```


</details>

---


## Getting Started 💡
Once the viewer is running, here are the main use cases for interacting with the platform.

### Uploading Images to Orthanc
<details>
<summary>View Image Upload Methods</summary>

You can upload DICOM images (`.dcm` files) using three methods:

* **Method A: OHIF Viewer (Recommended)**
    * Navigate to the viewer at `http://localhost:8081`.
    * Use the **upload button** to drag and drop your DICOM files.
* **Method B: Orthanc Web Interface**
    * Access the Orthanc dashboard at `http://localhost:8000`.
    * Click **"Upload"** and select your DICOM files.
* **Method C: DICOM Send (C-STORE)**
    * Configure your external DICOM source/PACS with:
        * **AE Title:** `ORTHANC-VIEWER`
        * **Host:** `localhost`
        * **Port:** `2000`
</details>

### AI Routing and Analysis
<details>
<summary>View AI Analysis Workflow</summary>

Send studies to the integrated AI models for analysis:

1.  **Open a Study** in the Odelia Viewer. The **AI Analysis** panel (right sidebar) will automatically detect the active study.
2.  **Select Series** - Choose the specific series you want to analyze and click **"Next"**.
3.  **Select AI Model** and click **"Send to AI"**:
    * Classification model (breast cancer)
    * MST AI model (requires `HF_TOKEN` configuration - see Configuration section).
4.  **View Results** - The AI-processed studies, including annotations and results, will appear in your study list.
</details>

### User Management
<details>
<summary>View User Management Details</summary>

Please see the documentation: [Adding New Users](docs/adding_new_users.md)
</details>

---

## Architecture Components 🔄
The Odelia Viewer deployment consists of several interconnected components, each serving a specific purpose:

| Component | Purpose |
| :--- | :--- |
| **Odelia Viewer** | The frontend application (OHIF-based) for viewing medical images, customized with features for AI routing and feedback. |
| **Keycloak** | Handles **User Authentication** and **Authorization** (OIDC). Admin Console is available at `http://localhost:8081/keycloak`. |
| **Local Orthanc Instance** | The core **DICOM Server**. It acts as a local PACS, receiving, storing, and managing all DICOM medical images. |
| **Orthanc Router** | The **Traffic Controller** for the AI pipeline. It receives studies, routes them to the appropriate AI model, wraps the inference results in DICOM, and sends everything back to the viewer. |
| **AI Models** | Services that receive studies from the Router and return inference results (e.g., `odelia-breast-cancer-classification`, `odelia-mst-classifier`). |
| **Nginx** | Acts as a **Reverse Proxy**, routing traffic from port `8081` to the appropriate backend services (Viewer, Keycloak, etc.). |
| **Grafana** | An optional component for **Monitoring** and visualization of system metrics. |

---


## Configuration Details ⚙️
The viewer is pre-configured, but you may need to adjust settings for production or advanced use.

### Main Viewer and Server Configuration
<details>
<summary>View Configuration Files and Settings</summary>

| File | Purpose | Keycloak/OIDC Settings | Other Settings |
| :--- | :--- | :--- | :--- |
| `config/app-config.js` | **OHIF Viewer Settings** | `oidc[0].authority`, `oidc[0].redirect_uri`, `oidc[0].post_logout_redirect_uri` | DICOMWeb endpoints, OHIF features. |
| `config/nginx.conf` | **Web Server Reverse Proxy** | Routes `/keycloak/` to the Keycloak service. | Maps `/` to the viewer and static content. Set `server_name` for production. |
| `docker-compose.yml` | **Service Environment Variables** | `KC_HOSTNAME_URL`, `KC_HOSTNAME_ADMIN_URL` (for production domain changes). | `HF_TOKEN` (for MST classifier), volumes, ports, etc. |
</details>

### Keycloak Configuration 🔐
<details>
<summary>View Keycloak Setup Details</summary>

The Odelia Viewer uses OHIF's internal OIDC module for authentication.

* **Admin Console:** Access at `http://localhost:8081/keycloak`
* **Default Credentials:** `admin` / `admin`
* **Realm:** The `ohif` realm is auto-imported from `config/ohif-keycloak-realm.json`.
* **Production Deployment:** When deploying to a server or changing the domain, you **must** update the `ohif_viewer` client settings in the Keycloak UI (or `ohif-keycloak-realm.json`):
    * `Root URL`, `Home URL`
    * `Valid Redirect URIs` (e.g., `https://YOUR_DOMAIN/viewer/callback`)
    * `Web Origins`
    * Also update `KC_HOSTNAME_URL` and `KC_HOSTNAME_ADMIN_URL` in `docker-compose.yml`.
</details>

### AI Model Configuration (Hugging Face) 🤖
<details>
<summary>View MST Classifier Setup (HF_TOKEN)</summary>

The **MST Classification model** requires a Hugging Face token for access. The default **breast-cancer-classification** model does not.

1.  **Obtain Token:** Get a "Read" access token from your Hugging Face account settings.
2.  **Accept Licenses:** **Crucially**, you must log in to Hugging Face and accept the licenses for:
    * [ODELIA-AI/MST](https://huggingface.co/ODELIA-AI/MST) (Model Usage Agreement)
    * [DINOv3](https://huggingface.co/facebook/dinov3-vits16-pretrain-lvd1689m) (DINOv3 License terms)
3.  **Configure Token:** Set the `HF_TOKEN` environment variable for the `mst-classifier` service:
    * **Option 1 (Recommended):** Edit `docker-compose.yml` and replace the placeholder:
        ```yaml
        environment:
          HF_TOKEN: "your_actual_token_here" # REPLACE with your token
        ```
    * **Option 2 (Environment Variable):**
        ```bash
        export HF_TOKEN=your_actual_token_here
        docker compose up -d
        ```
    > ⚠️ **Security Warning:** Never commit your actual token to a version control system like Git. Use environment variables or a `.env` file that is in your `.gitignore`.
</details>

---

## Support 🤝
If you encounter any issues, please follow the steps below to gather diagnostic information before reaching out.

### Collecting Logs for Support
<details>
<summary>View Log Collection Commands</summary>

Create a `logs` directory and run the following commands to collect logs from all components:

1.  **Viewer Logs**
    ```bash
    docker logs odelia-viewer > logs/viewer.log
    ```
2.  **Orthanc Logs**
    ```bash
    docker logs odelia-orthanc-viewer > logs/orthanc.log
    ```
3.  **Router Logs**
    ```bash
    docker logs odelia-orthanc-router > logs/router.log
    ```
4.  **System & Container Info**
    ```bash
    docker system info > logs/docker-info.log
    docker ps -a > logs/containers.log
    ```
5.  **Configuration Files**
    ```bash
    cp config/* logs/
    ```
</details>

### Contact
<details>
<summary>View Contact Information</summary>

Once all files are collected:
1.  Compress the `logs` directory.
2.  Provide a clear description of the issue.
3.  Attach the compressed logs and send them to: **b.maksudov@lua.tatar**
</details>


## Acknowledgement 🎗️
* This platform, the Odelia Viewer, is built upon the foundational work of the [Open Health Imaging Foundation (OHIF) Viewer](https://ohif.org/). We gratefully acknowledge the OHIF community for developing this platform, which serves as the base for our specialized AI integration features.

* This work was funded by the European Union's Horizon Europe research and innovation programme under Grant Agreement No. [101057091](https://doi.org/10.3030/101057091).
