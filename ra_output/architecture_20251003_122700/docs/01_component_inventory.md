# Component Inventory

## Overview

This document provides a comprehensive inventory of all Python modules, classes, and functions in the Lila MCP codebase. Components are categorized by their visibility (public API vs internal implementation) and role in the system.

**Project**: Lila MCP - Multi-domain agent orchestration system with psychological relationship modeling
**Last Updated**: 2025-10-03

---

## Public API

### MCP Servers

#### 1. LilaMCPServer
**File**: `lila_mcp_server.py:29`

**Purpose**: FastMCP server exposing Lila's psychological relationship data and tools with Neo4j integration.

**Key Methods**:
- `__init__(self)` - Initialize MCP server with Neo4j connection (`lila_mcp_server.py:32`)
- `_setup_database(self)` - Initialize Neo4j database connection (`lila_mcp_server.py:46`)
- `close(self)` - Close Neo4j connection (`lila_mcp_server.py:64`)
- `run_server(self, host: str, port: int)` - Run the MCP server (`lila_mcp_server.py:756`)

**Resources** (Neo4j-backed):
- `neo4j://personas/all` - Retrieve all personas with psychological profiles (`lila_mcp_server.py:72`)
- `neo4j://personas/{persona_id}` - Retrieve specific persona by ID (`lila_mcp_server.py:116`)
- `neo4j://relationships/all` - Retrieve all relationship data (`lila_mcp_server.py:163`)
- `neo4j://relationships/{persona1_id}/{persona2_id}` - Get specific relationship metrics (`lila_mcp_server.py:208`)
- `neo4j://interactions/recent/{count}` - Get recent interactions (`lila_mcp_server.py:255`)

**Tools** (Async):
- `update_relationship_metrics()` - Update trust/intimacy/strength between personas (`lila_mcp_server.py:301`)
- `record_interaction()` - Record interaction with psychological analysis (`lila_mcp_server.py:363`)
- `analyze_persona_compatibility()` - Assess relationship potential based on attachment styles (`lila_mcp_server.py:402`)
- `autonomous_strategy_selection()` - AI-driven strategy selection based on attachment theory (`lila_mcp_server.py:464`)
- `assess_goal_progress()` - Assess progress toward relationship goals (`lila_mcp_server.py:516`)
- `generate_contextual_response()` - Generate psychologically authentic responses (`lila_mcp_server.py:555`)

**Prompts**:
- `assess_attachment_style()` - Determine attachment style from behavioral observations (`lila_mcp_server.py:614`)
- `analyze_emotional_climate()` - Evaluate conversation emotional dynamics (`lila_mcp_server.py:646`)
- `generate_secure_response()` - Create attachment-security-building responses (`lila_mcp_server.py:697`)

**Entry Point**: `main()` at `lila_mcp_server.py:768` (async)

---

#### 2. SimpleLilaMCPServer
**File**: `simple_lila_mcp_server.py:33`

**Purpose**: Simplified MCP server with mock data for testing and development. Same interface as LilaMCPServer but uses in-memory mock data instead of Neo4j.

**Key Differences**:
- Uses `self.mock_personas`, `self.mock_relationships`, `self.mock_interactions` (`simple_lila_mcp_server.py:42-92`)
- Mock data includes Lila and Don personas with predefined psychological profiles
- Same resource/tool/prompt interface as full server
- Additional resources: `neo4j://emotional_climate/current`, `neo4j://attachment_styles/analysis`, `neo4j://goals/active`, `neo4j://psychological_insights/trends` (`simple_lila_mcp_server.py:270-358`)
- Additional tools: `commit_relationship_state()`, `finalize_demo_session()` (`simple_lila_mcp_server.py:610-631`)

**Module Export**: `mcp = SimpleLilaMCPServer().app` for FastMCP CLI discovery (`simple_lila_mcp_server.py:825`)

---

### Orchestrator Framework

#### 3. BaseOrchestrator (Abstract Base Class)
**File**: `orchestrators/base_orchestrator.py:26`

**Purpose**: Base class for domain-specific orchestrators providing common workflow patterns.

**Core Features**:
- Phase execution and management
- Agent lifecycle management
- Output directory structure
- Progress tracking and visualization
- Cost tracking and reporting
- Verification and checkpointing
- Error handling and recovery

