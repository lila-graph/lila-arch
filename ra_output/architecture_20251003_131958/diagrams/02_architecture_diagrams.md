# Architecture Diagrams

This document provides comprehensive architectural visualizations of the Lila MCP (Model Context Protocol) system, which consists of two main subsystems: a psychological relationship modeling MCP server and a repository analyzer framework for multi-domain code analysis.

## System Architecture

The system follows a layered architecture pattern with clear separation of concerns across presentation, application, domain, and data layers.

```mermaid
graph TD
    subgraph "Presentation Layer"
        MCP1[LilaMCPServer<br/>FastMCP Interface]
        MCP2[SimpleLilaMCPServer<br/>FastMCP Interface]
        CLI[CLI Tools<br/>import_data.py<br/>export_data.py]
    end

    subgraph "Application Layer"
        ARCH[ArchitectureOrchestrator<br/>5-phase analysis]
        UX[UXOrchestrator<br/>6-phase design workflow]
        BASE[BaseOrchestrator<br/>Framework core]
    end

    subgraph "Domain Layer"
        REG[AgentRegistry<br/>Agent discovery & loading]
        AGENTS[Agent Definitions<br/>JSON-based configs]
        TOOLS[Tool Integrations<br/>MCPRegistry<br/>FigmaIntegration]
    end

    subgraph "Data Layer"
        NEO4J[(Neo4j Database<br/>Psychological data)]
        FS[File System<br/>Analysis outputs<br/>ra_output/]
        AGENT_JSON[Agent JSON Files<br/>ra_agents/]
    end

    %% Presentation -> Application
    MCP1 --> NEO4J
    MCP2 --> NEO4J
    CLI --> NEO4J
    ARCH --> BASE
    UX --> BASE

    %% Application -> Domain
    BASE --> REG
    BASE --> AGENTS
    ARCH --> AGENTS
    UX --> AGENTS
    UX --> TOOLS

    %% Domain -> Data
    REG --> AGENT_JSON
    AGENTS --> FS
    TOOLS --> FS

    %% SDK Integration
    SDK[Claude Agent SDK]
    BASE -.-> SDK

    %% External Dependencies
    FASTMCP[FastMCP Framework]
    MCP1 -.-> FASTMCP
    MCP2 -.-> FASTMCP

    style MCP1 fill:#e1f5ff
    style MCP2 fill:#e1f5ff
    style ARCH fill:#fff4e1
    style UX fill:#fff4e1
    style BASE fill:#ffe1e1
    style NEO4J fill:#e8f5e9
    style FS fill:#e8f5e9
```

**Explanation**: The system architecture demonstrates a clean separation of concerns across four distinct layers:

- **Presentation Layer**: Exposes MCP servers (`LilaMCPServer` and `SimpleLilaMCPServer`) that provide resources, tools, and prompts for psychological relationship modeling. CLI tools handle data import/export operations with Neo4j.

- **Application Layer**: Houses the orchestrator framework with `BaseOrchestrator` providing core functionality and specialized orchestrators (`ArchitectureOrchestrator`, `UXOrchestrator`) implementing domain-specific workflows. Each orchestrator manages multi-phase analysis pipelines.

- **Domain Layer**: Contains reusable agent definitions (loaded from JSON), the agent registry for discovery and instantiation, and tool integrations for MCP servers and external services like Figma.

- **Data Layer**: Persists data to Neo4j (for psychological relationship modeling) and the file system (for analysis outputs in timestamped directories under `ra_output/`). Agent configurations are stored as JSON files in `ra_agents/`.

This architecture supports modularity, reusability, and clear dependency flow from top to bottom, while leveraging external frameworks (FastMCP, Claude Agent SDK) for MCP protocol implementation and agent orchestration.

## Component Relationships

The component relationship diagram illustrates how the main subsystems interact and depend on each other.

