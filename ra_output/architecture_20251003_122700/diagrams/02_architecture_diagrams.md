# Architecture Diagrams

## Overview

This document provides comprehensive architectural visualizations of the Lila MCP system, showing system layers, component relationships, class hierarchies, and module dependencies. These diagrams help understand the overall system structure and how different components interact.

---

## System Architecture

```mermaid
graph TB
    subgraph "Presentation Layer"
        CLI[CLI Entry Points]
        HTTP[HTTP Health Check]
    end

    subgraph "Orchestration Layer"
        BaseOrch[BaseOrchestrator<br/>Abstract Base Class]
        ArchOrch[ArchitectureOrchestrator]
        UXOrch[UXOrchestrator]
        CrossComm[CrossOrchestratorCommunication<br/>Mixin]
    end

    subgraph "Agent Management Layer"
        AgentReg[AgentRegistry<br/>Agent Discovery & Loading]
        AgentDef[AgentDefinition<br/>from claude_agent_sdk]
        AgentJSON[Agent JSON Files<br/>agents/*/]
    end

    subgraph "Tool Integration Layer"
        MCPReg[MCPRegistry<br/>MCP Server Management]
        FigmaInt[FigmaIntegration<br/>Design Tools]
        SDKTools[Claude SDK Tools<br/>Read, Write, Grep, Glob, Bash]
    end

    subgraph "MCP Server Layer"
        LilaMCP[LilaMCPServer<br/>Neo4j Backend]
        SimpleMCP[SimpleLilaMCPServer<br/>Mock Data]
        FastMCPCore[FastMCP Framework]
    end

    subgraph "Data Layer"
        Neo4jDB[(Neo4j Database<br/>Personas, Relationships)]
        MockData[In-Memory Mock Data]
        Importer[Neo4jDataImporter]
        Exporter[Neo4jDataExporter]
    end

    %% Connections
    CLI --> ArchOrch
    CLI --> UXOrch
    HTTP --> LilaMCP

    ArchOrch --> BaseOrch
    UXOrch --> BaseOrch
    BaseOrch --> AgentReg
    BaseOrch --> MCPReg

    AgentReg --> AgentJSON
    AgentReg --> AgentDef
    MCPReg --> FigmaInt

    ArchOrch --> SDKTools
    UXOrch --> SDKTools

    LilaMCP --> FastMCPCore
    SimpleMCP --> FastMCPCore
    LilaMCP --> Neo4jDB
    SimpleMCP --> MockData

    Importer --> Neo4jDB
    Exporter --> Neo4jDB

    style BaseOrch fill:#e1f5ff,stroke:#01579b
    style LilaMCP fill:#fff3e0,stroke:#e65100
    style Neo4jDB fill:#f3e5f5,stroke:#4a148c
    style AgentReg fill:#e8f5e9,stroke:#1b5e20
```

### Explanation

The system follows a **layered architecture** pattern with clear separation of concerns:

1. **Presentation Layer**: Entry points for CLI commands and HTTP health checks
2. **Orchestration Layer**: Domain-specific orchestrators that coordinate multi-phase workflows
3. **Agent Management Layer**: Registry system for discovering and loading agent definitions
4. **Tool Integration Layer**: Integration with MCP servers and Claude SDK tools
5. **MCP Server Layer**: FastMCP-based servers providing psychological intelligence services
6. **Data Layer**: Neo4j database for persistent storage and in-memory mock data for testing

This architecture enables:
- **Modularity**: Each layer has specific responsibilities
- **Extensibility**: New orchestrators can be added by extending BaseOrchestrator
- **Testability**: Mock data layer allows testing without database dependencies
- **Scalability**: Stateless orchestrators can run in parallel

---

## Component Relationships