**Constructor Parameters**:
- `domain_name: str` - Name of the domain (e.g., 'architecture', 'ux')
- `output_base_dir: Path` - Base directory for outputs (default: `Path("outputs")`)
- `show_tool_details: bool` - Whether to display tool usage (default: `True`)

**Key Methods**:
- `create_output_structure(subdirs)` - Create output directory structure (`base_orchestrator.py:62`)
- `display_message(msg, show_tools)` - Display message with tool usage visibility (`base_orchestrator.py:74`)
- `display_phase_header(phase_number, phase_name, emoji)` - Display formatted phase header (`base_orchestrator.py:117`)
- `track_phase_cost(phase_name, cost)` - Track cost for specific phase (`base_orchestrator.py:129`)
- `mark_phase_complete(phase_name)` - Mark phase as completed (`base_orchestrator.py:139`)
- `verify_outputs(expected_files)` - Verify all expected outputs created (`base_orchestrator.py:147`)
- `display_summary()` - Display orchestrator run summary (`base_orchestrator.py:172`)
- `execute_phase(phase_name, agent_name, prompt, client)` - Execute single phase (`base_orchestrator.py:216`)
- `create_client_options(permission_mode, cwd)` - Create Claude SDK client options (`base_orchestrator.py:242`)
- `run_with_client()` - Run orchestrator with automatic client setup/teardown (`base_orchestrator.py:266`)

**Abstract Methods** (must be implemented by subclasses):
- `get_agent_definitions()` - Return dict of AgentDefinition objects (`base_orchestrator.py:190`)
- `get_allowed_tools()` - Return list of allowed tool names (`base_orchestrator.py:199`)
- `run()` - Execute the orchestrator workflow (`base_orchestrator.py:208`)

---

#### 4. ArchitectureOrchestrator
**File**: `orchestrators/architecture_orchestrator.py:20`

**Purpose**: Orchestrator for comprehensive repository architecture analysis.

**Inherits**: `BaseOrchestrator`

**Output Structure**:
- `docs/` - Analysis documentation
- `diagrams/` - Architecture diagrams
- `reports/` - Summary reports

**Agents**:
- `analyzer` - Analyzes code structure, patterns, architecture (`architecture_orchestrator.py:62`)
  - Tools: `["Read", "Grep", "Glob", "Write", "Bash"]`
  - Model: `sonnet`
- `doc-writer` - Writes comprehensive technical documentation (`architecture_orchestrator.py:80`)
  - Tools: `["Read", "Write", "Grep", "Glob"]`
  - Model: `sonnet`

**Allowed Tools**: `["Read", "Write", "Grep", "Glob", "Bash"]` (`architecture_orchestrator.py:109`)

**Workflow Phases**:
1. `phase_1_component_inventory()` - Generate component inventory (`architecture_orchestrator.py:111`)
2. `phase_2_architecture_diagrams()` - Generate architecture diagrams (`architecture_orchestrator.py:144`)
3. `phase_3_data_flows()` - Document data flows (`architecture_orchestrator.py:183`)
4. `phase_4_api_documentation()` - Generate API documentation (`architecture_orchestrator.py:220`)
5. `phase_5_synthesis()` - Create final synthesis document (`architecture_orchestrator.py:241`)

**Entry Point**: `main()` at `architecture_orchestrator.py:304` (async)

---

#### 5. UXOrchestrator
**File**: `orchestrators/ux_orchestrator.py:21`

**Purpose**: Orchestrator for comprehensive UX/UI design workflow.

**Inherits**: `BaseOrchestrator`

**Constructor Parameters**:
- `project_name: str` - Name of project being designed (default: "Project")
- `output_base_dir: Path` - Base directory for outputs (default: `Path("outputs")`)
- `show_tool_details: bool` - Display tool usage (default: `True`)

**Output Structure**:
- `01_research/` - User research and personas
- `02_ia/` - Information architecture
- `03_design/` - Visual design specs
- `04_prototypes/` - Interactive prototypes
- `05_api_contracts/` - API specifications
- `06_design_system/` - Design system documentation

**Agents**:
- `ux-researcher` - Conducts user research, creates personas (`ux_orchestrator.py:72`)
  - Tools: `["Read", "Write", "Grep", "Glob", "WebSearch"]`
  - Model: `sonnet`
