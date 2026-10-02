# Workflow 4: CI/CD and Cloud Run Deployment Pipeline

> **Figure 9.6 / Chapter 9.4** — Deploying multimodal agents to Google Cloud Run

This diagram shows the full deployment pipeline from a manual trigger in GitHub Actions to a live Cloud Run service, as implemented in `.github/workflows/deploy.yml`.

```mermaid
flowchart LR
    subgraph STAGE1 ["Stage 1: GitHub Actions CI/CD"]
        direction TB
        A["<div style='min-width: 300px;'><b>Workflow Dispatch Trigger</b><br/>Manual execution on main branch</div>"]
        B["<div style='min-width: 300px;'><b>Repository Checkout</b><br/>Pulls code & uv lockfile dependencies</div>"]
        C["<div style='min-width: 300px;'><b>GCP Secret Authentication</b><br/>Loads service account credentials</div>"]
        D["<div style='min-width: 300px;'><b>Container Build (AMD64)</b><br/>Multi-stage build with cached layers</div>"]
        E["<div style='min-width: 300px;'><b>Push Image to Registry</b><br/>Tagged with git SHA and latest</div>"]
        A --> B --> C --> D --> E
    end

    subgraph STAGE2 ["Stage 2: Cloud Run Deployment"]
        direction TB
        F["<div style='min-width: 300px;'><b>Artifact Registry Ingestion</b><br/>Stores production container image</div>"]
        G["<div style='min-width: 300px;'><b>Cloud Run Service Deploy</b><br/>gcloud run deploy with 2 GiB / 2 vCPU</div>"]
        H["<div style='min-width: 300px;'><b>Runtime Secret Injection</b><br/>Binds Gemini & Serper API keys</div>"]
        I["<div style='min-width: 300px;'><b>Public Application Live</b><br/>agri-assistant-*.run.app:8080 active</div>"]
        F --> G --> H --> I
    end

    STAGE1 ==>|"Automated CI/CD<br/>artifact handoff"| STAGE2

    classDef s1 fill:#EFF6FF,stroke:#2563EB,stroke-width:1.5px,color:#000000
    classDef s2 fill:#F0FDF4,stroke:#16A34A,stroke-width:1.5px,color:#000000
    classDef live fill:#DCFCE7,stroke:#15803D,stroke-width:1.5px,color:#000000
    classDef node fill:#FFFFFF,stroke:#4B5563,stroke-width:1px,color:#000000

    class STAGE1 s1
    class STAGE2 s2
    class I live
    class A,B,C,D,E,F,G,H node
```

## Stateless Deployment Considerations (Section 9.4)

Cloud Run runs the container as **stateless serverless**. Key design choices to handle this:

| Challenge | Solution in Plant Doctor |
|-----------|-------------------------|
| Session state lost between requests | Streamlit `session_state` held in-process per user session |
| Secrets not in env | Google Secret Manager injected at deploy time |
| Image size & cold start | `python:3.11-slim` base + layer caching via `pyproject.toml` and `uv.lock` copy |
| Multi-platform compatibility | `--platform linux/amd64` explicit build flag for Apple Silicon devs |
| Scale to zero | `--min-instances 0` keeps costs at $0 when idle |
