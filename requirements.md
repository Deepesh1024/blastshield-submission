# Requirements Specification: BlastShield

## 1. Problem Statement

### Target Audience

Developers, students, and engineering teams building backend services who want to detect production failures before deployment.

### The Gap

Code that works locally often fails in production due to:

- Concurrency race conditions from shared mutable state
- Retry storms triggered by upstream timeouts
- Cascading latency under concurrent load
- Silent dependency failures (database, cache, external APIs)
- Runtime exceptions in untested edge cases

Traditional linters and static analysis tools cannot simulate real production behavior. They inspect code structure — not runtime system dynamics.

BlastShield solves this by running **realistic production stress simulations** against developer projects before any deployment occurs.

---

## 2. Functional Requirements

### FR-1: Workspace Analysis

System must accept Python projects as:
- Inline code strings (`application/json`)
- Compressed project archives (`multipart/form-data` `.zip` upload)

System must parse uploaded code to detect:
- FastAPI and Flask API route definitions
- Async function declarations
- Database and external service call sites
- Module-level import graphs and call chains

### FR-2: Production Failure Simulation

System must execute the following simulation drills against extracted code evidence:

| Drill | Description |
|---|---|
| **Concurrency Drill** | Simulate 10–100 concurrent requests; detect race conditions and shared state mutations |
| **Latency Injection** | Inject 50ms–5000ms artificial delays; expose timeout misconfigurations and cascading failures |
| **Chaos Engineering** | Random fault injection (network drops, exceptions, partial failures); validate resilience |
| **Edge Case Testing** | Generate null, empty, boundary, and malformed inputs; validate input sanitisation |
| **Load Testing** | Simulate HTTP request patterns across detected endpoints; measure error rates |

### FR-3: Sandboxed Runtime Execution

System must execute user code in an isolated Docker container with enforced constraints:

- Read-only container filesystem
- No external network access (`--network=none`)
- CPU limit: 1 core (`--cpus=1`)
- Memory limit: 512 MB (`--memory=512m`)
- Process limit: 64 PIDs (`--pids-limit=64`)
- Configurable execution timeout (default: 10 seconds)
- Ephemeral containers destroyed after each run (`--rm`)

System must capture:
- Deployment status (`success` | `failure` | `timeout`)
- Runtime exceptions with full stack traces
- Container stdout and stderr logs
- Container exit code

### FR-4: AI-Powered Failure Analysis

System must generate structured production failure reports using a multi-model AI cascade:

1. **Primary**: Amazon Nova Pro (Amazon Bedrock)
2. **Fallback**: Anthropic Prompt Router (Amazon Bedrock)
3. **Final Fallback**: Groq GPT-OSS (`openai/gpt-oss-120b`)
4. **Safety Net**: Static structured response

Each report must include:

- **Production risk score** (0–100, deterministic)
- **Failure timeline** with event sequence narrative
- **Root cause analysis** with technical explanation
- **Blast radius assessment** (affected modules and services)
- **Recommended patches** with before/after code diffs

### FR-5: Developer Integration — VS Code Extension

System must integrate directly into the developer workflow via a VS Code Extension:

- Single command trigger: `BlastShield: Run Production Simulation` (Command Palette)
- Automatic workspace compression (excludes `node_modules`, `.git`, `__pycache__`, build artefacts)
- Full-screen observability dashboard rendered in a VS Code Webview, including:
  - Risk score (0–100) and severity banner (LOW / MEDIUM / HIGH / CRITICAL)
  - Failure evidence metrics
  - Network traffic charts (Chart.js)
  - Failure propagation map (React Flow interactive graph)
  - Incident replay timeline (step-by-step)
  - Severity-filtered log viewer
  - Root cause and patch diff viewer
- **What-If Simulation**: Adjust traffic load, latency, and fault injection to re-test scenarios interactively
- **Offline Demo Mode**: Realistic mock data when backend is unavailable

---

## 3. User Stories

**As a developer**
I want to simulate production failures before deployment
so that I can detect reliability issues early, not at 2 AM.

**As a student learning backend engineering**
I want to see how real production failures propagate
so that I can understand production-grade system dynamics.

**As a startup engineering team**
I want to test system behaviour under concurrent load
so that we can avoid outages after launching to real users.

**As a DevOps engineer**
I want automated production readiness signals
so that unsafe deployments can be identified before they reach production.

**As an SRE practitioner**
I want evidence-backed postmortem reports with blast radius analysis
so that I can understand the full scope of a failure scenario.

---

## 4. Non-Functional Requirements

### NFR-1: Performance
- Full simulation pipeline must complete within **25 seconds** for projects under 10 MB
- Health check endpoint must respond within **100 ms**
- Dashboard must render within **2 seconds** of scan completion

### NFR-2: Scalability
- Lambda functions must scale to **1,000+ concurrent scan invocations**
- S3 artifact storage must support **unlimited project archive capacity**
- EC2 sandbox must support **horizontal scaling** via additional container instances

### NFR-3: Security
- User code must never execute on Lambda directly — always inside an isolated Docker container on EC2
- IAM policies must follow **least-privilege principles** for Lambda and EC2 roles
- VS Code Webview must enforce **Content Security Policy (CSP)** restrictions
- Sensitive credentials must never be committed to version control (`.env` pattern)

### NFR-4: Reliability
- System must return partial results (drill evidence) even when AI analysis fails
- All AI calls must follow the 4-tier cascade with **graceful degradation**
- Extension must support **offline demo mode** when backend is unavailable
- Extension must retry failed API requests with **exponential backoff** (3 attempts, 3s / 6s intervals)

### NFR-5: Cost Efficiency
- Serverless Lambda eliminates idle infrastructure cost
- S3 provides cost-effective artifact storage at **\$0.023/GB/month**
- EC2 sandbox instance requires only a `t3.small` — minimal fixed cost

---

## 5. Success Metrics

### Reliability Impact
BlastShield identifies production failure risks before code is deployed, shifting the discovery window from post-incident to pre-deployment.

### Developer Productivity
Structured reports with root cause explanations and ready-to-apply patches reduce the time from failure discovery to remediation.

### Learning Impact
Developers and students gain exposure to real production failure patterns — concurrency races, latency cascades, and blast radius propagation — without ever deploying to a live environment.

---

## 6. Unique Value Proposition

BlastShield is not a static analyser.

It is a **production failure simulator** that:

- Executes code in realistic, isolated runtime conditions
- Generates evidence-first failure timelines with causal chains
- Models blast radius across module boundaries
- Explains root causes in plain English with technical precision
- Suggests safe, context-aware patches

> **If static analysis is a spell-checker, BlastShield is a crash test dummy for your production system.**