- `ia-architect` - Designs information architecture, sitemaps (`ux_orchestrator.py:90`)
  - Tools: `["Read", "Write", "Grep", "Glob"]`
  - Model: `sonnet`
- `ui-designer` - Creates visual designs and mockups (`ux_orchestrator.py:108`)
  - Tools: `["Read", "Write", "Grep", "Glob"]`
  - Model: `sonnet`
- `prototype-developer` - Creates interactive prototypes (`ux_orchestrator.py:129`)
  - Tools: `["Read", "Write", "Grep", "Glob", "Bash"]`
  - Model: `sonnet`

**Allowed Tools**: `["Read", "Write", "Grep", "Glob", "Bash", "WebSearch"]` (`ux_orchestrator.py:163`)

**Workflow Phases**:
1. `phase_1_ux_research()` - UX research, personas, user journeys (`ux_orchestrator.py:165`)
2. `phase_2_information_architecture()` - Sitemaps, navigation, content structure (`ux_orchestrator.py:215`)
3. `phase_3_visual_design()` - High-fidelity mockups and design specs (`ux_orchestrator.py:273`)
4. `phase_4_interactive_prototyping()` - Working prototypes and interactions (`ux_orchestrator.py:342`)
5. `phase_5_api_contract_design()` - Frontend-backend interface specs (`ux_orchestrator.py:411`)
6. `phase_6_design_system_documentation()` - Component library and style guide (`ux_orchestrator.py:487`)

**Entry Point**: `main()` at `ux_orchestrator.py:607` (async)

---

#### 6. CrossOrchestratorCommunication (Mixin)
**File**: `orchestrators/base_orchestrator.py:298`

**Purpose**: Mixin for orchestrators that need cross-domain communication.

**Key Methods**:
- `register_orchestrator(name, orchestrator)` - Register orchestrator for cross-domain communication (`base_orchestrator.py:305`)
- `invoke_orchestrator(orchestrator_name, phase_name, context)` - Invoke another orchestrator (`base_orchestrator.py:314`)

**Usage Pattern**: Can be inherited alongside BaseOrchestrator for orchestrators needing cross-domain validation or coordination.

---

### Agent Management

#### 7. AgentRegistry
**File**: `agents/registry.py:10`

**Purpose**: Registry for discovering and loading agent definitions from JSON files.

**Constructor Parameters**:
- `agents_dir: Path` - Base directory containing agent definitions (default: package directory)

**Instance Variables**:
- `self._cache: Dict[str, AgentDefinition]` - Cache for loaded agents

**Key Methods**:
- `discover_agents(domain)` - Discover all available agent definition files (`registry.py:22`)
  - Returns: Dictionary mapping agent names to file paths
  - Scans subdirectories for `*.json` files
  - Skips directories starting with `_`
- `load_agent(agent_name, domain)` - Load agent definition from JSON file (`registry.py:45`)
  - Uses cache to avoid re-loading
  - Returns: `AgentDefinition` object or `None` if not found
- `load_domain_agents(domain)` - Load all agents for specific domain (`registry.py:82`)
  - Returns: Dictionary mapping agent names to `AgentDefinition` objects

**Caching**: Uses `self._cache` with keys like `"{domain}/{agent_name}"` to avoid re-loading agent definitions.

---

### Tool Integration

#### 8. MCPRegistry
**File**: `tools/mcp_registry.py:8`

**Purpose**: Registry for discovering and managing MCP server connections.

**Registered Servers** (`mcp_registry.py:26-51`):
- `figma` - Figma MCP Server for design context
  - Tools: `["figma_get_file", "figma_get_components"]`
  - Requires: `FIGMA_ACCESS_TOKEN`
- `v0` - Vercel v0 MCP Server for UI generation
  - Tools: `["v0_generate_ui", "v0_generate_from_image", "v0_chat_complete"]`
  - Requires: `V0_API_KEY`
- `sequential-thinking` - Advanced reasoning MCP tool
  - Tools: `["sequentialthinking"]`
  - No configuration required
- `playwright` - Browser automation MCP tool
  - Tools: `["browser_navigate", "browser_click", "browser_snapshot"]`
  - No configuration required

