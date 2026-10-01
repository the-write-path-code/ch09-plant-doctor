# Workflow 2: MCP Server Architecture

> **Chapter 9.2** — Integrating specialist tools and external APIs via Model Context Protocol

This diagram shows how the agricultural tools are exposed via the Model Context Protocol (MCP), enabling both the internal Gemini agent and external clients (Claude Desktop, etc.) to discover and invoke the same tools.

```mermaid
flowchart TD
    subgraph CONSUMERS ["Invocation Surfaces & Protocol Adapters"]
        direction LR
        subgraph PATH_A ["Internal Streamlit Workflow"]
            direction TB
            C1["<div style='min-width: 340px;'><b>Plant Doctor Application</b><br/>Streamlit Frontend Web Interface</div>"]
            NAT["<div style='min-width: 340px;'><b>Gemini Native Calling</b><br/>Direct Python function dispatch</div>"]
            C1 --> NAT
        end
        subgraph PATH_B ["External Agent Workflow"]
            direction TB
            C2["<div style='min-width: 340px;'><b>Claude Desktop & MCP Hosts</b><br/>Standard Model Context Protocol Clients</div>"]
            MCP["<div style='min-width: 340px;'><b>FastMCP Protocol Server</b><br/>Streamable HTTP with <code>@mcp.tool</code></div>"]
            C2 --> MCP
        end
    end

    subgraph TOOLS ["Shared Tool Implementations & External APIs (agri_tools.py)"]
        T1["<div style='min-width: 170px;'><b>get_weather</b><br/>NOAA Weather API</div>"]
        T2["<div style='min-width: 170px;'><b>get_soil_type</b><br/>USDA Soil Database</div>"]
        T3["<div style='min-width: 170px;'><b>get_location</b><br/>Composite Location</div>"]
        T4["<div style='min-width: 170px;'><b>search_products</b><br/>Serper / Amazon Search</div>"]

        NAT --> T1 & T2 & T4
        MCP --> T1 & T2 & T3 & T4
    end

    classDef client fill:#F3F4F6,stroke:#4B5563,color:#000000,stroke-width:1.5px
    classDef nat fill:#EDE9FE,stroke:#7C3AED,color:#000000,stroke-width:1.5px
    classDef mcp fill:#FEF9C3,stroke:#CA8A04,color:#000000,stroke-width:1.5px
    classDef tool fill:#DCFCE7,stroke:#15803D,color:#000000,stroke-width:1.5px

    class C1,C2 client
    class NAT nat
    class MCP mcp
    class T1,T2,T3,T4 tool
```

## Dual Interface Pattern

The same tool implementations serve two interfaces:

| Interface | Used By | Discovery | Schema |
|-----------|---------|-----------|--------|
| Gemini Native Function Calling | Plant Doctor app internally | Python function signatures | Auto-inferred |
| MCP Protocol (FastMCP) | Claude Desktop, external agents | `@mcp.tool()` decorators | JSON Schema |

This demonstrates **Section 9.2's** principle: specialist tools should be written once and exposed through standardized boundaries so any compliant agent can consume them.
