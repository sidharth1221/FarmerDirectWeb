# YOLO (Ultralytics) & Gemini (Vertex AI) — Setup Guide

This README explains how to download and set up YOLO (Ultralytics YOLOv8) for object detection and how to configure Google’s Gemini models through Vertex AI (often called the Gemini API). It includes recommended commands for Windows PowerShell, notes about GPU vs CPU installs, and repository-specific guidance.

**Important repository notes**
- This project already includes a YOLOv8 weight at `backend/models/yolov8n.pt`.
- There is a `backend/service-account.json` file in this repository. **Do not commit service account keys to public repositories.** See the "Protect Credentials" section below.
- The backend lists required packages in `backend/requirements.txt` (includes `ultralytics`, `torch`, `google-auth`, etc.).

**Contents**
- Overview
- Prerequisites
- YOLO (Ultralytics) setup
- Using the included model weights
- Gemini / Vertex AI setup
- Minimal Gemini example (Python)
- Protect credentials & Git tips
- Troubleshooting & links

---

**Overview**: This guide helps you set up a local Python environment, install YOLOv8 and other dependencies from `backend/requirements.txt`, run YOLO for detection, and configure access to Google’s Gemini models via Vertex AI.

**Prerequisites**
- Python 3.8+ installed and on PATH
- Git (optional, for repo management)
- (Optional) CUDA-capable GPU and matching NVIDIA drivers + CUDA toolkit if you want GPU-accelerated PyTorch
- A Google Cloud account with a project for using Vertex AI (Gemini)

**YOLO (Ultralytics) setup**
1. Create and activate a virtual environment (PowerShell):

```powershell
cd C:\Users\sidha\OneDrive\Desktop\FarmerDirectWeb
python -m venv .venv
.venv\Scripts\Activate.ps1
```

2. Install Python dependencies (recommended to use the repo requirements):

```powershell
pip install --upgrade pip
pip install -r backend/requirements.txt
```

Note about PyTorch: the `requirements.txt` includes `torch` and `torchvision`. For best performance on GPU, install the correct torch wheel matching your CUDA version from https://pytorch.org/ (use the selector there). If you don't have GPU or you want a CPU-only install, follow the CPU install instructions on PyTorch site or use the CPU wheel index.

3. Confirm `ultralytics` is available (this provides the `yolo` CLI):

```powershell
yolo -h
```

If the CLI works, `yolo` will show usage information.

**Using the included model weights**
- The repo includes a small YOLOv8 weight at `backend/models/yolov8n.pt`. You can run a quick detection on an image:

```powershell
# Example: detect on a local image
yolo detect predict model=backend/models/yolov8n.pt source=tests/grading_images

# Or predict on a single image
yolo detect predict model=backend/models/yolov8n.pt source=tests/grading_images/example.jpg
```

- To download other official weights from Ultralytics, see the Ultralytics docs or use the `ultralytics` API to load pretrained models.

**Gemini (Vertex AI) setup**
Gemini models are provided through Google Cloud Vertex AI (the managed GenAI offering). The steps below show the common setup flow.

1. Create/choose a Google Cloud project
- In the Google Cloud Console, create a project or pick an existing one.
- Enable the Vertex AI API (Vertex AI / Generative AI models) via the console or with `gcloud`:

```powershell
# (if you use Cloud SDK)
gcloud services enable aiplatform.googleapis.com --project=YOUR_PROJECT_ID
```

2. Create a service account and grant permissions
- Create a service account (Console > IAM & Admin > Service accounts).
- Grant roles such as `Vertex AI User` (or the minimal roles your app needs) and `Storage Object Viewer` if you access GCS artifacts.
- Create and download a JSON key for the service account. Save it somewhere safe.

3. Securely configure the key on your machine
- Set the `GOOGLE_APPLICATION_CREDENTIALS` environment variable to the JSON key path (PowerShell example):

```powershell
$env:GOOGLE_APPLICATION_CREDENTIALS = 'C:\path\to\your\service-account.json'
```

- For persistent use, set the variable in your system environment variables or add the above line to your PowerShell profile (only on a secure machine).

4. Install the Google client library

```powershell
pip install google-cloud-aiplatform
```

5. Choose the model name / region
- Vertex AI offers several generative models. "Gemini" model IDs and availability vary by region; check the Vertex AI docs for the current model names (for example `models/text-bison@001` or `models/gemini-*` may appear in docs).

**Minimal Python example (high level)**
Below is a concise example showing how to initialize Vertex AI and call a generative model. Check the Vertex AI docs for the exact class/method names and model IDs as they evolve.

```python
# example_gemini_test.py
from google.cloud import aiplatform

PROJECT_ID = "your-gcp-project-id"
LOCATION = "us-central1"  # pick the region you enabled

# Initialize the aiplatform client
aiplatform.init(project=PROJECT_ID, location=LOCATION)

# Replace the following with the exact model name / class recommended in Google docs
# Example (API surface evolves): many SDKs provide a TextGenerationModel helper
try:
    model = aiplatform.TextGenerationModel.from_pretrained("models/text-bison@001")
    response = model.predict("Give me a one-sentence summary of apples.")
    print(response)
except Exception as e:
    print("Check Vertex AI docs for the current model name and SDK usage. Error:", e)
```

Notes:
- The Vertex AI SDK and class names change over time; consult the official docs for the latest examples: https://cloud.google.com/vertex-ai/docs/generative-ai
- If you prefer REST, you can call the Vertex AI REST endpoints with an OAuth token from your service account.

**Protect Credentials & Git tips**
- Never commit `service-account.json` or other secrets into git. If a key is already committed, rotate it immediately.
- To remove the file from git history and add it to `.gitignore` (quick steps):

```powershell
# remove file from index but keep locally
git rm --cached backend/service-account.json
# add to .gitignore if not present
Add-Content .gitignore "backend/service-account.json"
git add .gitignore
git commit -m "Remove service account and add to .gitignore"
```

If the key has been pushed to a public repo, revoke and rotate the service account key immediately via Google Cloud Console.

**Troubleshooting**
- `yolo` CLI not found: ensure your virtualenv is active and `ultralytics` installed.
- PyTorch errors: confirm you installed a compatible torch wheel for your OS and CUDA version. See https://pytorch.org/
- Vertex AI auth errors: verify `GOOGLE_APPLICATION_CREDENTIALS` points to the correct key and that the service account has the required roles.

**Helpful links**
- Ultralytics YOLO: https://docs.ultralytics.com/
- PyTorch installation selector: https://pytorch.org/
- Vertex AI Generative Models (official): https://cloud.google.com/vertex-ai/docs/generative-ai
- Google Cloud IAM / Service accounts: https://cloud.google.com/iam/docs

---

If you want, I can:
- Commit this README to the `main` branch and open a PR.
- Add a short example script that runs YOLO on one of the repo test images.
- Remove `backend/service-account.json` from git history (I will not revoke keys; you must rotate them via Console).

Tell me which of those (if any) you'd like me to do next.
