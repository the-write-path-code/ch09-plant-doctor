# Workflow 1: Agentic Detection & Treatment Flow

> **Chapter 9.1 & 9.2** — Separating diagnosis from downstream action / Integrating specialist tools and external APIs

This diagram illustrates the end-to-end 3-stage agentic pipeline: from image upload through Gemini Vision detection to tool-calling for personalized treatment recommendations.

```mermaid
flowchart LR
    subgraph STAGE1 ["Stage 1: Diagnosis"]
        direction TB
        A["<div style='min-width: 300px;'><b>Leaf Image Upload</b><br/>User submits photo or selects sample</div>"]
        B["<div style='min-width: 300px;'><b>Gemini Vision Analysis</b><br/>Multimodal inspection & identification</div>"]
        C["<div style='min-width: 300px;'><b>Pathology Results</b><br/>Plant species, pest name & severity</div>"]
        D["<div style='min-width: 300px;'><b>Risk Assessment</b><br/>Detailed summary & damage evaluation</div>"]
        E["<div style='min-width: 300px;'><b>Context Halt Boundary</b><br/>System prompts for local ZIP code</div>"]
        A --> B --> C --> D --> E
    end

    subgraph STAGE2 ["Stage 2: Treatment Generation"]
        direction TB
        F["<div style='min-width: 300px;'><b>Environmental Telemetry</b><br/>NOAA weather & USDA soil data</div>"]
        G["<div style='min-width: 300px;'><b>Gemini Agent Synthesis</b><br/>Combines diagnosis with local context</div>"]
        H["<div style='min-width: 300px;'><b>Targeted Search</b><br/>Serper API finds organic remedies</div>"]
        I["<div style='min-width: 300px;'><b>Treatment Plan Delivered</b><br/>Actionable care advice & product links</div>"]
        F --> G --> H --> I
    end

    STAGE1 ==>|"User inputs local<br/>ZIP & infestation level"| STAGE2

    classDef s1 fill:#EDE9FE,stroke:#7C3AED,color:#000000,stroke-width:1.5px
    classDef s2 fill:#EBF5FF,stroke:#2563EB,color:#000000,stroke-width:1.5px
    classDef halt fill:#FEE2E2,stroke:#DC2626,color:#000000,stroke-width:1.5px
    classDef node fill:#FFFFFF,stroke:#4B5563,color:#000000,stroke-width:1px

    class STAGE1 s1
    class STAGE2 s2
    class E halt
    class A,B,C,D,F,G,H,I node
```

## Key Design Principles (Section 9.1)

- **Separation of concerns**: Detection (Gemini Vision) is decoupled from downstream tool-calling (Gemini Agent)
- **Deterministic first step**: Vision identification runs without tools — it cannot call external APIs
- **Tool calls on demand**: The agent autonomously decides when to call `get_weather`, `get_soil_type`, and `search_amazon_products`
- **State machine stages**: `upload → details → recommendations` prevents partial-state errors