```mermaid
graph TB
    subgraph "MCP Server Subsystem"
        LILA[LilaMCPServer]
        SIMPLE[SimpleLilaMCPServer]

        LILA -->|exposes| RES1[Resources<br/>personas, relationships,<br/>interactions, climate]
        LILA -->|provides| TOOL1[Tools<br/>update_metrics,<br/>record_interaction,<br/>analyze_compatibility]
        LILA -->|offers| PROMPT1[Prompts<br/>assess_attachment,<br/>analyze_climate,<br/>generate_response]

        SIMPLE -->|exposes| RES2[Resources<br/>Mock data variants]
        SIMPLE -->|provides| TOOL2[Tools<br/>Same interface]
        SIMPLE -->|offers| PROMPT2[Prompts<br/>Same templates]
    end

    subgraph "Orchestrator Framework Subsystem"
        BASE_ORCH[BaseOrchestrator<br/>ABC]
        ARCH_ORCH[ArchitectureOrchestrator]
        UX_ORCH[UXOrchestrator]

        ARCH_ORCH -->|inherits| BASE_ORCH
        UX_ORCH -->|inherits| BASE_ORCH

        ARCH_ORCH -->|uses| ARCH_AGENTS[Architecture Agents<br/>analyzer, doc-writer]
        UX_ORCH -->|uses| UX_AGENTS[UX Agents<br/>ux-researcher, ia-architect,<br/>ui-designer, prototype-developer]

        BASE_ORCH -->|manages| PHASES[Phase Execution<br/>Progress tracking<br/>Cost tracking]
        BASE_ORCH -->|uses| SDK[ClaudeSDKClient]
    end

    subgraph "Agent & Tool Registry"
        AGENT_REG[AgentRegistry]
        MCP_REG[MCPRegistry]
        FIGMA[FigmaIntegration]

        AGENT_REG -->|loads| AGENT_JSON[Agent JSON Definitions<br/>ra_agents/*/]
        MCP_REG -->|discovers| MCP_SERVERS[MCP Servers<br/>figma, v0, playwright]
        FIGMA -->|integrates| FIGMA_API[Figma API & MCP]
    end

    subgraph "Data Access Layer"
        NEO4J_IMPORT[Neo4jDataImporter]
        NEO4J_EXPORT[Neo4jDataExporter]

        NEO4J_IMPORT -->|writes to| DB[(Neo4j Database)]
        NEO4J_EXPORT -->|reads from| DB
    end

    %% Cross-subsystem relationships
    LILA -->|connects to| DB
    SIMPLE -->|mock fallback for| DB

    ARCH_ORCH -->|discovers agents via| AGENT_REG
    UX_ORCH -->|discovers agents via| AGENT_REG
    UX_ORCH -->|integrates with| MCP_REG
    UX_ORCH -->|may use| FIGMA

    %% Future extension
    CROSS[CrossOrchestratorCommunication<br/>Mixin - Future]
    BASE_ORCH -.->|future extension| CROSS

    style BASE_ORCH fill:#ffe1e1
    style ARCH_ORCH fill:#fff4e1
    style UX_ORCH fill:#fff4e1
    style LILA fill:#e1f5ff
    style SIMPLE fill:#e1f5ff
    style DB fill:#e8f5e9
    style CROSS fill:#f0f0f0,stroke-dasharray: 5 5
```

**Explanation**: This diagram reveals the dual-nature architecture of the system:

**MCP Server Subsystem** (left):
- `LilaMCPServer` provides full Neo4j integration for psychological relationship modeling
- `SimpleLilaMCPServer` offers a mock-data fallback for testing and development
- Both expose the same MCP interface (resources, tools, prompts) ensuring compatibility
- Resources provide read access to psychological data (personas, relationships, interactions)
- Tools enable mutations (updating metrics, recording interactions, analyzing compatibility)
- Prompts guide AI-assisted psychological assessment and response generation

**Orchestrator Framework Subsystem** (center):
- `BaseOrchestrator` defines the abstract framework for multi-phase analysis workflows
- Concrete orchestrators (`ArchitectureOrchestrator`, `UXOrchestrator`) implement domain-specific phases
- Each orchestrator uses specialized agents loaded dynamically from JSON definitions
- The framework manages phase execution, progress tracking, cost accounting, and output verification
- Integration with Claude Agent SDK enables AI-powered analysis and documentation generation

**Agent & Tool Registry** (right):
- `AgentRegistry` provides dynamic discovery and loading of agent definitions from JSON files
- `MCPRegistry` discovers available MCP servers (Figma, v0, Playwright) for tool integration
- `FigmaIntegration` bridges Figma MCP server and REST API for design workflow support

**Data Access Layer** (bottom):
- Import/export utilities provide bulk data operations with Neo4j
- Orchestrators write analysis outputs to timestamped directories in the file system

**Cross-cutting Concerns**:
- `CrossOrchestratorCommunication` mixin (future) will enable orchestrators to invoke each other for validation and integration (e.g., UX orchestrator validating API contracts with Architecture orchestrator)

## Class Hierarchies

The class hierarchy diagram shows inheritance relationships and key composition patterns.