```mermaid
graph LR
    subgraph "Architecture Analysis Domain"
        AO[ArchitectureOrchestrator]
        Analyzer[Analyzer Agent]
        DocWriter[Doc Writer Agent]
    end

    subgraph "UX Design Domain"
        UO[UXOrchestrator]
        UXResearcher[UX Researcher Agent]
        IAArchitect[IA Architect Agent]
        UIDesigner[UI Designer Agent]
        ProtoDev[Prototype Developer Agent]
    end

    subgraph "Orchestrator Framework"
        BO[BaseOrchestrator]
        PhaseExec[Phase Execution]
        CostTrack[Cost Tracking]
        OutputVerify[Output Verification]
    end

    subgraph "Agent System"
        AR[AgentRegistry]
        AD[AgentDefinition]
        AgentCache[Agent Cache]
    end

    subgraph "Tool System"
        MCP[MCPRegistry]
        Figma[FigmaIntegration]
        ToolList[Allowed Tools List]
    end

    subgraph "MCP Servers"
        Lila[LilaMCPServer]
        Simple[SimpleLilaMCPServer]
        Resources[Resources: Personas, Relationships]
        Tools[Tools: update_metrics, record_interaction]
        Prompts[Prompts: assess_attachment, analyze_climate]
    end

    %% Architecture Domain
    AO -->|extends| BO
    AO -->|uses| Analyzer
    AO -->|uses| DocWriter
    Analyzer -->|is-a| AD
    DocWriter -->|is-a| AD

    %% UX Domain
    UO -->|extends| BO
    UO -->|uses| UXResearcher
    UO -->|uses| IAArchitect
    UO -->|uses| UIDesigner
    UO -->|uses| ProtoDev
    UXResearcher -->|is-a| AD
    IAArchitect -->|is-a| AD
    UIDesigner -->|is-a| AD
    ProtoDev -->|is-a| AD

    %% Framework
    BO -->|delegates to| PhaseExec
    BO -->|tracks with| CostTrack
    BO -->|verifies with| OutputVerify
    BO -->|queries| AR
    BO -->|validates| MCP

    %% Agent System
    AR -->|loads| AD
    AR -->|caches in| AgentCache

    %% Tool System
    MCP -->|manages| Figma
    BO -->|configures| ToolList

    %% MCP Servers
    Lila -->|provides| Resources
    Lila -->|provides| Tools
    Lila -->|provides| Prompts
    Simple -->|provides| Resources
    Simple -->|provides| Tools
    Simple -->|provides| Prompts

    style BO fill:#e1f5ff,stroke:#01579b,stroke-width:3px
    style AO fill:#c8e6c9,stroke:#2e7d32
    style UO fill:#ffccbc,stroke:#d84315
    style Lila fill:#fff3e0,stroke:#e65100
```

### Explanation

This diagram shows the **relationships between major components**:

- **Domain Orchestrators** (ArchitectureOrchestrator, UXOrchestrator) extend BaseOrchestrator and use domain-specific agents
- **BaseOrchestrator** provides common functionality for phase execution, cost tracking, and output verification
- **AgentRegistry** manages agent discovery, loading, and caching
- **MCPRegistry** manages MCP server availability and tool integration
- **MCP Servers** expose resources (data), tools (actions), and prompts (AI templates)

Key patterns:
- **Inheritance**: Domain orchestrators extend BaseOrchestrator
- **Composition**: Orchestrators use agents, which are instances of AgentDefinition
- **Registry Pattern**: AgentRegistry and MCPRegistry centralize component discovery
- **Facade Pattern**: SimpleLilaMCPServer provides same interface as LilaMCPServer with mock data

---

## Class Hierarchies

### Orchestrator Hierarchy

