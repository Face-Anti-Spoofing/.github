# Biometric Anti-Spoofing & Deepfake Detection Platform

An advanced face anti-spoofing and presentation attack detection (PAD) system. This platform combines Vision Transformers (ViT) and YOLO detection models with an intelligent consensus fusion engine, explainable AI (XAI) overlays, and a multi-model research dashboard to analyze decision vulnerabilities and model biases.

---

## Table of Contents

- [Key Features](#key-features)
- [System Architecture & Consensus Logic](#system-architecture--consensus-logic)
- [Repository Structure](#repository-structure)
- [Prerequisites](#prerequisites)
- [Installation & Setup](#installation--setup)
  - [Backend](#1-backend-service-configuration)
  - [Frontend](#2-frontend-user-interface-configuration)
- [API Reference](#api-reference)
- [Repository Management & Organization Migration](#repository-management--organization-migration)

---

## Key Features

| Feature | Description |
|---|---|
| **Hierarchical Consensus Fusion Engine** | Dynamically weights and combines predictions from independent YOLO and Vision Transformer models. |
| **Explainable AI (XAI)** | Generates real-time visual attention maps using Grad-CAM to expose what features the ViT model focuses on during inference. |
| **Multi-Model Research Dashboard** | Concurrently runs and compares four models (Local ResNet-18, Local ViT, and two remote Hugging Face deepfake detectors) to analyze disagreements and bias labels. |
| **Ablation Evaluation Tools** | Built-in command-line benchmarking to isolate single-model performance against metrics like APCER, BPCER, and ACER. |

---

## System Architecture & Consensus Logic

The platform operates in three execution modes to optimize accuracy, throughput, and research analysis.

### 1. Hierarchical Consensus Mode (`mode=consensus`)

Roboflow (YOLO) acts as the primary structural gatekeeper for face-localized analysis:

- **No Detection** → Returns `INCONCLUSIVE`.
- **YOLO detects Spoof** → Immediately flags as `SPOOF`. The final confidence score integrates the ViT prediction weight if both models agree on the attack vector.
- **YOLO detects Real** → Consults the Vision Transformer.
  - If ViT **agrees** → final verdict is `REAL`.
  - If ViT **disagrees** (flags as spoof) → returns `REAL` but applies a strict **25% confidence penalty** on the fused score to reflect the systemic conflict.
- **API Exceptions** → Roboflow API failures surface gracefully as an `ERROR` rather than defaulting to a spoof verdict.

### 2. Single-Model Modes (`vit_only`, `yolo_only`)

Bypasses the fusion layer to route requests directly to a single isolated pipeline. This mode is explicitly used for ablation studies and evaluating independent model performance within the UI.

---

## Repository Structure

```
├── backend/
│   ├── main.py          # FastAPI application server & routing gateway
│   ├── engine.py        # Shared inference, model mapping, consensus logic & metrics
│   ├── benchmark.py     # CLI validation script for custom evaluation sets
│   └── xai_vit.py       # ViT Grad-CAM visual extraction engine
├── frontend/            # React + Vite single-page user interface
├── datasets/            # Benchmarking environment (local only)
│   ├── live/            # Verification targets for genuine presentations
│   └── spoof/           # Verification targets for presentation attacks
└── docs/                # Comprehensive technical and academic logs
    ├── architecture.md  # System designs and architectural schemas
    └── benchmarks.md    # Academic performance records (ACER, APCER)
```

> **Note on Datasets:** To protect storage quotas, local evaluation images inside `datasets/live/` and `datasets/spoof/` are managed via `.gitignore`. Only initialization files (`.gitkeep`) are committed to version control.

---

## Prerequisites

| Requirement | Details |
|---|---|
| **Runtime Environments** | Node.js v16+ and Python 3.9+ |
| **External Access** | Active internet connection for Hugging Face Hub downloads and the serverless Roboflow API endpoint |
| **Credentials** | Roboflow API Key and deployed Model ID. An optional `HF_TOKEN` can be supplied to bypass low rate limits on Hugging Face infrastructure |

---

## Installation & Setup

### 1. Backend Service Configuration

Navigate to the backend directory and create a virtual environment:

```bash
cd backend
python -m venv venv

# Activate the virtual environment
# On Windows:
venv\Scripts\activate
# On macOS/Linux:
source venv/bin/activate

pip install -r requirements.txt
```

Create an environment profile at `backend/.env`:

```env
ROBOFLOW_API_KEY=your_api_key_here
ROBOFLOW_MODEL_ID=your_model_id_here

# Optional: override for dedicated or on-premise infrastructure
# ROBOFLOW_API_URL=https://roboflow.com

# Optional: for higher Hugging Face rate limits
# HF_TOKEN=your_huggingface_token_here
```

Launch the FastAPI execution engine:

```bash
python main.py
```

The server initializes at `http://localhost:8000`. Confirm connection integrations at `http://localhost:8000/health`.

---

### 2. Frontend User Interface Configuration

```bash
cd frontend
npm install
npm run dev
```

The web interface initializes locally at `http://localhost:5173`.

---

## API Reference

### Dual Anti-Spoofing Evaluation

**Endpoint:** `POST /api/dual_antispoof`  
**Format:** `multipart/form-data`

| Parameter | Type | Description |
|---|---|---|
| `image` | Binary Image File | The image to analyze |
| `mode` | `consensus` \| `vit_only` \| `yolo_only` | Inference execution mode |

**Response:** Returns a JSON payload with the system verdict, mathematical confidence, execution mode, and granular raw responses inside nested `details.huggingface` and `details.roboflow` objects.

---

### Explainable AI Feature Mapping

**Endpoint:** `POST /api/explain`  
**Format:** `multipart/form-data`

| Parameter | Type | Description |
|---|---|---|
| `image` | Binary Image File | The image to analyze |

**Response:** Returns a `predicted_label` alongside `overlay_base64` — a transparent PNG showing the Grad-CAM heat map generated from the Vision Transformer's final attention layer.

---

### System Liveness Check

**Endpoint:** `GET /health`

**Response:** Verifies server status, active deep learning model configurations, validation arrays, and external API connectivity mappings.

---
