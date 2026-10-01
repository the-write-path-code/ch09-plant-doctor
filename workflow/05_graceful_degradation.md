# Workflow 5: Graceful Degradation Under Incomplete Context

> **Chapter 9.3** — Graceful degradation under incomplete context

This diagram shows how the Plant Doctor handles failures at each external dependency — weather API, soil API, and product search — without crashing or producing empty responses.

```mermaid
flowchart TD
    A(["<div style='min-width: 460px;'><b>Agentic Diagnostic Session</b><br/>ZIP Code & Infestation Level Provided</div>"])

    D{"Weather Telemetry<br/>Available?"}
    E{"Soil Telemetry<br/>Available?"}

    F["<div style='min-width: 190px;'><b>Weather Context</b><br/>Temp · Forecast · Wind</div>"]
    H["<div style='min-width: 190px;'><b>Generic Weather</b><br/>Standard Parameters</div>"]

    G["<div style='min-width: 190px;'><b>Soil Context</b><br/>Texture · pH · Drainage</div>"]
    I["<div style='min-width: 190px;'><b>Generic Soil</b><br/>Balanced Loam Default</div>"]

    J["<div style='min-width: 480px;'><b>Generate Treatment Plan</b><br/>Gemini Model Synthesizes Available Environmental Telemetry</div>"]

    L{"Product Search<br/>Available?"}

    M["<div style='min-width: 220px;'><b>Add Curated Product Links</b><br/>Verified Amazon Cards & Pricing</div>"]
    N["<div style='min-width: 220px;'><b>Dynamic Search Link Fallback</b><br/>Direct Amazon Query URL</div>"]

    O(["<div style='min-width: 480px;'><b>Complete Treatment Plan Delivered</b><br/>Actionable Advice with Explicit Telemetry Warning Tags</div>"])

    A --> D & E

    D -- Yes --> F --> J
    D -- "No / Timeout" --> H --> J

    E -- Yes --> G --> J
    E -- "No / Timeout" --> I --> J

    J --> L
    L -- Yes --> M --> O
    L -- "No / Rate Limit" --> N --> O

    classDef start fill:#F3F4F6,stroke:#4B5563,color:#000000,stroke-width:1.5px
    classDef decision fill:#EDE9FE,stroke:#7C3AED,color:#000000,stroke-width:1.5px
    classDef ok fill:#DCFCE7,stroke:#15803D,color:#000000,stroke-width:1.5px
    classDef fb fill:#FEF3C7,stroke:#D97706,color:#000000,stroke-width:1.5px
    classDef plan fill:#FEF9C3,stroke:#CA8A04,color:#000000,stroke-width:1.5px

    class A,O start
    class D,E,L decision
    class F,G,M ok
    class H,I,N fb
    class J plan
```

## Degradation Strategy

The system follows a **best-effort delivery** principle:

1. **API failures** return structured error dicts, not exceptions — the agent always receives *something*
2. **`format_*_display()` functions** check for the `"error"` key and return a human-readable warning string
3. **Gemini still generates** useful generic treatment advice even when location data is missing
4. **Product search fallback** builds an Amazon search URL from the query string, so the user always gets a clickable link
5. **No hard crashes** — `try/except` blocks at every external call boundary

This maps directly to **Section 9.3's** principle: an agent that silently degrades is safer than one that throws unhandled exceptions in production.