```mermaid
classDiagram
    class ABC {
        <<abstract>>
    }

    class BaseOrchestrator {
        <<abstract>>
        +domain_name: str
        +output_dir: Path
        +show_tool_details: bool
        +total_cost: float
        +phase_costs: Dict
        +completed_phases: List
        +create_output_structure()
        +display_message()
        +display_phase_header()
        +track_phase_cost()
        +mark_phase_complete()
        +verify_outputs() bool
        +display_summary()
        +execute_phase()
        +create_client_options()
        +run_with_client()
        +get_agent_definitions()* Dict
        +get_allowed_tools()* List
        +run()*
    }

    class ArchitectureOrchestrator {
        +docs_dir: Path
        +diagrams_dir: Path
        +reports_dir: Path
        +phase_1_component_inventory()
        +phase_2_architecture_diagrams()
        +phase_3_data_flows()
        +phase_4_api_documentation()
        +phase_5_synthesis()
        +get_agent_definitions() Dict
        +get_allowed_tools() List
        +run()
    }

    class UXOrchestrator {
        +project_name: str
        +research_dir: Path
        +ia_dir: Path
        +design_dir: Path
        +prototypes_dir: Path
        +api_contracts_dir: Path
        +design_system_dir: Path
        +phase_1_ux_research()
        +phase_2_information_architecture()
        +phase_3_visual_design()
        +phase_4_interactive_prototyping()
        +phase_5_api_contract_design()
        +phase_6_design_system_documentation()
        +get_agent_definitions() Dict
        +get_allowed_tools() List
        +run()
    }

    class CrossOrchestratorCommunication {
        <<mixin>>
        +orchestrator_registry: Dict
        +register_orchestrator()
        +invoke_orchestrator() Any
    }

    ABC <|-- BaseOrchestrator
    BaseOrchestrator <|-- ArchitectureOrchestrator
    BaseOrchestrator <|-- UXOrchestrator
    BaseOrchestrator <.. CrossOrchestratorCommunication : can mix in

    note for BaseOrchestrator "Abstract methods enforce\nconsistent interface:\n- get_agent_definitions()\n- get_allowed_tools()\n- run()"
```

### MCP Server Hierarchy

```mermaid
classDiagram
    class LilaMCPServer {
        +app: FastMCP
        +driver: GraphDatabase
        -_setup_database()
        +close()
        -_register_resources()
        -_register_tools()
        -_register_prompts()
        -_register_health_check()
        +run_server()
    }

    class SimpleLilaMCPServer {
        +app: FastMCP
        +mock_personas: Dict
        +mock_relationships: Dict
        +mock_interactions: List
        -_setup_mock_data()
        -_register_resources()
        -_register_tools()
        -_register_prompts()
        +run_server()
    }

    class FastMCP {
        <<external>>
        +resource()
        +tool()
        +prompt()
        +custom_route()
        +run()
    }

    LilaMCPServer --> FastMCP : uses
    SimpleLilaMCPServer --> FastMCP : uses

    note for LilaMCPServer "Production server with\nNeo4j backend"
    note for SimpleLilaMCPServer "Development server with\nmock data"
```

### Registry Classes

```mermaid
classDiagram
    class AgentRegistry {
        +agents_dir: Path
        -_cache: Dict~str, AgentDefinition~
        +discover_agents(domain) Dict
        +load_agent(agent_name, domain) AgentDefinition
        +load_domain_agents(domain) Dict
    }

    class MCPRegistry {
        +available_servers: Dict
        +discover_mcp_servers() Dict
        +is_server_available(server_name) bool
        +get_server_tools(server_name) List
        +validate_tool_availability(tool_name) bool
        +get_configuration_requirements(server_name) Dict
        +get_fallback_options(tool_name) List
    }

    class FigmaIntegration {
        +access_token: str
        +mcp_available: bool
        +is_available() bool
        +get_design_context(file_id) Dict
        +export_to_code(component_id, framework) str
        +create_component(spec) str
        +get_setup_instructions() str
    }

    MCPRegistry --> FigmaIntegration : manages
```

### Data Management Classes

```mermaid
classDiagram
    class Neo4jDataImporter {
        +uri: str
        +user: str
        +password: str
        +driver: GraphDatabase
        +max_retries: int
        -_connect_with_retry()
        +close()
        +clear_database()
        +load_schema(schema_path)
        -_load_family_graph_data(schema, session)
        -_map_behavioral_to_bigfive(behavioral_style) Dict
        +import_seed_data(seed_data_path)
        +create_default_personas()
        +verify_import()
    }

    class Neo4jDataExporter {
        +uri: str
        +user: str
        +password: str
        +driver: GraphDatabase
        +close()
        +export_personas() List
        +export_relationships() List
        +export_memories() List
        +export_goals() List
        +generate_cypher_script(...) str
    }

    class GraphDatabase {
        <<external>>
    }

    Neo4jDataImporter --> GraphDatabase : uses
    Neo4jDataExporter --> GraphDatabase : uses
```

### Explanation

The class hierarchies demonstrate several design patterns:

