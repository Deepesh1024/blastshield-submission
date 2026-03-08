<p align="center">
  <img src="https://img.shields.io/badge/BlastShield-Production%20Failure%20Simulator-FF6B00?style=for-the-badge&logoColor=white" alt="BlastShield"/>
</p>

<h1 align="center">🛡️ BlastShield</h1>

<p align="center">
  <strong>SRE-grade production failure simulation — right inside your editor.</strong><br/>
  Simulate production traffic, catch catastrophic failures before they happen, and generate postmortem reports — all without leaving VS Code.
</p>

<p align="center">
  <a href="https://github.com/Deepesh1024/blastshield-prodsim"><img src="https://img.shields.io/badge/Backend-blastshield--prodsim-blue?style=flat-square&logo=amazonaws" alt="Backend Repo"/></a>
  <a href="https://github.com/Deepesh1024/blastshield-extension"><img src="https://img.shields.io/badge/Extension-blastshield--extension-purple?style=flat-square&logo=visualstudiocode" alt="Extension Repo"/></a>
  <img src="https://img.shields.io/badge/python-3.13-3776AB?style=flat-square&logo=python&logoColor=white" alt="Python"/>
  <img src="https://img.shields.io/badge/typescript-5.x-3178C6?style=flat-square&logo=typescript&logoColor=white" alt="TypeScript"/>
  <img src="https://img.shields.io/badge/AWS-Lambda%20%7C%20Bedrock%20%7C%20S3-FF9900?style=flat-square&logo=amazonaws" alt="AWS"/>
  <img src="https://img.shields.io/badge/Docker-Sandboxed-2496ED?style=flat-square&logo=docker&logoColor=white" alt="Docker"/>
</p>

<br/>

<table align="center">
  <tr>
    <th>🔗 Source Code</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><a href="https://github.com/Deepesh1024/blastshield-prodsim"><b>📡 blastshield-prodsim</b></a></td>
    <td>Backend — Lambda orchestrator, EC2 sandbox server, AI analysis engine</td>
  </tr>
  <tr>
    <td><a href="https://github.com/Deepesh1024/blastshield-extension"><b>🧩 blastshield-extension</b></a></td>
    <td>VS Code Extension — Observability dashboard, failure maps, incident replay</td>
  </tr>
</table>

---

## 🎯 What is BlastShield?

BlastShield is an **end-to-end production failure simulation platform** that allows developers to stress-test their Python projects under realistic production conditions. It combines a serverless AI-powered backend with a rich VS Code extension to deliver a seamless "shift-left" SRE experience.

> **Think of it as a chaos engineering lab that lives inside your IDE.**

### The Problem

Most production failures are discovered _in production_ — cascading timeouts, race conditions, memory leaks under concurrency, and uncaught edge cases. Traditional testing catches syntax and logic bugs, but not the systemic failures that take down services at 2 AM.

### The Solution

BlastShield simulates real production traffic patterns against your code, executes it inside an isolated Docker sandbox, and uses AI to generate detailed postmortem reports — complete with failure timelines, blast radius analysis, and ready-to-apply patches.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        VS Code Extension                            │
│  ┌──────────────┐  ┌──────────────────┐  ┌───────────────────────┐ │
│  │ Command       │  │ Workspace Zipper │  │ Webview UI (React)    │ │
│  │ Palette       │──│ (adm-zip)        │  │ • Dashboard           │ │
│  │ Trigger       │  │                  │  │ • React Flow Graph    │ │
│  └──────┬───────┘  └────────┬─────────┘  │ • Timeline Replay     │ │
│         │                   │            │ • Diff Viewer          │ │
│         └───────────────────┤            │ • What-If Simulator    │ │
│                             │            └───────────────────────┘ │
└─────────────────────────────┼─────────────────────────────────────┘
                              │ POST /scan (zip payload)
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    AWS Lambda (FastAPI + Mangum)                     │
│                                                                     │
│  ┌────────────┐  ┌────────────┐  ┌─────────────┐  ┌────────────┐  │
│  │ Concurrency│  │ Latency    │  │ Chaos        │  │ Edge Case  │  │
│  │ Drills     │  │ Drills     │  │ Drills       │  │ Tests      │  │
│  └────────────┘  └────────────┘  └─────────────┘  └────────────┘  │
│                                                                     │
│  ┌──────────────────────┐       ┌──────────────────────────────┐   │
│  │ S3 Artifact Storage  │       │ EC2 Sandbox Client           │   │
│  │ (blastshield-artifacts)│      │ POST /run-sandbox            │   │
│  └──────────────────────┘       └──────────────┬───────────────┘   │
└─────────────────────────────────────────────────┼──────────────────┘
                                                  │
                                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     EC2 Sandbox Server (FastAPI)                     │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  Docker Container (read-only filesystem, no network)         │   │