**Key Methods**:
- `discover_mcp_servers()` - Auto-discover available MCP servers (`mcp_registry.py:16`)
- `is_server_available(server_name)` - Check if MCP server is available (`mcp_registry.py:55`)
- `get_server_tools(server_name)` - Get list of tools provided by server (`mcp_registry.py:69`)
- `validate_tool_availability(tool_name)` - Validate if specific tool is available (`mcp_registry.py:83`)
- `get_configuration_requirements(server_name)` - Get configuration requirements (`mcp_registry.py:98`)
  - Returns configuration template with required env vars and setup instructions
- `get_fallback_options(tool_name)` - Get fallback options if tool unavailable (`mcp_registry.py:130`)
  - Returns list of alternative approaches

---

#### 9. FigmaIntegration
**File**: `tools/figma_integration.py:7`

**Purpose**: Wrapper for Figma MCP server and REST API integration.

**Environment Variables**:
- `FIGMA_ACCESS_TOKEN` - Figma personal access token (checked in `__init__`)

**Instance Variables**:
- `self.access_token` - Figma access token from environment
- `self.mcp_available` - Boolean indicating MCP server availability

**Key Methods**:
- `is_available()` - Check if Figma integration is available (`figma_integration.py:15`)
  - Returns: `True` if token exists or MCP is available
- `get_design_context(file_id)` - Get design context from Figma file (`figma_integration.py:23`)
  - Args: `file_id` - Figma file ID
  - Returns: Design context dict or error with fallback instructions
- `export_to_code(component_id, framework)` - Export Figma component to code (`figma_integration.py:59`)
  - Args: `component_id` - Figma component ID, `framework` - Target framework (default: "react")
  - Returns: Generated code string or `None` if unavailable
- `create_component(spec)` - Create Figma component from specification (`figma_integration.py:86`)
  - Args: `spec` - Component specification dictionary
  - Returns: Component ID or `None` if MCP unavailable
- `get_setup_instructions()` - Get setup instructions for Figma integration (`figma_integration.py:103`)
  - Returns: Markdown-formatted setup instructions (3 options: MCP, REST API, Manual)

**Note**: Current implementation is a stub/skeleton with detailed instructions for future integration.

---

### Data Management

#### 10. Neo4jDataImporter
**File**: `import_data.py:22`

**Purpose**: Imports psychological intelligence data and schema into Neo4j for MCP standalone.

**Constructor Parameters**:
- `uri: str` - Neo4j connection URI
- `user: str` - Neo4j username
- `password: str` - Neo4j password
- `max_retries: int` - Maximum connection retry attempts (default: 30)

**Key Methods**:
- `_connect_with_retry()` - Connect to Neo4j with retry logic (`import_data.py:34`)
  - Retries every 2 seconds for container startup
- `close()` - Close Neo4j connection (`import_data.py:51`)
- `clear_database()` - Clear all existing data (`import_data.py:56`)
  - Runs: `MATCH (n) DETACH DELETE n`
- `load_schema(schema_path)` - Load schema constraints, indexes, and persona data (`import_data.py:63`)
  - Creates constraints for PersonaAgent, Memory, Goal
  - Creates indexes for performance (attachment_style, memory_type, etc.)
  - Calls `_load_family_graph_data()` to import personas
- `_load_family_graph_data(schema, session)` - Load personas and relationships from JSON (`import_data.py:122`)
  - Parses `family_graph` structure from schema JSON
  - Maps behavioral styles to Big Five personality traits
  - Creates PersonaAgent nodes and RELATIONSHIP edges
- `_map_behavioral_to_bigfive(behavioral_style)` - Map DISC to Big Five traits (`import_data.py:239`)
  - Maps D (Dominance), I (Influence), S (Steadiness), C (Conscientiousness)
  - Returns dict with openness, conscientiousness, extraversion, agreeableness, neuroticism
- `import_seed_data(seed_data_path)` - Import seed data from Cypher file (`import_data.py:282`)
  - Splits Cypher script into statements (semicolon-separated)
  - Executes each statement sequentially
- `create_default_personas()` - Create default personas if no data imported (`import_data.py:312`)
  - Creates Lila (AI Research Assistant, secure attachment)
  - Creates Alex (Software Engineer, secure attachment)
  - Creates friendship relationship between them
- `verify_import()` - Verify data was imported successfully (`import_data.py:379`)
  - Counts personas, relationships, memories, goals
  - Prints verification summary

