# Workflow 4: CI/CD and Cloud Run Deployment Pipeline

> **Chapter 9.4** — Deploying multimodal agents to Google Cloud Run

This diagram shows the full deployment pipeline from a manual trigger in GitHub Actions to a live Cloud Run service, as implemented in `.github/workflows/deploy.yml`.

```mermaid
%%{init: {"theme": "neutral", "themeVariables": {"fontFamily": "Arial, Helvetica, sans-serif", "fontSize": "11px", "actorFontSize": "11px", "noteFontSize": "10px", "messageFontSize": "10px"}}}%%
flowchart LR
    A(["👨💻 Developer<br/>Run Deployment"])

    subgraph GHA["1. GitHub Actions — CI/CD"]
        direction TB
        B["Workflow Dispatch"]
        C["Checkout Code"]
        D["Authenticate with GCP<br/>Using GitHub Secrets"]
        E["Configure Cloud SDK<br/>and Artifact Registry"]
        F["Build Container Image<br/>AMD64"]
        G["Push Image<br/>SHA Tag + Latest"]

        S["Required GitHub Secrets<br/>Project ID · Region · Service Account Key"]

        B --> C --> D --> E --> F --> G
        S -.-> D
    end

    subgraph GCP["2. Google Cloud Platform"]
        direction TB
        H["Artifact Registry<br/>Store Container Image"]
        I["Deploy Service<br/>to Cloud Run"]
        J["Inject Runtime Secrets<br/>Google API Key · Serper API Key"]
        K["Cloud Run Service<br/>2 GiB · 2 CPU · 300 s Timeout<br/>0–10 Instances"]

        H --> I --> J --> K
    end

    M(["🌐 Public Application URL<br/>agri-assistant-*.run.app"])

    A --> GHA
    GHA --> GCP
    GCP --> M

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
