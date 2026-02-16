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
    Open your web browser and navigate to:
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

## Updating 🔄

This section covers how to update an existing Odelia Viewer deployment to the latest version.

<details>
<summary>View Full Update Procedure</summary>

#### 1. Back Up Persistent Data

Before updating, back up your persistent data. The `volumes/` directory contains DICOM images and the feedback database. The `postgres_data` named Docker volume holds the Keycloak database.

```bash
# Back up the volumes directory
cp -r volumes/ volumes-backup-$(date +%F)/

# Back up the Keycloak Postgres database
docker run --rm -v postgres_data:/data -v $(pwd):/backup alpine tar czf /backup/postgres-backup.tar.gz /data
```

#### 2. Stop Running Containers

```bash
docker compose down
```

> **Note:** `docker compose down` does **not** delete volumes. Your data remains intact.

#### 3. Pull Latest Deployment Repository

```bash
git pull origin main
```

Replace `main` with the branch you are tracking if different.

#### 4. Update Submodules

The deployment uses **nested submodules** (two levels deep), so the `--recursive` flag is required:

```bash
git submodule update --init --recursive
```

This updates:
- `orthanc-routing-example/` — Orthanc router and viewer configurations
- `orthanc-routing-example/MLIntegration/` — all AI model services (breast-cancer-classification, MST, MedGemma, chat-middleware)

#### 5. Review Configuration Changes

If you have customized any configuration files, check for upstream changes before restarting:

```bash
git diff HEAD@{1} -- docker-compose.yml config/
```

Files to watch for changes:
- `docker-compose.yml` — new environment variables, ports, or services
- `config/nginx.conf` — reverse proxy routes
- `config/app-config.js` — OHIF viewer settings
- `config/orthanc-router.json` — Orthanc routing configuration

> **Important:** Re-apply any custom settings (e.g., `HF_TOKEN`, production domain URLs) after pulling, as they may be overwritten by the update.

#### 6. Rebuild Docker Images

Rebuild all images that are built from source:

```bash
docker compose build --no-cache
```

Or rebuild only specific services:

```bash
docker compose build <service-name>
```

**Services built from source** (require rebuild): `orthanc-viewer`, `orthanc-router`, `orthanc-router-mst`, `orthanc-router-medgemma`, `breast-cancer-classification`, `mst-classifier`, `medgemma-mri`, `chat-middleware`

**Pre-built images** (update via pull): `viewer`, `grafana`, `keycloak`, `postgres`

```bash
docker compose pull viewer grafana keycloak postgres
```

#### 7. Update Ollama Model (if applicable)

If you use the Chat AI feature, update the Ollama model on the host machine:

```bash
ollama pull thiagomoraes/medgemma-1.5-4b-it:F16
```

#### 8. Start Services

```bash
docker compose up -d
```

#### 9. Verify Deployment

```bash
# Check all containers are running
docker compose ps

# Check logs for errors
docker compose logs --tail=50
```

Then open `http://localhost:8081` in your browser to confirm the viewer loads correctly.

</details>

<details>
<summary>Quick-Reference Commands</summary>

For experienced users, here is the condensed update sequence:

```bash
docker compose down
git pull origin main
git submodule update --init --recursive
docker compose build --no-cache
docker compose up -d
docker compose ps
```

</details>

---

## Architecture Components 🏗️
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

### Ollama + MedGemma Setup (Chat AI) 💬
<details>
<summary>View Ollama Installation and MedGemma Configuration</summary>

The **Chat AI** panel in the viewer uses a local [Ollama](https://ollama.com/) instance running the **MedGemma** vision-language model. Ollama runs on the **host machine** (not inside Docker) and is accessed by the `chat-middleware` container via `host.docker.internal`.

#### 1. Install Ollama

**Linux:**
```bash
curl -fsSL https://ollama.com/install.sh | sh
```

**macOS / Windows:**
Download the installer from [https://ollama.com/download](https://ollama.com/download).

Verify the installation:
```bash
ollama --version
```

#### 2. Pull the MedGemma Model

Pull the MedGemma model that the chat middleware expects by default:
```bash
ollama pull thiagomoraes/medgemma-1.5-4b-it:F16
```

> **Note:** This model is ~8 GB. Ensure you have sufficient disk space and a GPU with enough VRAM for acceptable performance. Smaller quantizations (e.g., `Q8_0`, `Q4_K_M`) are available if resources are limited — adjust the `OLLAMA_MODEL` variable accordingly.

Verify the model is available:
```bash
ollama list
```

#### 3. Start Ollama

Make sure the Ollama server is running before starting the Docker stack:
```bash
ollama serve
```

By default, Ollama listens on `http://localhost:11434`. You can confirm it is running:
```bash
curl http://localhost:11434/api/tags
```

#### 4. Docker Compose Configuration

The `chat-middleware` service in `docker-compose.yml` is pre-configured to connect to Ollama on the host:

```yaml
chat-middleware:
  environment:
    OLLAMA_URL: "http://host.docker.internal:11434"
    OLLAMA_MODEL: "thiagomoraes/medgemma-1.5-4b-it:F16"
    OLLAMA_NUM_CTX: "128000"  # 128k context window
    NUM_SLICES: "5"
  extra_hosts:
    - "host.docker.internal:host-gateway"
```

**Key variables you may want to adjust:**

| Variable | Default | Description |
| :--- | :--- | :--- |
| `OLLAMA_URL` | `http://host.docker.internal:11434` | URL of the Ollama API. Change if Ollama runs on a remote machine. |
| `OLLAMA_MODEL` | `thiagomoraes/medgemma-1.5-4b-it:F16` | Model tag in Ollama. Change if you pulled a different quantization or model. |
| `OLLAMA_NUM_CTX` | `128000` | Context window size in tokens. Reduce if you run into memory issues. |
| `NUM_SLICES` | `5` | Number of DICOM slices sent to the model per study for analysis. |

> **Linux note:** The `extra_hosts` mapping (`host.docker.internal:host-gateway`) is required on Linux to allow containers to reach the host network. This is included in the default `docker-compose.yml` and should not be removed.

#### 5. Using a Different Model

To use a different Ollama-compatible model:

1. Pull the desired model:
    ```bash
    ollama pull <model_name>:<tag>
    ```
2. Update the `OLLAMA_MODEL` variable in `docker-compose.yml`:
    ```yaml
    OLLAMA_MODEL: "<model_name>:<tag>"
    ```
3. Restart the chat middleware:
    ```bash
    docker compose up -d chat-middleware
    ```

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