1. **Template Method Pattern**: BaseOrchestrator defines workflow template, subclasses implement specific steps
2. **Abstract Base Class Pattern**: BaseOrchestrator enforces interface through abstract methods
3. **Mixin Pattern**: CrossOrchestratorCommunication adds functionality without inheritance complexity
4. **Facade Pattern**: LilaMCPServer and SimpleLilaMCPServer provide unified interface to FastMCP
5. **Registry Pattern**: AgentRegistry and MCPRegistry centralize component management
6. **Strategy Pattern**: Different orchestrators implement different domain strategies

Key inheritance relationships:
- `BaseOrchestrator` extends `ABC` (Abstract Base Class)
- `ArchitectureOrchestrator` extends `BaseOrchestrator`
- `UXOrchestrator` extends `BaseOrchestrator`

Key composition relationships:
- Orchestrators use `AgentRegistry` to load agents
- Orchestrators use `MCPRegistry` to validate tools
- MCPRegistry manages `FigmaIntegration`

---

## Module Dependencies

```mermaid
graph TD
    subgraph "External Dependencies"
        ClaudeSDK[claude_agent_sdk<br/>AgentDefinition, ClaudeSDKClient]
        FastMCP[fastmcp<br/>FastMCP, Resource, Tool, Prompt]
        Neo4jDriver[neo4j<br/>GraphDatabase]
        StdLib[Standard Library<br/>asyncio, pathlib, typing, abc]
    end

    subgraph "orchestrators/"
        BaseOrch[base_orchestrator.py]
        ArchOrch[architecture_orchestrator.py]
        UXOrch[ux_orchestrator.py]
        OrchInit[__init__.py]
    end

    subgraph "agents/"
        AgentReg[registry.py]
        AgentInit[__init__.py]
        AgentJSON[JSON Files<br/>ux/, architecture/]
    end

    subgraph "tools/"
        MCPReg[mcp_registry.py]
        FigmaInt[figma_integration.py]
        ToolsInit[__init__.py]
    end

    subgraph "MCP Servers"
        LilaMCP[lila_mcp_server.py]
        SimpleMCP[simple_lila_mcp_server.py]
    end

    subgraph "Data Management"
        ImportData[import_data.py]
        ExportData[export_data.py]
    end

    subgraph "Tests"
        TestOrch[test_orchestrators.py]
        TestMCP[test_mcp_validation.py]
    end

    %% orchestrators/ dependencies
    BaseOrch --> ClaudeSDK
    BaseOrch --> StdLib
    ArchOrch --> BaseOrch
    ArchOrch --> ClaudeSDK
    UXOrch --> BaseOrch
    UXOrch --> ClaudeSDK
    OrchInit --> BaseOrch

    %% agents/ dependencies
    AgentReg --> ClaudeSDK
    AgentReg --> StdLib
    AgentReg --> AgentJSON
    AgentInit --> AgentReg

    %% tools/ dependencies
    MCPReg --> StdLib
    FigmaInt --> StdLib
    ToolsInit --> MCPReg
    ToolsInit --> FigmaInt

    %% MCP Servers dependencies
    LilaMCP --> FastMCP
    LilaMCP --> Neo4jDriver
    LilaMCP --> StdLib
    SimpleMCP --> FastMCP
    SimpleMCP --> StdLib

    %% Data Management dependencies
    ImportData --> Neo4jDriver
    ImportData --> StdLib
    ExportData --> Neo4jDriver
    ExportData --> StdLib

    %% Tests dependencies
    TestOrch --> ArchOrch
    TestOrch --> UXOrch
    TestOrch --> AgentReg
    TestOrch --> MCPReg
    TestOrch --> FigmaInt
    TestMCP --> SimpleMCP
    TestMCP --> FastMCP

    style ClaudeSDK fill:#e1f5ff,stroke:#01579b
    style FastMCP fill:#fff3e0,stroke:#e65100
    style Neo4jDriver fill:#f3e5f5,stroke:#4a148c
    style BaseOrch fill:#e8f5e9,stroke:#1b5e20
```

### Module Import Matrix