**Entry Point**: `main()` at `import_data.py:409`

**CLI Arguments**:
- `--seed-data` - Seed data Cypher file (default: `seed_data.cypher`)
- `--schema` - Schema JSON file (default: `graphs/lila-graph-schema-v8.json`)
- `--uri` - Neo4j URI (default: `bolt://localhost:7687`)
- `--user` - Neo4j username (default: `neo4j`)
- `--password` - Neo4j password (or use `NEO4J_PASSWORD` env var)
- `--create-defaults` - Create default personas if no seed data found

---

#### 11. Neo4jDataExporter
**File**: `export_data.py:21`

**Purpose**: Exports psychological intelligence data from Neo4j for MCP standalone seeding.

**Constructor Parameters**:
- `uri: str` - Neo4j connection URI
- `user: str` - Neo4j username
- `password: str` - Neo4j password

**Key Methods**:
- `close()` - Close Neo4j connection (`export_data.py:28`)
- `export_personas()` - Export all PersonaAgent nodes (`export_data.py:32`)
  - Returns: List of persona dictionaries with all properties
  - Query: `MATCH (p:PersonaAgent) RETURN p.*`
- `export_relationships()` - Export all relationships between personas (`export_data.py:64`)
  - Returns: List of relationship dictionaries
  - Query: `MATCH (p1:PersonaAgent)-[r:RELATIONSHIP]->(p2:PersonaAgent)`
- `export_memories()` - Export memory nodes associated with personas (`export_data.py:91`)
  - Returns: List of memory dictionaries
  - Query: `MATCH (p:PersonaAgent)-[:HAS_MEMORY]->(m:Memory)`
- `export_goals()` - Export goal nodes associated with personas (`export_data.py:115`)
  - Returns: List of goal dictionaries
  - Query: `MATCH (p:PersonaAgent)-[:HAS_GOAL]->(g:Goal)`
- `generate_cypher_script(personas, relationships, memories, goals)` - Generate Cypher import script (`export_data.py:141`)
  - Creates complete Cypher script with CREATE statements
  - Handles escaping of quotes and newlines
  - Returns: Multi-line Cypher script string

**Entry Point**: `main()` at `export_data.py:245`

**CLI Arguments**:
- `--output` - Output Cypher file (default: `seed_data.cypher`)
- `--uri` - Neo4j URI (default: `bolt://localhost:7687`)
- `--user` - Neo4j username (default: `neo4j`)
- `--password` - Neo4j password (or use `NEO4J_PASSWORD` env var)

---

## Internal Implementation

### Package Initialization Files

#### 12. orchestrators/__init__.py
**File**: `orchestrators/__init__.py:1`

**Exports**: `BaseOrchestrator`

```python
from .base_orchestrator import BaseOrchestrator
__all__ = ["BaseOrchestrator"]
```

---

#### 13. agents/__init__.py
**File**: `agents/__init__.py:1`

**Exports**: `AgentRegistry`

```python
from .registry import AgentRegistry
__all__ = ["AgentRegistry"]
```

---

#### 14. tools/__init__.py
**File**: `tools/__init__.py:1`

**Exports**: `MCPRegistry`, `FigmaIntegration`

```python
from .mcp_registry import MCPRegistry
from .figma_integration import FigmaIntegration
__all__ = ["MCPRegistry", "FigmaIntegration"]
```

---

### Internal Utilities

#### 15. architecture.py (Legacy/Original)
**File**: `architecture.py:1`

**Purpose**: Original architecture analysis script (now refactored into ArchitectureOrchestrator).

**Note**: This appears to be the original implementation before the orchestrator framework was extracted. The functionality has been moved to `orchestrators/architecture_orchestrator.py` and `orchestrators/base_orchestrator.py`.

**Function**: `display_message(msg, show_tools)` - Display message with tool visibility (`architecture.py:33`)

**Entry Point**: `if __name__ == "__main__"` at `architecture.py:361`

---

### Test Modules

#### 16. test_orchestrators.py
**File**: `test_orchestrators.py:1`

**Purpose**: Validation tests for multi-domain orchestrator system without requiring Claude API calls.

