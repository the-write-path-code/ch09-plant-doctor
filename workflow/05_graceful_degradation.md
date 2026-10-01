# Workflow 5: Graceful Degradation Under Incomplete Context

> **Chapter 9.3** — Graceful degradation under incomplete context

This diagram shows how the Plant Doctor handles failures at each external dependency — weather API, soil API, and product search — without crashing or producing empty responses.

```mermaid
flowchart TD
    subgraph STACK ["Graceful Degradation Architecture (Zero Unhandled Exceptions)"]
        direction TB

        S1["<div style='min-width: 750px;'><b>Phase 1 — Input & Coordinate Validation:</b> User submits ZIP code & infestation level<br/>➔ Validates geographical coordinates; prevents unhandled exceptions before external dispatch</div>"]

        S2["<div style='min-width: 750px;'><b>Phase 2 — Resilient Weather Lookup (NOAA api.weather.gov):</b><br/>• <b>Healthy Path:</b> Ingests real-time temperature, wind velocity, and 7-day precipitation forecast<br/>• <b>Degraded Fallback:</b> On HTTP timeout/503, returns standard regional climate baseline with warning flag</div>"]

        S3["<div style='min-width: 750px;'><b>Phase 3 — Resilient Soil Lookup (USDA sdmdataaccess.nrcs.usda.gov):</b><br/>• <b>Healthy Path:</b> Retrieves SSURGO soil texture, pH balance, and drainage classification<br/>• <b>Degraded Fallback:</b> On query timeout/empty grid, falls back to balanced loam defaults with warning flag</div>"]

        S4["<div style='min-width: 750px;'><b>Phase 4 — Resilient Product Retrieval (Serper Search API ➔ Amazon):</b><br/>• <b>Healthy Path:</b> Fetches curated Amazon product cards with verified pricing and direct affiliate links<br/>• <b>Degraded Fallback:</b> On API quota limit/error, dynamically synthesizes targeted Amazon search URL</div>"]

        S5["<div style='min-width: 750px;'><b>Phase 5 — Synthesis & Actionable Delivery:</b> Gemini model generates treatment plan<br/>➔ Best-effort execution succeeds; UI surfaces human-readable advisories detailing any fallback data</div>"]

        S1 ==> S2 ==> S3 ==> S4 ==> S5
    end

    classDef stack fill:#FFFFFF,stroke:#2563EB,color:#000000,stroke-width:2px
    classDef s1 fill:#F3F4F6,stroke:#4B5563,color:#000000,stroke-width:1.5px
    classDef s2 fill:#EBF5FF,stroke:#2563EB,color:#000000,stroke-width:1.5px
    classDef s3 fill:#FEF9C3,stroke:#CA8A04,color:#000000,stroke-width:1.5px
    classDef s4 fill:#EDE9FE,stroke:#7C3AED,color:#000000,stroke-width:1.5px
    classDef s5 fill:#DCFCE7,stroke:#15803D,color:#000000,stroke-width:1.5px

    class STACK stack
    class S1 s1
    class S2 s2
    class S3 s3
    class S4 s4
    class S5 s5
```

## Degradation Strategy

The system follows a **best-effort delivery** principle:

1. **API failures** return structured error dicts, not exceptions — the agent always receives *something*
2. **`format_*_display()` functions** check for the `"error"` key and return a human-readable warning string
3. **Gemini still generates** useful generic treatment advice even when location data is missing
4. **Product search fallback** builds an Amazon search URL from the query string, so the user always gets a clickable link
5. **No hard crashes** — `try/except` blocks at every external call boundary

This maps directly to **Section 9.3's** principle: an agent that silently degrades is safer than one that throws unhandled exceptions in production.