```mermaid
classDiagram
    %% Abstract Base Classes
    class ABC {
        <<abstract>>
    }

    %% BaseOrchestrator hierarchy
    class BaseOrchestrator {
        <<abstract>>
        +domain_name: str
        +output_dir: Path
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
        +get_agent_definitions()* Dict
        +get_allowed_tools()* List
        +run()* async
        +execute_phase() async
        +create_client_options()
        +run_with_client() async
    }

    class ArchitectureOrchestrator {
        +docs_dir: Path
        +diagrams_dir: Path
        +reports_dir: Path
        +phase_1_component_inventory() async
        +phase_2_architecture_diagrams() async
        +phase_3_data_flows() async
        +phase_4_api_documentation() async
        +phase_5_synthesis() async
        +get_agent_definitions() Dict
        +get_allowed_tools() List
        +run() async
    }

    class UXOrchestrator {
        +project_name: str
        +research_dir: Path
        +ia_dir: Path
        +design_dir: Path
        +prototypes_dir: Path
        +api_contracts_dir: Path
        +design_system_dir: Path
        +phase_1_ux_research() async
        +phase_2_information_architecture() async
        +phase_3_visual_design() async
        +phase_4_interactive_prototyping() async
        +phase_5_api_contract_design() async
        +phase_6_design_system_documentation() async
        +get_agent_definitions() Dict
        +get_allowed_tools() List
        +run() async
    }

    %% MCP Server classes
    class LilaMCPServer {
        +app: FastMCP
        +driver: GraphDatabase
        +_setup_database()
        +_register_resources()
        +_register_tools()
        +_register_prompts()
        +_register_health_check()
        +run_server() async
        +close()
    }

    class SimpleLilaMCPServer {
        +app: FastMCP
        +driver: GraphDatabase
        +mock_personas: Dict
        +mock_relationships: Dict
        +mock_interactions: List
        +_setup_database()
        +_register_resources()
        +_register_tools()
        +_register_prompts()
        +run_server() async
        +close()
    }

    %% Registry classes
    class AgentRegistry {
        +agents_dir: Path
        -_cache: Dict
        +discover_agents() Dict
        +load_agent() AgentDefinition
        +load_domain_agents() Dict
    }

    class MCPRegistry {
        +available_servers: Dict
        +discover_mcp_servers() Dict
        +is_server_available() bool
        +get_server_tools() List
        +validate_tool_availability() bool
        +get_configuration_requirements() Dict
        +get_fallback_options() List
    }

    %% Integration classes
    class FigmaIntegration {
        +access_token: str
        +mcp_available: bool
        +is_available() bool
        +get_file() Dict
        +get_components() List
        +get_fallback_instructions() str
    }

    class CrossOrchestratorCommunication {
        <<mixin>>
        +orchestrator_registry: Dict
        +register_orchestrator()
        +invoke_orchestrator() async
    }

    %% Data access classes
    class Neo4jDataImporter {
        +driver: GraphDatabase
        +import_personas()
        +import_relationships()
        +import_all()
        +close()
    }

    class Neo4jDataExporter {
        +driver: GraphDatabase
        +export_personas()
        +export_relationships()
        +export_all()
        +close()
    }

    %% Inheritance relationships
    ABC <|-- BaseOrchestrator
    BaseOrchestrator <|-- ArchitectureOrchestrator
    BaseOrchestrator <|-- UXOrchestrator

    %% Composition relationships
    ArchitectureOrchestrator ..> AgentRegistry : uses
    UXOrchestrator ..> AgentRegistry : uses
    UXOrchestrator ..> MCPRegistry : uses
    UXOrchestrator ..> FigmaIntegration : may use

    LilaMCPServer ..> GraphDatabase : owns
    SimpleLilaMCPServer ..> GraphDatabase : owns
    Neo4jDataImporter ..> GraphDatabase : owns
    Neo4jDataExporter ..> GraphDatabase : owns

    %% Future mixin
    BaseOrchestrator ..|> CrossOrchestratorCommunication : future mixin

    %% External dependencies
    class FastMCP {
        <<external>>
    }
    class GraphDatabase {
        <<external>>
    }
    class AgentDefinition {
        <<external>>
    }

    LilaMCPServer ..> FastMCP : uses
    SimpleLilaMCPServer ..> FastMCP : uses
    AgentRegistry ..> AgentDefinition : creates
```

**Explanation**: The class hierarchy reveals several key architectural patterns:

**Abstract Template Pattern** (`BaseOrchestrator`):
- Inherits from Python's `ABC` to enforce implementation of abstract methods
- Provides template methods (`run_with_client()`) that orchestrate the workflow
- Forces subclasses to implement `get_agent_definitions()`, `get_allowed_tools()`, and `run()`
- Encapsulates common functionality: phase execution, progress tracking, cost accounting, output verification
- Enables polymorphic orchestrator usage and extensibility for new domains