**Test Functions**:
- `test_imports()` - Test that all modules can be imported (`test_orchestrators.py:7`)
- `test_agent_registry()` - Test agent registry functionality (`test_orchestrators.py:36`)
  - Verifies discovery of agents
  - Tests loading of specific agents
  - Checks agent tools and configuration
- `test_mcp_registry()` - Test MCP registry functionality (`test_orchestrators.py:81`)
  - Verifies expected servers registered
  - Tests availability checking
- `test_orchestrator_instantiation()` - Test orchestrator instantiation (`test_orchestrators.py:113`)
  - Creates ArchitectureOrchestrator instance
  - Creates UXOrchestrator instance
  - Verifies agent and tool configuration
- `test_figma_integration()` - Test Figma integration setup (`test_figma_integration.py:142`)
  - Verifies FigmaIntegration can be created
  - Checks setup instructions availability

**Entry Point**: `main()` at `test_orchestrators.py:167`

**Expected Agents**:
```python
expected_agents = {
    'ux_researcher', 'ia_architect', 'ui_designer', 'prototype_developer',
    'analyzer', 'doc_writer'
}
```

---

## Entry Points

### 1. MCP Server Entry Points

#### Run LilaMCPServer
```bash
python lila_mcp_server.py
```
**File**: `lila_mcp_server.py:778`
**Function**: Starts the FastMCP server with Neo4j backend on localhost:8765

```python
if __name__ == "__main__":
    asyncio.run(main())
```

#### Run SimpleLilaMCPServer
```bash
python simple_lila_mcp_server.py
# Or using FastMCP CLI:
fastmcp dev simple_lila_mcp_server.py
```
**File**: `simple_lila_mcp_server.py:827`
**Function**: Starts the simplified MCP server with mock data

```python
if __name__ == "__main__":
    # For testing purposes
    print("Simple Lila MCP Server ready for FastMCP Inspector")
    print("Run with: fastmcp dev simple_lila_mcp_server.py")
```

---

### 2. Orchestrator Entry Points

#### Run Architecture Analysis
```bash
python -m orchestrators.architecture_orchestrator
```
**File**: `orchestrators/architecture_orchestrator.py:310`
**Function**: Runs comprehensive repository architecture analysis

```python
if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

#### Run UX Design Workflow
```bash
python -m orchestrators.ux_orchestrator "Project Name"
```
**File**: `orchestrators/ux_orchestrator.py:616`
**Function**: Runs comprehensive UX design workflow for specified project

```python
if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

---

### 3. Data Management Entry Points

#### Import Data to Neo4j
```bash
python import_data.py --schema graphs/lila-graph-schema-v8.json --create-defaults
```
**File**: `import_data.py:465`
**Function**: Imports schema and seed data into Neo4j database

**Arguments**:
- `--seed-data` - Seed data Cypher file (default: `seed_data.cypher`)
- `--schema` - Schema JSON file (default: `graphs/lila-graph-schema-v8.json`)
- `--uri` - Neo4j URI (default: `bolt://localhost:7687`)
- `--user` - Neo4j username (default: `neo4j`)
- `--password` - Neo4j password (or use `NEO4J_PASSWORD` env var)
- `--create-defaults` - Create default personas if no seed data found

#### Export Data from Neo4j
```bash
python export_data.py --output seed_data.cypher
```
**File**: `export_data.py:294`
**Function**: Exports personas, relationships, memories, and goals from Neo4j to Cypher file

**Arguments**:
- `--output` - Output Cypher file (default: `seed_data.cypher`)
- `--uri` - Neo4j URI (default: `bolt://localhost:7687`)
- `--user` - Neo4j username (default: `neo4j`)
- `--password` - Neo4j password (or use `NEO4J_PASSWORD` env var)

---

### 4. Test Entry Points

#### Run Orchestrator Tests
```bash
python test_orchestrators.py
```
**File**: `test_orchestrators.py:209`
**Function**: Validates all components without requiring Claude API calls

**Tests**:
- Import validation for all modules
- AgentRegistry functionality
- MCPRegistry functionality
- Orchestrator instantiation
- FigmaIntegration setup

```python
if __name__ == "__main__":
    import sys
    sys.exit(main())
```

---

## Module Organization

### Public Modules (Exported in __all__)

