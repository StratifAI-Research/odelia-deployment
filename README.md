# Odelia Viewer Deployment Guide

## Overview 🎯
This repository provides deployment instructions for the Odelia Viewer, a medical imaging platform for Odelia AI integration.

## Quick Start ⚡
1. Clone the repository:
   ```bash
   git clone https://github.com/StratifAI-Research/odelia-deployment
   cd odelia-deployment
   ```

2. Make sure you have Docker installed on your computer
3. Open a terminal in the project directory
4. Run the following command:
   ```bash
   docker-compose up -d
   ```
5. Open your web browser and navigate to `http://localhost:80`

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
odelia-viewer/
├── config/           # Configuration files
│   ├── nginx.conf    # Web server settings
│   └── app-config.js # Viewer settings
├── logs/            # System logs
└── volumes/         # Storage for medical images
```

## Configuration ⚙️
The viewer comes pre-configured for most use cases. If you need to make changes:
1. The main configuration file is located in `config/app-config.js`
2. Web server settings can be modified in `config/nginx.conf`

## Accessing the Viewer 🌐
- Local deployment: `http://localhost:80`
- Remote deployment: `http://your-server-address:80`

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
