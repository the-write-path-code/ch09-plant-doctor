# Chapter 9: Case Study, Context-Aware Actuation with Plant Doctor

Companion code for *Building Safe Agentic AI for Enterprise Systems* by Mohit Aggarwal.

Plant Doctor begins with a simple question, identify a pest or disease from a plant image, then follows the harder engineering question: what information does the system need before it can recommend an action? Image recognition alone cannot account for local weather, soil conditions, severity, product availability, or incomplete evidence.

This repository separates diagnosis from downstream action. A vision model identifies the likely pest or disease. A separate agent gathers bounded local context through specialist tools, combines the available evidence, and produces a treatment plan with product options. The application supports MCP exposure so the same specialist tools can be consumed through a standard boundary.

## What You Will Run

| Chapter section | Demonstration | What it shows |
| --- | --- | --- |
| 9.1 | Diagnosis separated from action | Vision-based identification occurs before treatment planning and does not make product or intervention decisions on its own. |
| 9.2 | Specialist tools and external APIs | Weather, soil, and product-retrieval tools supply local context to the treatment-planning stage. |
| 9.3 | Graceful degradation | The system distinguishes missing context from a confident treatment recommendation and gives a constrained response when evidence is incomplete. |
| 9.4 | Cloud Run deployment | Docker, Artifact Registry, GitHub Actions, Secret Manager, and Cloud Run deployment configuration. |
| 9.5 | Regulated-industry transfer | The same separation of recognition, contextual retrieval, tool use, and downstream action applies outside gardening. |

## Production Warning

Plant Doctor is a software architecture demonstration, not a replacement for agricultural extension advice, pesticide labeling, local regulation, or professional diagnosis. Product suggestions and treatment recommendations require a human review before they become a purchase, chemical application, or operational instruction.

The application uses external services for model inference, weather, soil information, and product search. Do not send sensitive images, personal location data, credentials, or protected business information to those services without a reviewed data-handling and egress policy.

## Prerequisites