│  │  └── runner.py executes user code in isolation               │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌──────────────────┐                                              │
│  │ Groq AI Analyzer │ ← Runtime error analysis                     │
│  └──────────────────┘                                              │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼  Results returned to Lambda
┌─────────────────────────────────────────────────────────────────────┐
│                    AI Analysis (Model Cascade)                       │
│                                                                     │
│  1. Amazon Nova Pro (Bedrock)        ✅ Primary                     │
│  2. Anthropic Prompt Router (Bedrock)   Fallback                    │
│  3. Groq openai/gpt-oss-120b           Final fallback              │
│  4. Static fallback                     Safety net                  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📦 Repositories

### [`blastshield-prodsim`](https://github.com/Deepesh1024/blastshield-prodsim) — Backend Engine

The serverless backend that powers all simulations. Built with **Python 3.13**, deployed on **AWS Lambda** via Mangum.

```
blastshield-agent/
├── app/
│   ├── api/scan.py              # Orchestrator endpoint
│   ├── core/drills.py           # Concurrency, latency, chaos drills
│   ├── core/edge_cases.py       # Edge case testing
│   ├── core/curl_runner.py      # Simulated load tests
│   ├── core/s3_storage.py       # S3 artifact upload
│   ├── core/ec2_client.py       # EC2 sandbox HTTP client
│   ├── core/extract.py          # Code/zip extraction
│   ├── ai/bedrock.py            # Bedrock + Groq AI client
│   └── ai/prompt.py             # Prompt builder
├── ec2_sandbox/
│   ├── server.py                # FastAPI sandbox server
│   ├── docker_runner.py         # Docker execution engine
│   ├── groq_analyzer.py         # Groq AI error analysis
│   ├── runner.py                # Runs INSIDE Docker container
│   ├── Dockerfile               # Docker image definition
│   └── requirements.txt         # EC2 dependencies
├── handler.py                   # Lambda entry point (Mangum)
├── serverless.yml               # Lambda deployment config
└── requirements.txt             # Lambda dependencies
```

**Key capabilities:**
| Capability | Description |
|---|---|
| 🔀 **Concurrency Drills** | Simulates concurrent request storms to identify race conditions |
| ⏱️ **Latency Injection** | Adds artificial latency to expose timeout vulnerabilities |
| 💥 **Chaos Engineering** | Random fault injection to test system resilience |
| 🧪 **Edge Case Testing** | Automated edge case generation and execution |
| 🐳 **Docker Sandboxing** | Code runs in read-only, network-isolated containers |
| 🧠 **AI Analysis** | Multi-model cascade for intelligent failure analysis |

---

### [`blastshield-extension`](https://github.com/Deepesh1024/blastshield-extension) — VS Code Extension

A full-screen observability platform inside VS Code. Built with **TypeScript**, **React 18**, **React Flow**, and **Chart.js**.

**Features:**

| Feature | Description |
|---|---|
| 📊 **Observability Dashboard** | High-fidelity engineering dashboard mimicking Datadog & Sentry |
| 🗺️ **Failure Propagation Map** | Visual React Flow graph showing how errors cascade across files |
| ⏪ **Incident Replay Timeline** | Step-by-step playback of the system crash |
| 🔧 **Root Cause & Patches** | AI-generated patches with a built-in diff viewer |
| 🔮 **What-If Simulations** | Adjust traffic, latency, and fault injection to re-test scenarios |
| 🎮 **Demo Mode** | Works fully offline with realistic mock data |

**Languages:** TypeScript (77.6%) · CSS (21.3%) · Other (1.1%)

---

## 🚀 Quick Start

### Prerequisites

- Python 3.13+
- Node.js 18+
- Docker
- AWS account (for Lambda/Bedrock/S3)
- API keys: `GROQ_API_KEY`, `AWS_BEARER_TOKEN_BEDROCK`