| Module | Imports From | Exports To |
|--------|-------------|------------|
| **orchestrators/base_orchestrator.py** | claude_agent_sdk, abc, asyncio, pathlib, typing | architecture_orchestrator, ux_orchestrator, orchestrators/__init__ |
| **orchestrators/architecture_orchestrator.py** | base_orchestrator, claude_agent_sdk, pathlib, typing | test_orchestrators |
| **orchestrators/ux_orchestrator.py** | base_orchestrator, claude_agent_sdk, pathlib, typing | test_orchestrators, test_orchestrator_run |
| **agents/registry.py** | claude_agent_sdk, json, pathlib, typing | agents/__init__, test_orchestrators |
| **tools/mcp_registry.py** | typing, subprocess, json | tools/__init__, test_orchestrators |
| **tools/figma_integration.py** | typing, os | tools/__init__, test_orchestrators |
| **lila_mcp_server.py** | fastmcp, neo4j, asyncio, logging, os, datetime, typing, pathlib, dotenv | test_mcp_validation |
| **simple_lila_mcp_server.py** | fastmcp, neo4j, asyncio, logging, os, datetime, typing, pathlib, dotenv | test_mcp_validation |
| **import_data.py** | neo4j, os, sys, argparse, json, time, pathlib, typing | (standalone script) |
| **export_data.py** | neo4j, os, sys, argparse, pathlib, typing, json | (standalone script) |

### Dependency Flow

```mermaid
graph LR
    Entry[Entry Points] --> Orchestrators
    Entry --> MCPServers[MCP Servers]
    Entry --> DataMgmt[Data Management]

    Orchestrators --> BaseFramework[Base Framework]
    Orchestrators --> AgentSystem[Agent System]
    Orchestrators --> ToolSystem[Tool System]

    BaseFramework --> ClaudeSDK[Claude Agent SDK]
    AgentSystem --> ClaudeSDK
    MCPServers --> FastMCPLib[FastMCP Library]
    MCPServers --> Neo4j[Neo4j Driver]
    DataMgmt --> Neo4j

    ClaudeSDK --> StdLib[Python Standard Library]
    FastMCPLib --> StdLib
    Neo4j --> StdLib

    style ClaudeSDK fill:#e1f5ff,stroke:#01579b
    style FastMCPLib fill:#fff3e0,stroke:#e65100
    style Neo4j fill:#f3e5f5,stroke:#4a148c
```

### Explanation

The module dependencies show:

1. **Clean Separation**: Each package (orchestrators, agents, tools) has clear boundaries
2. **External Dependencies**:
   - `claude_agent_sdk`: Agent orchestration and SDK tools
   - `fastmcp`: FastMCP server framework
   - `neo4j`: Neo4j Python driver
   - Standard library: asyncio, pathlib, typing, abc, etc.

3. **Internal Dependencies**:
   - Domain orchestrators depend on BaseOrchestrator
   - All orchestrators depend on AgentRegistry and MCPRegistry
   - Tests depend on all components for validation

4. **No Circular Dependencies**: Dependency graph is acyclic (DAG)

5. **Package Initialization Files**:
   - `orchestrators/__init__.py`: Exports BaseOrchestrator
   - `agents/__init__.py`: Exports AgentRegistry
   - `tools/__init__.py`: Exports MCPRegistry, FigmaIntegration

This dependency structure enables:
- **Modularity**: Components can be developed independently
- **Testability**: Mock implementations can replace external dependencies
- **Maintainability**: Clear dependency boundaries reduce coupling
- **Extensibility**: New components can be added without modifying existing code

---

## Data Flow Architecture

### Orchestrator Workflow

```mermaid
sequenceDiagram
    participant CLI
    participant Orchestrator
    participant BaseOrchestrator
    participant ClaudeSDKClient
    participant Agent
    participant Tools
    participant OutputFiles

    CLI->>Orchestrator: run_with_client()
    Orchestrator->>BaseOrchestrator: create_client_options()
    BaseOrchestrator->>AgentRegistry: load agents
    BaseOrchestrator->>MCPRegistry: validate tools
    BaseOrchestrator-->>Orchestrator: ClaudeAgentOptions

    Orchestrator->>ClaudeSDKClient: create client context

    loop For each phase
        Orchestrator->>BaseOrchestrator: execute_phase(prompt)
        BaseOrchestrator->>ClaudeSDKClient: query(prompt)
        ClaudeSDKClient->>Agent: process with tools

        loop Agent execution
            Agent->>Tools: Read/Write/Grep/Glob/Bash
            Tools-->>Agent: results
            Agent->>OutputFiles: Write documentation
        end

        Agent-->>ClaudeSDKClient: completion
        ClaudeSDKClient-->>BaseOrchestrator: ResultMessage (cost)
        BaseOrchestrator->>BaseOrchestrator: track_phase_cost()
        BaseOrchestrator->>BaseOrchestrator: mark_phase_complete()
    end

    Orchestrator->>BaseOrchestrator: verify_outputs()
    BaseOrchestrator->>BaseOrchestrator: display_summary()
    BaseOrchestrator-->>CLI: completion status
```

