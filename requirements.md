# Requirements Specification

## Project Overview

BlastShield is an end-to-end production failure simulation platform that enables developers to stress-test Python projects under realistic production conditions before deployment. The system combines a serverless AI-powered backend (AWS Lambda + EC2 sandbox) with a rich VS Code extension to deliver an SRE-grade "shift-left" testing experience directly inside the IDE.

The platform simulates production traffic patterns, executes code in isolated Docker containers, and generates detailed postmortem reports with failure timelines, blast radius analysis, and AI-generated patches.

## Problem Statement

Modern software teams face a critical gap in their testing pipeline: **production failures that are invisible during development**. Traditional testing tools catch syntax errors and logic bugs, but they miss the systemic failures that only surface under real-world conditions:

- **Race conditions** that emerge under concurrent load
- **Cascading timeouts** triggered by latency spikes
- **Memory leaks** that appear only after sustained traffic
- **Edge cases** that slip through unit tests
- **Uncaught exceptions** in async flows
- **Resource exhaustion** under stress

These failures are discovered in production at 2 AM, causing outages, data loss, and customer impact. BlastShield eliminates this guesswork by bringing production-grade stress testing into the development workflow.

## Target Users

### Primary Users
- **Backend Developers** building Python web services (FastAPI, Flask, Django)
- **DevOps Engineers** responsible for deployment reliability
- **SRE Teams** conducting chaos engineering and failure analysis
- **Technical Leads** reviewing code for production readiness

### Secondary Users
- **QA Engineers** expanding test coverage beyond functional tests
- **Engineering Students** learning about production failure modes
- **Open Source Maintainers** validating library reliability under stress

## User Stories

### Core Simulation Workflow
- As a **backend developer**, I want to simulate production traffic against my local code so that I can identify race conditions before deployment.
- As an **SRE engineer**, I want to inject latency and chaos into my system so that I can validate resilience under degraded conditions.
- As a **DevOps engineer**, I want to run edge case tests automatically so that I can catch boundary condition failures early.

### Observability & Analysis
- As a **developer**, I want to see a visual failure propagation map so that I can understand how errors cascade through my system.
- As a **technical lead**, I want an AI-generated postmortem report so that I can quickly understand root causes without manual analysis.
- As an **SRE**, I want to replay the incident timeline step-by-step so that I can identify the exact sequence of events leading to failure.

### Remediation & Iteration
- As a **developer**, I want AI-generated patches with diff previews so that I can fix issues without manual code rewriting.
- As a **QA engineer**, I want to adjust traffic and latency parameters so that I can test different production scenarios.
- As a **team lead**, I want to run simulations in CI/CD so that PRs are automatically validated for production readiness.

### Demo & Education
- As a **hackathon judge**, I want to see the tool work offline with demo data so that I can evaluate it without backend dependencies.
- As a **student**, I want to learn about production failure modes so that I can build more reliable systems.

## Functional Requirements

### FR-1: Workspace Analysis
- **FR-1.1**: Accept Python projects as zip files or code strings
- **FR-1.2**: Parse AST to detect FastAPI/Flask endpoints
- **FR-1.3**: Build call graphs and interaction maps between modules
- **FR-1.4**: Identify async functions, database calls, and external dependencies

### FR-2: Production Simulation Drills
- **FR-2.1**: Execute concurrency drills (10-100 concurrent requests)
- **FR-2.2**: Inject artificial latency (50ms-5000ms delays)
- **FR-2.3**: Perform chaos engineering (random failures, timeouts, exceptions)
- **FR-2.4**: Generate and test edge cases (null inputs, boundary values, malformed data)
- **FR-2.5**: Simulate load tests with curl-style HTTP requests

### FR-3: Sandboxed Execution
- **FR-3.1**: Upload project artifacts to S3 for persistence
- **FR-3.2**: Execute code in Docker containers with security constraints:
  - Read-only filesystem
  - No network access
  - CPU limit: 1 core
  - Memory limit: 512MB
  - PID limit: 64 processes
  - Timeout: configurable (default 10s)
- **FR-3.3**: Capture runtime errors, logs, and exit codes
- **FR-3.4**: Validate deployment success/failure status

### FR-4: AI-Powered Analysis
- **FR-4.1**: Generate failure timelines with event sequences
- **FR-4.2**: Identify root causes using multi-model AI cascade:
  - Primary: Amazon Nova Pro (Bedrock)
  - Fallback: Anthropic Prompt Router
  - Final fallback: Groq GPT-OSS
- **FR-4.3**: Calculate blast radius (affected services/modules)
- **FR-4.4**: Produce postmortem reports with:
  - Executive summary
  - Root cause analysis
  - Impact assessment
  - Timeline narrative
  - Remediation steps
- **FR-4.5**: Generate code patches with before/after diffs

