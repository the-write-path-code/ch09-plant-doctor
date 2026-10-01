# Workflow 4: CI/CD and Cloud Run Deployment Pipeline

> **Chapter 9.4** — Deploying multimodal agents to Google Cloud Run

This diagram shows the full deployment pipeline from a manual trigger in GitHub Actions to a live Cloud Run service, as implemented in `.github/workflows/deploy.yml`.

```mermaid
flowchart TD
    subgraph STACK ["Cloud Deployment & Production Architecture (.github/workflows/deploy.yml)"]
        direction TB

        S1["<div style='min-width: 750px;'><b>Step 1 — Deployment Trigger:</b> Developer invokes GitHub Actions via <code>workflow_dispatch</code><br/>➔ Triggers automated CI/CD pipeline on the <code>main</code> branch with full parameter audit</div>"]

        S2["<div style='min-width: 750px;'><b>Step 2 — Secure GCP Authentication:</b> GitHub Actions authenticates with Google Cloud<br/>➔ Loads <code>GCP_PROJECT_ID</code> & <code>GCP_SA_KEY</code> secrets; configures <code>gcloud</code> CLI & Artifact Registry</div>"]

        S3["<div style='min-width: 750px;'><b>Step 3 — Multi-Stage Container Build:</b> Builds Docker container for <code>linux/amd64</code><br/>➔ <code>python:3.11-slim</code> base + <code>uv.lock</code> layer caching; tags and pushes to Artifact Registry</div>"]

        S4["<div style='min-width: 750px;'><b>Step 4 — Cloud Run Service Deployment:</b> Deploys container via <code>gcloud run deploy</code><br/>➔ Enforces 2 GiB RAM, 2 vCPU, 300s timeout, and 0–10 autoscaling instances (scale-to-zero)</div>"]

        S5["<div style='min-width: 750px;'><b>Step 5 — Runtime Secret Injection:</b> Injects API credentials at runtime startup<br/>➔ <code>GOOGLE_API_KEY</code> & <code>SERPER_API_KEY</code> bound directly to the live Cloud Run container</div>"]

        S6["<div style='min-width: 750px;'><b>Step 6 — Live Public Application URL:</b> Streamlit healthcheck confirms readiness<br/>➔ Container listens on <code>0.0.0.0:8080</code>; live at <code>https://agri-assistant-*.run.app</code></div>"]

        S1 ==> S2 ==> S3 ==> S4 ==> S5 ==> S6
    end

    classDef stack fill:#FFFFFF,stroke:#2563EB,color:#000000,stroke-width:2px
    classDef s1 fill:#F3F4F6,stroke:#4B5563,color:#000000,stroke-width:1.5px
    classDef s2 fill:#EDE9FE,stroke:#7C3AED,color:#000000,stroke-width:1.5px
    classDef s3 fill:#EBF5FF,stroke:#2563EB,color:#000000,stroke-width:1.5px
    classDef s4 fill:#FEF9C3,stroke:#CA8A04,color:#000000,stroke-width:1.5px
    classDef s5 fill:#FEF3C7,stroke:#D97706,color:#000000,stroke-width:1.5px
    classDef s6 fill:#DCFCE7,stroke:#15803D,color:#000000,stroke-width:1.5px

    class STACK stack
    class S1 s1
    class S2 s2
    class S3 s3
    class S4 s4
    class S5 s5
    class S6 s6
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