- Git
- [uv](https://docs.astral.sh/uv/)
- Python 3.11
- A Google Gemini API key for image analysis and agent reasoning
- A Serper API key for product-search and web-search functions
- Optional: Docker for local container testing
- Optional: Google Cloud project, Artifact Registry access, and Cloud Run permissions for deployment

The local application requires the Gemini and Serper credentials. Do not expect a fully offline path for this repository.

## Quick Start

### 1. Install uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### 2. Clone and synchronize the repository

```bash
git clone https://github.com/the-write-path-code/ch09-plant-doctor.git
cd ch09-plant-doctor
uv sync
```

The repository includes `uv.lock`. Run `uv sync` after pulling changes so the local environment matches the committed dependency set.

### 3. Create local configuration

```bash
cp .env.example .env
```

Set the required values:

```dotenv
GOOGLE_API_KEY=your-google-gemini-api-key
SERPER_API_KEY=your-serper-api-key
```

Do not commit `.env`.

### 4. Run the local application

```bash
uv run streamlit run app.py
```

Streamlit prints the local URL, normally `http://localhost:8501`. Use a supplied sample image before uploading any of your own material.

## Configuration

| Variable | Required? | Purpose |
| --- | --- | --- |
| `GOOGLE_API_KEY` | Yes | Gemini vision analysis and agent reasoning |
| `SERPER_API_KEY` | Yes | Web and product-search functions |

The repository's environment file should contain only credentials and deployment-specific configuration. Do not place treatment policy, safety thresholds, search-result limits, or tool permissions in `.env`. Those controls belong in versioned, reviewed application configuration.

> **Tip**
>
> Use a supplied plant image for the first run. It isolates the application setup from the uncertainty of a new image, a new geography, or a product-search result that changes over time.

## Run the Chapter Demonstrations

### 1. Identify a Plant Problem, Section 9.1

Start the Streamlit application and upload a sample plant image or select one of the supplied examples.

The vision stage identifies the likely plant, pest or disease, and severity. Treat the output as an assessment input, not the final recommendation. The diagnosis stage has no direct route to product search, purchase, or any irreversible action.

### 2. Add Local Context, Section 9.2

Provide the location information the application requests. The specialist tools gather relevant context such as weather conditions and soil information, then return structured results to the treatment-planning stage.

The agent can use these tools to answer questions such as:

```text
What conditions favor this pest?
What treatment timing is appropriate for the current weather?
Which soil conditions should change the recommendation?
What products match the identified issue and treatment constraints?
```

The tool layer should limit what the agent can request and what each tool returns. Keep API keys, raw provider responses, and provider-specific request construction inside the tool implementation.

### 3. Inspect Graceful Degradation, Section 9.3

Run a case with incomplete local context, for example a missing or ambiguous location. The application should identify what it cannot determine and constrain its response rather than presenting an apparently complete recommendation.

A safe degraded result can provide general, supported guidance and identify the missing information needed for a location-aware recommendation. It must not invent weather, soil, product availability, or local regulatory facts.

### 4. Inspect the MCP Tool Boundary, Section 9.2

The agricultural tools are decorated for MCP exposure. This allows compliant clients to discover and invoke the same weather, soil, product, and location-context functions through a standardized interface.

Read the MCP workflow document before exposing the tools to another client. MCP schema validation constrains argument shape; it does not certify that an agricultural recommendation is safe, complete, or appropriate for a specific location.

### 5. Run the Container Locally, Optional

Build the image:

```bash
docker build -t plant-doctor-local .
```

Run the container with the local environment file:

```bash
docker run --rm -p 8080:8080 --env-file .env plant-doctor-local
```

Open the application at `http://localhost:8080`.

### 6. Deploy to Cloud Run, Section 9.4

The repository includes a manually triggered GitHub Actions deployment workflow. Before using it, create the required repository secrets:

| GitHub Actions secret | Purpose |
| --- | --- |
| `GCP_PROJECT_ID` | Google Cloud project identifier |
| `GCP_REGION` | Deployment region, for example `us-central1` |
| `GCP_SA_KEY` | Service-account credential used by the deployment workflow |

The workflow builds a Docker image, pushes it to Artifact Registry, deploys to Cloud Run, and supplies the Gemini and Serper values through Google Secret Manager.

> **Production Warning**
>
> The current Cloud Run workflow uses `--allow-unauthenticated`. Do not deploy a write-capable, image-uploading, or externally connected application publicly without reviewing authentication, abuse controls, rate limits, logging, data retention, secret access, and egress permissions. Change the Cloud Run access policy to match the intended audience before deployment.

## Expected Results

A normal local run should show four separate stages:

1. An image-based diagnosis.
2. A request for local context or a clearly stated context gap.
3. Specialist-tool results for weather, soil, and product retrieval where available.
4. A treatment plan that ties its advice to the available evidence and identifies missing context.

The output may vary because external model and search services change over time. The architecture should remain stable: diagnosis stays separate from contextual retrieval; specialist tools provide structured information; and incomplete context leads to an explicit limitation rather than a fabricated recommendation.

## Run the Tests

Run the repository tests:

```bash
uv run pytest
```

Run the tests before changing image handling, tool schemas, local-context retrieval, degradation logic, MCP exposure, or deployment configuration. The tests should establish that:

- Diagnosis and downstream recommendation are separate steps.
- Tool calls return the documented result shapes.
- Missing context produces a defined degraded path.
- The agent cannot bypass the registered tool boundary.
- MCP tool registrations expose only the intended tool set.

## Repository Layout

```text
.
├── README.md
├── pyproject.toml
├── uv.lock
├── .env.example
├── Dockerfile
├── app.py                              # Streamlit application entry point
├── src/
│   ├── plant_pest_detector.py          # Gemini vision diagnosis
│   ├── qa_engine_agentic.py            # Context-aware agent and treatment planning
│   ├── location_service.py             # Weather and soil context
│   ├── agri_tools.py                   # Specialist tool definitions and MCP exposure
│   └── mcp_server.py                   # MCP server configuration, if present
├── samples/                             # Plant images for local demonstration
├── evaluation/                          # Benchmark queries and agent-flow tests
├── workflow/
│   ├── 01_agentic_detection_flow.md
│   ├── 02_mcp_architecture.md
│   ├── 03_tool_calling_sequence.md
│   ├── 04_cloud_deployment.md
│   ├── 05_graceful_degradation.md
│   └── 06_deployment_tutorial.md
└── .github/workflows/
    └── deploy.yml                       # Manual Cloud Run deployment workflow
```

## Architecture Diagrams and Supporting Documents

The `workflow/` directory contains the Chapter 9 figures:

- Diagnosis and location-aware treatment planning.
- MCP exposure of specialist agricultural tools.
- Tool-calling sequence and return values.
- GitHub Actions, Artifact Registry, and Cloud Run deployment.
- Graceful degradation when local context is incomplete.
- A step-by-step Cloud Run deployment guide.

Start with `01_agentic_detection_flow.md`. It shows the design boundary that governs the repository: image recognition produces a diagnosis; local context and specialist tools support treatment planning; neither stage should take an irreversible action on its own.

## Safety and Operational Limits

- Model output and product-search results are advisory. A reader must verify treatment suitability, product labels, local restrictions, and physical conditions before taking action.
- A diagnosis can be wrong, incomplete, or unable to distinguish visually similar problems. Do not treat confidence language as professional certification.
- Weather, soil, and product data can be unavailable, stale, incomplete, or mismatched to the user's exact location. Missing context must remain visible.
- MCP exposes a controlled interface but does not supply authorization, idempotency, approval, or transaction controls.
- The public Cloud Run configuration must be reviewed before any deployment beyond a limited demonstration.
- Do not commit API keys, Google Cloud service-account credentials, user-uploaded images, or production logs.

## Troubleshooting

### The Streamlit application does not start

Synchronize the project environment and run the application from the repository root:

```bash
uv sync
uv run streamlit run app.py
```

### Image analysis fails

Confirm that `.env` exists, `GOOGLE_API_KEY` is configured, and the selected Gemini model is available to the key. Start with a supplied sample image before troubleshooting a new image.

### Product search or web fallback fails

Confirm that `SERPER_API_KEY` is set and that the service is reachable. Do not replace a missing search result with a fabricated product recommendation.

### The result lacks local weather or soil context

Check the location input and the tool responses. Return a constrained result when location resolution fails. Do not have the agent infer local conditions from the image alone.

### The Cloud Run deployment fails

Check the Actions workflow log, Google Cloud project and region values, Artifact Registry permissions, service-account permissions, Secret Manager versions, and Cloud Run service settings. Test the Docker image locally before changing cloud configuration.

## Related Chapters

- Chapter 6 builds the typed multimodal-perception and handoff patterns that support image-based extraction.
- Chapter 7 shows how location-aware systems preserve useful geography while limiting sensitive location data in the agent layer.
- Chapter 8 establishes the MCP boundary for specialist tools and external systems.
- Chapter 10 explains why a successful local service needs durable state and explicit cloud assumptions before deployment.
- Chapters 11 through 14 show the controls required when an agent moves from advising to consequential writes or actions.
- Chapter 15 tests model behavior, tools, and safety gates continuously after deployment.

## License and Errata

See `LICENSE` for licensing terms. Report documentation or code issues through this repository's GitHub issue tracker.