```
orchestrators/
  __init__.py                  → exports: BaseOrchestrator
  base_orchestrator.py         → BaseOrchestrator, CrossOrchestratorCommunication
  architecture_orchestrator.py → ArchitectureOrchestrator
  ux_orchestrator.py           → UXOrchestrator

agents/
  __init__.py                  → exports: AgentRegistry
  registry.py                  → AgentRegistry

tools/
  __init__.py                  → exports: MCPRegistry, FigmaIntegration
  mcp_registry.py              → MCPRegistry
  figma_integration.py         → FigmaIntegration
```

### Standalone Modules (Direct Execution)

```
lila_mcp_server.py             → LilaMCPServer (main MCP server)
simple_lila_mcp_server.py      → SimpleLilaMCPServer (mock data server)
import_data.py                 → Neo4jDataImporter
export_data.py                 → Neo4jDataExporter
test_orchestrators.py          → Test suite
```

### Legacy/Reference Modules

```
architecture.py                → Original analysis script (pre-orchestrator framework)
```

---

## Design Patterns

### 1. Abstract Base Class Pattern
- `BaseOrchestrator` defines common orchestrator interface
- Subclasses implement domain-specific workflows
- Enforces consistent API through abstract methods: `get_agent_definitions()`, `get_allowed_tools()`, `run()`

### 2. Registry Pattern
- `AgentRegistry` - Centralized agent discovery and loading
  - Discovers JSON agent definitions from filesystem
  - Caches loaded agents to avoid re-parsing
- `MCPRegistry` - Centralized MCP server management
  - Tracks available servers and their capabilities
  - Provides fallback options for unavailable tools
- Enables dynamic configuration and extensibility

### 3. Mixin Pattern
- `CrossOrchestratorCommunication` - Adds cross-domain communication capabilities
- Can be mixed into orchestrators that need inter-orchestrator coordination
- Maintains registry of orchestrators and provides invocation interface

### 4. Strategy Pattern
- Multiple orchestrators (Architecture, UX) implement different domain strategies
- Same base interface (`BaseOrchestrator`), different workflow implementations
- Each orchestrator defines its own agents, tools, and phase execution

### 5. Template Method Pattern
- `BaseOrchestrator.run_with_client()` provides template workflow:
  1. Display header
  2. Create client options
  3. Create client context
  4. Call subclass `run()` method
  5. Display summary
  6. Clean up client
- Subclasses implement specific `run()` method with phases

### 6. Facade Pattern
- `FigmaIntegration` provides unified interface to Figma MCP/API
- Hides complexity of multiple integration methods (MCP server, REST API, manual)
- Provides fallback instructions when integration unavailable

### 7. Dependency Injection
- Orchestrators receive `ClaudeSDKClient` through `run_with_client()`
- Agents receive tool lists through `AgentDefinition`
- Allows for easier testing and flexibility

---

## Dependencies

### External Libraries (from imports)

**Core SDK**:
- `claude_agent_sdk` - Claude Agent SDK for orchestration
  - `AgentDefinition`, `ClaudeAgentOptions`, `ClaudeSDKClient`
  - Message types: `AssistantMessage`, `UserMessage`, `ResultMessage`
  - Block types: `TextBlock`, `ToolUseBlock`, `ToolResultBlock`

**MCP Framework**:
- `fastmcp` - FastMCP server framework
  - `FastMCP` - Main server class
  - Decorators: `@app.resource()`, `@app.tool()`, `@app.prompt()`, `@app.custom_route()`

**Database**:
- `neo4j` - Neo4j Python driver
  - `GraphDatabase` - Connection driver

**HTTP Server**:
- `starlette` - ASGI framework (used by FastMCP)
  - `Request`, `Response`, `JSONResponse` - HTTP primitives

**Standard Library**:
- `asyncio` - Async/await support
- `pathlib` - Path operations (`Path`)
- `typing` - Type hints (`Dict`, `List`, `Optional`, `Any`, `Union`)
- `abc` - Abstract base classes (`ABC`, `abstractmethod`)
- `json` - JSON parsing
- `argparse` - CLI argument parsing
- `logging` - Logging framework
- `os` - Environment variables, file operations
- `subprocess` - Process management
- `sys` - System operations, exit codes
- `time` - Sleep/retry delays
- `datetime` - Timestamps (`datetime`)

**Environment**:
- `dotenv` - Environment variable management (`load_dotenv`)

---