**Concrete Orchestrator Implementations**:
- `ArchitectureOrchestrator`: Implements 5-phase repository analysis workflow (inventory → diagrams → data flows → API docs → synthesis)
- `UXOrchestrator`: Implements 6-phase UX design workflow (research → IA → design → prototyping → API contracts → design system)
- Both orchestrators define domain-specific directory structures and phase methods
- Phase methods are private (by convention) and called from the public `run()` method

**MCP Server Variants**:
- `LilaMCPServer` and `SimpleLilaMCPServer` are parallel implementations (no shared base class)
- Both follow the same pattern: `_setup_database()`, `_register_resources/tools/prompts()`, `run_server()`
- `SimpleLilaMCPServer` includes mock data fallback for offline/testing scenarios
- Both own a Neo4j `GraphDatabase` driver for data persistence

**Registry Pattern**:
- `AgentRegistry`: Discovers and caches agent definitions from JSON files, supports domain-scoped queries
- `MCPRegistry`: Auto-discovers available MCP servers and validates tool availability, provides fallback strategies

**Strategy Pattern** (via composition):
- Orchestrators compose with registries to dynamically load agents and tools
- `FigmaIntegration` provides pluggable Figma API access with MCP fallback
- `CrossOrchestratorCommunication` mixin (future) will enable cross-domain collaboration

**Data Access Layer**:
- `Neo4jDataImporter` and `Neo4jDataExporter` provide bulk operations on Neo4j
- Both own a `GraphDatabase` driver with lifecycle management (close method)

**External Dependencies**:
- `FastMCP`: MCP protocol implementation framework
- `GraphDatabase`: Neo4j Python driver
- `AgentDefinition`: Claude Agent SDK agent configuration
- `ClaudeSDKClient`: AI agent execution runtime (used in BaseOrchestrator)

## Module Dependencies

The module dependency graph shows how Python modules import and depend on each other.

```mermaid
graph LR
    subgraph "Orchestrators Package (ra_orchestrators/)"
        BASE[base_orchestrator.py]
        ARCH[architecture_orchestrator.py]
        UX[ux_orchestrator.py]
        ARCH_OLD[architecture.py<br/>legacy]
        ORCH_INIT[__init__.py]
    end

    subgraph "Agents Package (ra_agents/)"
        AGENT_REG[registry.py]
        AGENT_INIT[__init__.py]
        ARCH_JSON[architecture/*.json]
        UX_JSON[ux/*.json]
    end

    subgraph "Tools Package (ra_tools/)"
        MCP_REG[mcp_registry.py]
        FIGMA[figma_integration.py]
        TOOLS_INIT[__init__.py]
    end

    subgraph "MCP Servers (root level)"
        LILA[lila_mcp_server.py]
        SIMPLE[simple_lila_mcp_server.py]
    end

    subgraph "Data Utilities (root level)"
        IMPORT[import_data.py]
        EXPORT[export_data.py]
    end

    subgraph "Testing (root level)"
        TEST_MCP[test_mcp_validation.py]
        TEST_ORCH[test_orchestrators.py]
    end

    subgraph "External Dependencies"
        SDK[claude_agent_sdk<br/>AgentDefinition<br/>ClaudeSDKClient<br/>Message types]
        FASTMCP[fastmcp<br/>FastMCP<br/>Resource<br/>Tool<br/>Prompt]
        NEO4J[neo4j<br/>GraphDatabase]
        STDLIB[Python stdlib<br/>asyncio, pathlib,<br/>typing, json, os]
    end

    %% Orchestrator internal dependencies
    ARCH --> BASE
    UX --> BASE
    ORCH_INIT --> BASE

    %% Orchestrator external dependencies
    BASE --> SDK
    ARCH --> SDK
    UX --> SDK
    BASE --> STDLIB

    %% Orchestrator to Agent dependencies
    ARCH --> AGENT_REG
    UX --> AGENT_REG
    AGENT_REG --> STDLIB
    AGENT_REG --> SDK
    AGENT_INIT --> AGENT_REG

    %% Orchestrator to Tools dependencies
    UX --> MCP_REG
    UX --> FIGMA
    TOOLS_INIT --> MCP_REG
    TOOLS_INIT --> FIGMA
    MCP_REG --> STDLIB
    FIGMA --> STDLIB

    %% Agent JSON dependencies
    AGENT_REG -.-> ARCH_JSON
    AGENT_REG -.-> UX_JSON

    %% MCP Server dependencies
    LILA --> FASTMCP
    LILA --> NEO4J
    LILA --> STDLIB
    SIMPLE --> FASTMCP
    SIMPLE --> NEO4J
    SIMPLE --> STDLIB

    %% Data utility dependencies
    IMPORT --> NEO4J
    IMPORT --> STDLIB
    EXPORT --> NEO4J
    EXPORT --> STDLIB

    %% Test dependencies
    TEST_MCP --> SIMPLE
    TEST_MCP --> FASTMCP
    TEST_MCP --> STDLIB
    TEST_ORCH --> BASE
    TEST_ORCH --> ARCH
    TEST_ORCH --> UX
    TEST_ORCH --> STDLIB

    style BASE fill:#ffe1e1
    style ARCH fill:#fff4e1
    style UX fill:#fff4e1
    style SDK fill:#e8f5e9
    style FASTMCP fill:#e8f5e9
    style NEO4J fill:#e8f5e9
    style STDLIB fill:#f0f0f0
```