### MCP Server Request Flow

```mermaid
sequenceDiagram
    participant Client
    participant LilaMCPServer
    participant FastMCP
    participant Neo4j
    participant ResourceHandler
    participant ToolHandler

    Client->>LilaMCPServer: HTTP/SSE request
    LilaMCPServer->>FastMCP: route request

    alt Resource Request
        FastMCP->>ResourceHandler: handle resource
        ResourceHandler->>Neo4j: query data
        Neo4j-->>ResourceHandler: results
        ResourceHandler-->>FastMCP: formatted response
    else Tool Request
        FastMCP->>ToolHandler: execute tool
        ToolHandler->>Neo4j: update data
        Neo4j-->>ToolHandler: confirmation
        ToolHandler-->>FastMCP: execution result
    else Prompt Request
        FastMCP->>Client: prompt template
    end

    FastMCP-->>LilaMCPServer: response
    LilaMCPServer-->>Client: JSON response
```

### Explanation

These data flow diagrams illustrate:

1. **Orchestrator Workflow**: Shows how orchestrators coordinate multi-phase workflows through the Claude SDK
2. **MCP Server Flow**: Shows how MCP servers handle resources (data queries), tools (actions), and prompts (templates)

Key flow characteristics:
- **Asynchronous**: All flows use async/await patterns
- **Phase-based**: Orchestrators execute workflows in sequential phases
- **Cost-tracked**: Each phase tracks API usage costs
- **Output-verified**: Results are validated after completion
- **Error-handled**: Exceptions are caught and reported at each layer

---

## Summary

This architecture demonstrates several key principles:

1. **Layered Architecture**: Clear separation between presentation, orchestration, services, and data layers
2. **Abstract Base Classes**: BaseOrchestrator provides template for domain-specific implementations
3. **Registry Pattern**: Centralized discovery and management of agents and MCP servers
4. **Composition over Inheritance**: Agents are composed into orchestrators rather than inherited
5. **Dependency Injection**: Clients and options are injected rather than constructed internally
6. **Single Responsibility**: Each class has a focused, well-defined purpose
7. **Open/Closed Principle**: System is open for extension (new orchestrators) but closed for modification

The architecture supports:
- **Multiple domains**: Architecture analysis, UX design, and future domains
- **Flexible workflows**: Each orchestrator defines its own phases
- **Tool integration**: MCP servers can be added without modifying core code
- **Testing**: Mock implementations support testing without external dependencies
- **Scalability**: Stateless orchestrators can run in parallel

### Key Files Reference

- **Base Framework**: `/home/donbr/lila-graph/lila-mcp/orchestrators/base_orchestrator.py` (lines 26-344)
- **Architecture Orchestrator**: `/home/donbr/lila-graph/lila-mcp/orchestrators/architecture_orchestrator.py` (lines 20-313)
- **UX Orchestrator**: `/home/donbr/lila-graph/lila-mcp/orchestrators/ux_orchestrator.py` (lines 21-619)
- **Agent Registry**: `/home/donbr/lila-graph/lila-mcp/agents/registry.py` (lines 10-100)
- **MCP Registry**: `/home/donbr/lila-graph/lila-mcp/tools/mcp_registry.py` (lines 8-153)
- **Lila MCP Server**: `/home/donbr/lila-graph/lila-mcp/lila_mcp_server.py` (lines 29-779)
- **Data Importer**: `/home/donbr/lila-graph/lila-mcp/import_data.py` (lines 22-465)
- **Data Exporter**: `/home/donbr/lila-graph/lila-mcp/export_data.py` (lines 21-294)