## Configuration

### Environment Variables

**Neo4j Connection**:
- `NEO4J_URI` - Neo4j connection URI (default: `bolt://localhost:7687`)
- `NEO4J_USER` - Neo4j username (default: `neo4j`)
- `NEO4J_PASSWORD` - Neo4j password (default: `passw0rd`)

**Figma Integration**:
- `FIGMA_ACCESS_TOKEN` - Figma personal access token (optional)
- `FIGMA_FILE_ID` - Figma file ID (optional)

**Vercel v0 Integration**:
- `V0_API_KEY` - Vercel v0 API key (optional)

**Logging**:
- FastMCP debug logging enabled in `simple_lila_mcp_server.py:28-29`:
  ```python
  logging.basicConfig(level=logging.DEBUG)
  logging.getLogger("fastmcp.server").setLevel(logging.DEBUG)
  ```

---

## Summary

### Component Counts

- **MCP Servers**: 2 (LilaMCPServer, SimpleLilaMCPServer)
- **Orchestrators**: 3 (Base, Architecture, UX) + 1 mixin (CrossOrchestratorCommunication)
- **Registries**: 2 (AgentRegistry, MCPRegistry)
- **Integrations**: 1 (FigmaIntegration)
- **Data Importers/Exporters**: 2 (Neo4jDataImporter, Neo4jDataExporter)
- **Test Modules**: 1 (test_orchestrators)
- **Legacy Modules**: 1 (architecture.py)

### Public API Surface

**Classes**: 11 public classes
- 2 MCP servers
- 4 orchestrators (including base + mixin)
- 2 registries
- 1 integration wrapper
- 2 data management classes

**Entry Points**: 8 main entry points
- 2 MCP server runners
- 2 orchestrator runners
- 2 data management scripts
- 1 test runner
- 1 legacy script

**Total Python Files**: 17
- 11 implementation files
- 3 `__init__.py` files
- 2 test files
- 1 legacy file

### Internal Implementation

**Private Methods**: Prefixed with `_`
- `_setup_database()`, `_connect_with_retry()`, `_load_family_graph_data()`, `_map_behavioral_to_bigfive()`

**Internal Modules**:
- `architecture.py` (legacy implementation)
- Package `__init__.py` files (define public exports)

---

## Usage Patterns

### Starting an MCP Server
```python
from lila_mcp_server import LilaMCPServer
import asyncio

server = LilaMCPServer()
asyncio.run(server.run_server())
```

### Running an Orchestrator
```python
from orchestrators.architecture_orchestrator import ArchitectureOrchestrator
import asyncio

orchestrator = ArchitectureOrchestrator()
asyncio.run(orchestrator.run_with_client())
```

### Loading Agents
```python
from agents.registry import AgentRegistry

registry = AgentRegistry()
ux_researcher = registry.load_agent('ux_researcher', domain='ux')
print(f"Tools: {ux_researcher.tools}")
```

### Checking MCP Tool Availability
```python
from tools.mcp_registry import MCPRegistry

mcp = MCPRegistry()
if mcp.is_server_available('figma'):
    tools = mcp.get_server_tools('figma')
    print(f"Figma tools: {tools}")
else:
    fallbacks = mcp.get_fallback_options('figma_get_file')
    print(f"Fallbacks: {fallbacks}")
```

### Creating a Custom Orchestrator
```python
from orchestrators.base_orchestrator import BaseOrchestrator
from claude_agent_sdk import AgentDefinition
from typing import Dict, List

class CustomOrchestrator(BaseOrchestrator):
    def __init__(self):
        super().__init__(domain_name="custom", output_base_dir=Path("outputs"))

    def get_agent_definitions(self) -> Dict[str, AgentDefinition]:
        return {
            "my-agent": AgentDefinition(
                description="Custom agent",
                prompt="You are a custom agent...",
                tools=["Read", "Write"],
                model="sonnet"
            )
        }

    def get_allowed_tools(self) -> List[str]:
        return ["Read", "Write", "Grep", "Glob"]

    async def run(self):
        await self.execute_phase(
            phase_name="Phase 1",
            agent_name="my-agent",
            prompt="Do something...",
            client=self.client
        )
```

---

This component inventory provides a complete map of the codebase's structure, making it easier to navigate, extend, and maintain the system.