**Explanation**: The module dependency graph illustrates the layered and modular nature of the codebase:

**Package Organization**:
- **ra_orchestrators/**: Contains the orchestrator framework with clear internal dependencies (`ArchitectureOrchestrator` → `BaseOrchestrator`)
- **ra_agents/**: Houses agent registry and JSON agent definitions organized by domain (architecture/, ux/)
- **ra_tools/**: Provides tool integration utilities (MCP registry, Figma integration)
- **Root level**: MCP servers, data utilities, and tests

**Dependency Layers** (bottom-up):

1. **External Dependencies** (foundation):
   - `claude_agent_sdk`: AI agent orchestration framework (AgentDefinition, ClaudeSDKClient, message types)
   - `fastmcp`: MCP protocol implementation (FastMCP, Resource, Tool, Prompt decorators)
   - `neo4j`: Graph database driver for psychological data persistence
   - Python stdlib: Standard library modules (asyncio, pathlib, typing, json, os, datetime)

2. **Core Utilities** (middle layer):
   - `AgentRegistry`: Loads agent definitions from JSON files, no dependencies on other ra_* modules
   - `MCPRegistry`: Discovers MCP servers, standalone utility
   - `FigmaIntegration`: Figma API integration, standalone utility
   - Data utilities (`import_data.py`, `export_data.py`): Direct Neo4j access, no orchestrator dependencies

3. **Orchestrator Framework** (top layer):
   - `BaseOrchestrator`: Depends only on Claude Agent SDK and stdlib
   - `ArchitectureOrchestrator`: Depends on `BaseOrchestrator`, `AgentRegistry`, and SDK
   - `UXOrchestrator`: Depends on `BaseOrchestrator`, `AgentRegistry`, `MCPRegistry`, `FigmaIntegration`, and SDK
   - `architecture.py` (legacy): Standalone implementation before refactoring to base framework

4. **MCP Servers** (parallel to orchestrators):
   - `LilaMCPServer` and `SimpleLilaMCPServer`: Depend on FastMCP and Neo4j, independent of orchestrator framework
   - Enable the "Lila as MCP server" use case separate from "Lila analysis framework" use case

**Key Dependency Patterns**:

- **No circular dependencies**: Clean acyclic graph structure
- **Minimal coupling**: Packages depend on interfaces (SDK types) rather than implementations
- **Plugin architecture**: Orchestrators discover agents and tools via registries (dependency inversion)
- **Parallel subsystems**: MCP servers and orchestrator framework are independent, can be used separately
- **Test isolation**: Tests depend on specific modules under test, not the entire system

**Import Conventions**:
- Relative imports within packages (e.g., `from .base_orchestrator import BaseOrchestrator`)
- Absolute imports for external dependencies (e.g., `from claude_agent_sdk import AgentDefinition`)
- Package `__init__.py` files expose key classes for easier imports

**Data Flow**:
- JSON agent definitions → `AgentRegistry` → Orchestrators → `ClaudeSDKClient` → Analysis outputs
- Neo4j data → MCP servers → MCP protocol → Claude Code
- Repository files → Orchestrators (via Read/Grep/Glob tools) → Analysis outputs (via Write tool)

This modular architecture supports:
- **Independent development**: MCP servers, orchestrators, and utilities can evolve separately
- **Reusability**: Agent registry and tool integrations are shared across orchestrators
- **Testability**: Each module can be tested in isolation with minimal mocking
- **Portability**: The ra_* framework can be dropped into any repository without conflicts
- **Extensibility**: New orchestrators can be added by implementing the `BaseOrchestrator` interface