### 1. Backend — Local Development

```bash
# Clone the backend
git clone https://github.com/Deepesh1024/blastshield-prodsim.git
cd blastshield-prodsim

# Setup Python environment
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt

# Configure environment
cp .env.example .env   # Edit with your keys
export $(cat .env | xargs)

# Run the API server
uvicorn app.main:app --port 8000
```

### 2. EC2 Sandbox Server

```bash
cd ec2_sandbox
pip install -r requirements.txt

# Build the sandboxed runner image
docker build -t blastshield-runner .

# Configure and run
export GROQ_API_KEY="your-key"
export AWS_REGION="us-east-1"
python server.py   # Runs on port 9000
```

### 3. VS Code Extension

```bash
# Clone the extension
git clone https://github.com/Deepesh1024/blastshield-extension.git
cd blastshield-extension

# Install dependencies
npm install
cd webview-ui && npm install && cd ..

# Build
npm run build

# Or start dev server for UI testing
cd webview-ui && npm run dev
```

### 4. Run a Simulation

1. Open a workspace in VS Code
2. Open the Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`)
3. Run **`BlastShield: Run Production Simulation`**
4. The observability dashboard opens with full results

---

## 🔌 API Reference

### `POST /scan`

Accepts Python code as a JSON string or a `.zip` file upload:

```bash
# JSON code string
curl -X POST http://localhost:8000/scan \
  -H "Content-Type: application/json" \
  -d '{"code": "from fastapi import FastAPI\napp = FastAPI()\n@app.get(\"/health\")\ndef health():\n    return {\"ok\": True}"}'

# Zip file upload
curl -X POST http://localhost:8000/scan \
  -F "file=@project.zip"
```

<details>
<summary><strong>📋 Response Schema</strong></summary>

```json
{
  "scan_id": "uuid",
  "overall_score": 45,
  "simulation_results": {
    "files": [],
    "endpoints": [],
    "drills": {
      "concurrency": [],
      "latency": [],
      "chaos": []
    },
    "edge_cases": [],
    "curl_results": []
  },
  "deployment_validation": {
    "deployment_status": "success|failure|timeout",
    "runtime_errors": [],
    "logs": "...",
    "container_exit_code": 0
  },
  "ai_analysis": {
    "timeline": "...",
    "outage_scenario": "...",
    "blast_radius": "...",
    "patches": []
  },
  "s3_artifact": {
    "bucket": "...",
    "key": "..."
  }
}
```

</details>

### `GET /health`

```bash
curl http://localhost:8000/health
# → {"status": "ok"}
```

---

## ⚙️ Environment Variables

| Variable | Description | Default |
|---|---|---|
| `AWS_BEARER_TOKEN_BEDROCK` | AWS Bedrock authentication token | — |
| `GROQ_API_KEY` | Groq API key for fallback model | — |
| `AWS_REGION` | AWS region for services | `us-east-1` |
| `S3_BUCKET` | S3 bucket for scan artifacts | `blastshield-artifacts` |
| `EC2_SANDBOX_URL` | URL of the EC2 sandbox server | `http://localhost:9000` |

---

## 🔒 Security

- **Sandboxed execution** — User code runs inside Docker containers with a read-only filesystem and network disabled
- **No persistent state** — Containers are ephemeral and destroyed after each run
- **IAM least privilege** — Lambda and EC2 roles have minimal S3 and Bedrock permissions
- **CSP-restricted webviews** — Extension UI runs within Content Security Policy restrictions

---

## 🛠️ Tech Stack

| Layer | Technologies |
|---|---|
| **Extension** | TypeScript, React 18, React Flow, Chart.js, Vite, esbuild |
| **Backend API** | Python 3.13, FastAPI, Mangum, AWS Lambda |
| **Sandbox** | Docker, EC2, Groq AI |
| **AI Models** | Amazon Nova Pro, Anthropic Prompt Router, Groq GPT-OSS |
| **Storage** | AWS S3 |
| **Infrastructure** | AWS Lambda, API Gateway, EC2 |

---

## 📄 License

This project is open source.

---

<p align="center">
  <strong>Built with 🔥 by <a href="https://github.com/Deepesh1024">Deepesh Kumar Jha</a></strong><br/>
  <sub>Production breaks shouldn't be discovered in production.</sub>
</p>