### FR-5: VS Code Extension
- **FR-5.1**: Provide command palette trigger: "BlastShield: Run Production Simulation"
- **FR-5.2**: Zip workspace automatically (exclude node_modules, .git, etc.)
- **FR-5.3**: Display full-screen observability dashboard with:
  - Risk score (0-100)
  - Severity banner (LOW/MEDIUM/HIGH/CRITICAL)
  - Evidence metrics (lost updates, timeouts, exceptions)
  - Network activity charts (Chart.js)
  - Failure propagation graph (React Flow)
  - Timeline replay with step-by-step playback
  - Logs viewer with severity filtering
  - Root cause analysis
  - Patch viewer with diff highlighting
- **FR-5.4**: Support "What-If" simulations (adjust traffic, latency, failure rate)
- **FR-5.5**: Provide demo mode with realistic mock data for offline use

### FR-6: Demo Sandbox Environment
- **FR-6.1**: Dynamically provision Docker containers running code-server (VS Code in browser)
- **FR-6.2**: Pre-configure sample projects (CacheStorm, CartRush, PayFlow, QueryBurst)
- **FR-6.3**: Auto-install BlastShield extension in containerized environments
- **FR-6.4**: Manage port allocation (9000-9100) for isolated environments
- **FR-6.5**: Provide zero-setup browser-based IDE access

## Non-Functional Requirements

### NFR-1: Performance
- **NFR-1.1**: Complete full simulation pipeline in under 25 seconds
- **NFR-1.2**: Support projects up to 50MB compressed size
- **NFR-1.3**: Handle concurrent simulations via Lambda auto-scaling
- **NFR-1.4**: Respond to health checks within 100ms

### NFR-2: Scalability
- **NFR-2.1**: Scale Lambda functions to handle 1000+ concurrent scans
- **NFR-2.2**: Support EC2 sandbox horizontal scaling for high throughput
- **NFR-2.3**: Use S3 for artifact storage with unlimited capacity
- **NFR-2.4**: Implement exponential backoff for API retries (3 attempts, 3s/6s delays)

### NFR-3: Security
- **NFR-3.1**: Execute user code in isolated Docker containers with no network access
- **NFR-3.2**: Use read-only filesystems to prevent malicious writes
- **NFR-3.3**: Enforce resource limits (CPU, memory, PIDs) to prevent DoS
- **NFR-3.4**: Apply IAM least-privilege policies for Lambda and EC2 roles
- **NFR-3.5**: Destroy containers immediately after execution (ephemeral)
- **NFR-3.6**: Use CSP-restricted webviews in VS Code extension

### NFR-4: Reliability
- **NFR-4.1**: Implement AI model cascade with 4-tier fallback strategy
- **NFR-4.2**: Provide static fallback responses when all AI models fail
- **NFR-4.3**: Handle Lambda cold starts gracefully with retry logic
- **NFR-4.4**: Support offline demo mode when backend is unavailable
- **NFR-4.5**: Log all errors with structured logging for debugging

### NFR-5: Usability
- **NFR-5.1**: Provide single-command execution from VS Code Command Palette
- **NFR-5.2**: Display real-time progress notifications during scan
- **NFR-5.3**: Use intuitive visual representations (graphs, charts, timelines)
- **NFR-5.4**: Support interactive "What-If" scenario testing
- **NFR-5.5**: Include comprehensive error messages with actionable guidance

### NFR-6: Cost Efficiency
- **NFR-6.1**: Use serverless Lambda to eliminate idle infrastructure costs
- **NFR-6.2**: Leverage S3 for cheap artifact storage ($0.023/GB/month)
- **NFR-6.3**: Optimize Docker image size for faster cold starts
- **NFR-6.4**: Implement request batching where possible

### NFR-7: Maintainability
- **NFR-7.1**: Use TypeScript for type-safe extension code
- **NFR-7.2**: Modularize backend into clear layers (drills, AI, infrastructure)
- **NFR-7.3**: Provide comprehensive inline documentation
- **NFR-7.4**: Use environment variables for all configuration
- **NFR-7.5**: Support local development without AWS dependencies

## Success Criteria

### Technical Success Metrics
- ✅ Simulation completes in under 25 seconds for projects <10MB
- ✅ Risk score calculation accuracy validated against known failure patterns
- ✅ Docker sandbox successfully isolates and executes untrusted code
- ✅ AI analysis generates actionable postmortem reports in >90% of cases
- ✅ Extension works offline with demo data when backend unavailable
- ✅ Zero security incidents from sandboxed code execution

### User Experience Metrics
- ✅ Single-command workflow from VS Code Command Palette
- ✅ Visual dashboard loads within 2 seconds of scan completion
- ✅ Failure propagation graph renders correctly for projects with 10+ files
- ✅ Patch diffs are syntactically valid and applicable
- ✅ Demo mode provides realistic experience without backend setup

### Business Success Metrics
- ✅ Successfully demonstrates value at AWS Nationwide Hackathon
- ✅ Receives positive feedback from judges and developers
- ✅ Generates interest for open-source adoption
- ✅ Validates "shift-left SRE" concept for developer productivity

### Deployment Success Metrics
- ✅ Lambda functions deploy successfully via Mangum adapter
- ✅ EC2 sandbox server runs stably for 24+ hours
- ✅ VS Code extension packages as .vsix and installs without errors
- ✅ Demo sandbox provisions containers within 30 seconds
- ✅ All components integrate end-to-end without manual intervention
