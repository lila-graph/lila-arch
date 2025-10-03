# Component Inventory

## Public API

### MCP Server Modules

#### `lila_mcp_server.py` - Main MCP Server (Lines 1-779)
**Purpose**: FastMCP server exposing psychological relationship intelligence data via Neo4j

**Public Classes**:
- `LilaMCPServer` (Line 29-777)
  - Main server class providing MCP resources, tools, and prompts
  - Entry point: `main()` function (Line 768-776)
  - Module-level instance: `mcp` (Line 766)

**Public Resources** (Lines 72-295):
- `neo4j://personas/all` (Line 72) - Retrieve all personas with psychological profiles
- `neo4j://personas/{persona_id}` (Line 116) - Get specific persona by ID
- `neo4j://relationships/all` (Line 163) - All relationships with metrics
- `neo4j://relationships/{persona1_id}/{persona2_id}` (Line 208) - Specific relationship
- `neo4j://interactions/recent/{count}` (Line 255) - Recent interactions

**Public Tools** (Lines 300-608):
- `update_relationship_metrics()` (Line 301) - Update trust/intimacy/strength metrics
- `record_interaction()` (Line 363) - Record interaction between personas
- `analyze_persona_compatibility()` (Line 402) - Assess relationship potential
- `autonomous_strategy_selection()` (Line 464) - AI-driven strategy selection
- `assess_goal_progress()` (Line 516) - Track relationship goals
- `generate_contextual_response()` (Line 555) - Generate psychologically authentic responses

**Public Prompts** (Lines 613-739):
- `assess_attachment_style()` (Line 614) - Determine attachment style from behavior
- `analyze_emotional_climate()` (Line 646) - Evaluate conversation dynamics
- `generate_secure_response()` (Line 697) - Create attachment-security-building responses

**Health Check**:
- `/health` endpoint (Line 746-754) - HTTP health check for orchestration

#### `simple_lila_mcp_server.py` - Simplified MCP Server (Lines 1-830)
**Purpose**: Standalone MCP server with in-memory mock data for testing

**Public Classes**:
- `SimpleLilaMCPServer` (Line 33-823)
  - Simplified version with mock personas and relationships
  - Same API surface as main server but without Neo4j dependency
  - Module-level instance: `mcp` (Line 825)

**Mock Data Storage** (Lines 42-92):
- `mock_personas` - In-memory persona storage
- `mock_relationships` - In-memory relationship metrics
- `mock_interactions` - Interaction history

**Additional Resources** (compared to main server):
- `neo4j://emotional_climate/current` (Line 271) - Current emotional climate
- `neo4j://attachment_styles/analysis` (Line 292) - Attachment compatibility matrix
- `neo4j://goals/active` (Line 311) - Active relationship goals
- `neo4j://psychological_insights/trends` (Line 334) - Psychological trends

**Additional Tools**:
- `commit_relationship_state()` (Line 610) - Commit relationship state
- `finalize_demo_session()` (Line 622) - Finalize demo session

### Data Management Modules

#### `import_data.py` - Neo4j Data Importer (Lines 1-466)
**Purpose**: Import seed data and schema into Neo4j database

**Public Classes**:
- `Neo4jDataImporter` (Line 22-406)
  - Connection management with retry logic (Line 25-49)
  - Schema loading from JSON (Line 63-120)
  - Family graph data import (Line 122-237)
  - Default persona creation (Line 312-377)

**Public Functions**:
- `main()` (Line 409-462) - CLI entry point with argument parsing

**Entry Point**: Command-line script with `--seed-data`, `--schema`, `--uri`, `--user`, `--password` flags

#### `export_data.py` - Neo4j Data Exporter (Lines 1-295)
**Purpose**: Export Neo4j data to Cypher seed files

**Public Classes**:
- `Neo4jDataExporter` (Line 21-242)
  - Export personas, relationships, memories, goals
  - Generate Cypher import scripts

**Public Functions**:
- `main()` (Line 245-291) - CLI entry point

**Entry Point**: Command-line script with `--output`, `--uri`, `--user`, `--password` flags

### Repository Analyzer Framework

#### `ra_orchestrators/base_orchestrator.py` - Base Orchestrator (Lines 1-357)
**Purpose**: Foundational framework for multi-domain agent orchestration

**Public Classes**:
- `BaseOrchestrator` (Line 30-309) - Abstract base for domain orchestrators
  - Phase execution management
  - Output directory structure creation
  - Progress tracking and cost reporting
  - Verification and checkpointing

**Abstract Methods**:
- `get_agent_definitions()` (Line 202-209) - Must return agent definitions
- `get_allowed_tools()` (Line 211-218) - Must return allowed tools list
- `run()` (Line 220-227) - Must implement workflow

**Public Methods**:
- `create_output_structure()` (Line 75-85) - Create output directories
- `display_message()` (Line 87-120) - Display agent messages with tool visibility
- `display_phase_header()` (Line 130-140) - Display phase headers
- `track_phase_cost()` (Line 142-150) - Track phase costs
- `mark_phase_complete()` (Line 152-158) - Mark phase completion
- `verify_outputs()` (Line 160-183) - Verify expected outputs
- `display_summary()` (Line 185-200) - Display orchestrator summary
- `execute_phase()` (Line 229-253) - Execute workflow phase
- `create_client_options()` (Line 255-277) - Create Claude SDK options
- `run_with_client()` (Line 279-308) - Run with automatic client setup

**Mixins**:
- `CrossOrchestratorCommunication` (Line 311-356) - Inter-orchestrator communication

#### `ra_orchestrators/architecture_orchestrator.py` - Architecture Analysis (Lines 1-316)
**Purpose**: Comprehensive repository architecture analysis orchestrator

**Public Classes**:
- `ArchitectureOrchestrator` (Line 20-305) - Extends BaseOrchestrator
  - 5-phase analysis workflow
  - Output directories: docs/, diagrams/, reports/

**Workflow Phases**:
1. `phase_1_component_inventory()` (Line 114-145) - Component inventory
2. `phase_2_architecture_diagrams()` (Line 147-184) - Architecture diagrams
3. `phase_3_data_flows()` (Line 186-221) - Data flow analysis
4. `phase_4_api_documentation()` (Line 223-242) - API documentation
5. `phase_5_synthesis()` (Line 244-284) - Final synthesis

**Entry Point**: `main()` function (Line 307-310) - `if __name__ == "__main__"`

#### `ra_orchestrators/ux_orchestrator.py` - UX Design Workflow (Lines 1-623)
**Purpose**: Comprehensive UX/UI design workflow orchestrator

**Public Classes**:
- `UXOrchestrator` (Line 21-608) - Extends BaseOrchestrator
  - 6-phase design workflow
  - Project-specific output structure

**Workflow Phases**:
1. `phase_1_ux_research()` (Line 169-217) - User research & personas
2. `phase_2_information_architecture()` (Line 219-274) - IA & sitemaps
3. `phase_3_visual_design()` (Line 276-344) - Visual design & mockups
4. `phase_4_interactive_prototyping()` (Line 346-413) - Interactive prototypes
5. `phase_5_api_contract_design()` (Line 415-489) - API specifications
6. `phase_6_design_system_documentation()` (Line 491-586) - Design system docs

**Entry Point**: `main()` function (Line 611-617) - Accepts project name as CLI argument

### Agent Registry System

#### `ra_agents/registry.py` - Agent Registry (Lines 1-100)
**Purpose**: Discover and load agent definitions from JSON files

**Public Classes**:
- `AgentRegistry` (Line 10-99)
  - Agent discovery by domain (Line 22-43)
  - Agent loading with caching (Line 45-80)
  - Domain-wide agent loading (Line 82-99)

**Public Methods**:
- `discover_agents()` (Line 22) - Find all agent JSON files
- `load_agent()` (Line 45) - Load single agent definition
- `load_domain_agents()` (Line 82) - Load all domain agents

### Tool Integration Layer

#### `ra_tools/mcp_registry.py` - MCP Tool Registry (Lines 1-153)
**Purpose**: Discover and manage MCP server connections

**Public Classes**:
- `MCPRegistry` (Line 8-152)
  - MCP server discovery and validation
  - Tool availability checking
  - Configuration requirements

**Registered MCP Servers** (Lines 26-51):
- `figma` - Figma MCP Server for design context
- `v0` - Vercel v0 MCP Server for UI generation
- `sequential-thinking` - Advanced reasoning tool
- `playwright` - Browser automation

**Public Methods**:
- `discover_mcp_servers()` (Line 16) - Auto-discover MCP servers
- `is_server_available()` (Line 55) - Check server availability
- `get_server_tools()` (Line 69) - Get tools for server
- `validate_tool_availability()` (Line 83) - Validate tool availability
- `get_configuration_requirements()` (Line 98) - Get config requirements
- `get_fallback_options()` (Line 130) - Get fallback approaches

#### `ra_tools/figma_integration.py` - Figma Integration (Lines 1-157)
**Purpose**: Wrapper for Figma MCP server and REST API

**Public Classes**:
- `FigmaIntegration` (Line 7-156)
  - Figma MCP and API wrapper
  - Design-to-code integration stub

**Public Methods**:
- `is_available()` (Line 15) - Check Figma availability
- `get_design_context()` (Line 23) - Get design context from file
- `export_to_code()` (Line 59) - Export component to code
- `create_component()` (Line 86) - Create Figma component
- `get_setup_instructions()` (Line 103) - Setup instructions

### Testing Modules

#### `test_mcp_validation.py` - MCP Server Validation (Lines 1-172)
**Purpose**: Comprehensive validation for MCP server functionality

**Public Functions**:
- `test_direct_connection()` (Line 15-96) - Test in-memory connection
- `test_inspector_connection()` (Line 98-132) - Test HTTP Inspector connection
- `main()` (Line 134-168) - Run comprehensive validation

**Entry Point**: Async main with exit code (Line 171)

#### `test_orchestrators.py` - Orchestrator Testing (Lines 1-212)
**Purpose**: Validation tests for multi-domain orchestrator system

**Public Functions**:
- `test_imports()` (Line 7-33) - Test module imports
- `test_agent_registry()` (Line 36-78) - Test agent registry
- `test_mcp_registry()` (Line 81-110) - Test MCP registry
- `test_orchestrator_instantiation()` (Line 113-139) - Test orchestrators
- `test_figma_integration()` (Line 142-164) - Test Figma integration
- `main()` (Line 167-206) - Run all validation tests

**Entry Point**: CLI with exit code (Line 210)

## Internal Implementation

### Package Initialization Files

#### `ra_orchestrators/__init__.py` (Lines 1-6)
**Purpose**: Package initialization for orchestrators
**Exports**: `BaseOrchestrator`

#### `ra_agents/__init__.py` (Lines 1-6)
**Purpose**: Package initialization for agents
**Exports**: `AgentRegistry`

#### `ra_tools/__init__.py` (Lines 1-7)
**Purpose**: Package initialization for tools
**Exports**: `MCPRegistry`, `FigmaIntegration`

### Internal Helper Methods

#### MCP Server Internals

**LilaMCPServer Private Methods**:
- `_setup_database()` (Line 46-62) - Initialize Neo4j connection
- `_register_resources()` (Line 69-295) - Register MCP resources
- `_register_tools()` (Line 297-608) - Register MCP tools
- `_register_prompts()` (Line 610-739) - Register prompts
- `_register_health_check()` (Line 741-754) - Register health endpoint
- `close()` (Line 64-67) - Close database connection

**SimpleLilaMCPServer Private Methods**:
- Same structure as LilaMCPServer but with mock data implementation

#### Data Importer Internals

**Neo4jDataImporter Private Methods**:
- `_connect_with_retry()` (Line 34-49) - Connection retry logic
- `_load_family_graph_data()` (Line 122-237) - Load personas from JSON
- `_map_behavioral_to_bigfive()` (Line 239-280) - Map DISC to Big Five traits

#### Base Orchestrator Internals

**Tracking State** (Lines 70-73):
- `total_cost` - Cumulative cost tracking
- `phase_costs` - Per-phase cost dictionary
- `completed_phases` - List of completed phase names

**Output Directories** (Architecture Orchestrator):
- `docs_dir` (Line 45) - `ra_output/architecture_{timestamp}/docs/`
- `diagrams_dir` (Line 46) - `ra_output/architecture_{timestamp}/diagrams/`
- `reports_dir` (Line 47) - `ra_output/architecture_{timestamp}/reports/`

**Output Directories** (UX Orchestrator):
- `research_dir` (Line 50) - `01_research/`
- `ia_dir` (Line 51) - `02_ia/`
- `design_dir` (Line 52) - `03_design/`
- `prototypes_dir` (Line 53) - `04_prototypes/`
- `api_contracts_dir` (Line 54) - `05_api_contracts/`
- `design_system_dir` (Line 55) - `06_design_system/`

## Entry Points

### Main Application Entry Points

#### 1. **MCP Server (Production)** - `/home/donbr/lila-graph/lila-mcp/lila_mcp_server.py:779`
```python
if __name__ == "__main__":
    asyncio.run(main())
```
- **Purpose**: Start FastMCP server with Neo4j backend
- **Usage**: `python lila_mcp_server.py`
- **Configuration**: Requires `NEO4J_URI`, `NEO4J_USER`, `NEO4J_PASSWORD` env vars
- **Port**: Default 8765 (configurable)

#### 2. **MCP Server (Development/Testing)** - `/home/donbr/lila-graph/lila-mcp/simple_lila_mcp_server.py:827-830`
```python
if __name__ == "__main__":
    print("Simple Lila MCP Server ready for FastMCP Inspector")
    print("Run with: fastmcp dev simple_lila_mcp_server.py")
```
- **Purpose**: Start simplified MCP server with mock data
- **Usage**: `fastmcp dev simple_lila_mcp_server.py`
- **FastMCP Discovery**: Module-level `mcp` variable (Line 825)
- **Port**: 6274 (FastMCP Inspector default)

#### 3. **Data Import Tool** - `/home/donbr/lila-graph/lila-mcp/import_data.py:466`
```python
if __name__ == "__main__":
    main()
```
- **Purpose**: Import seed data and schema into Neo4j
- **Usage**: `python import_data.py --schema schema.json --seed-data data.cypher`
- **Arguments**:
  - `--seed-data`: Path to Cypher seed file (default: seed_data.cypher)
  - `--schema`: Path to schema JSON (default: graphs/lila-graph-schema-v8.json)
  - `--uri`: Neo4j URI (default: bolt://localhost:7687)
  - `--user`: Neo4j username (default: neo4j)
  - `--password`: Neo4j password (env: NEO4J_PASSWORD)
  - `--create-defaults`: Create default personas if no seed data

#### 4. **Data Export Tool** - `/home/donbr/lila-graph/lila-mcp/export_data.py:295`
```python
if __name__ == "__main__":
    main()
```
- **Purpose**: Export Neo4j data to Cypher seed files
- **Usage**: `python export_data.py --output seed_data.cypher`
- **Arguments**:
  - `--output`: Output Cypher file (default: seed_data.cypher)
  - `--uri`: Neo4j URI (default: bolt://localhost:7687)
  - `--user`: Neo4j username (default: neo4j)
  - `--password`: Neo4j password (env: NEO4J_PASSWORD)

### Repository Analyzer Entry Points

#### 5. **Architecture Orchestrator** - `/home/donbr/lila-graph/lila-mcp/ra_orchestrators/architecture_orchestrator.py:313-315`
```python
if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```
- **Purpose**: Run comprehensive repository architecture analysis
- **Usage**: `python -m ra_orchestrators.architecture_orchestrator`
- **Output**: `ra_output/architecture_{timestamp}/`
- **Phases**: 5 phases (inventory, diagrams, flows, API docs, synthesis)
- **Cost**: ~$1-3, 5-10 minutes

#### 6. **UX Orchestrator** - `/home/donbr/lila-graph/lila-mcp/ra_orchestrators/ux_orchestrator.py:620-622`
```python
if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```
- **Purpose**: Run comprehensive UX design workflow
- **Usage**: `python -m ra_orchestrators.ux_orchestrator "Project Name"`
- **Output**: `ra_output/ux_{timestamp}/`
- **Phases**: 6 phases (research, IA, design, prototyping, API, design system)
- **Cost**: ~$3-8, 10-20 minutes

### Testing Entry Points

#### 7. **MCP Validation Tests** - `/home/donbr/lila-graph/lila-mcp/test_mcp_validation.py:171-172`
```python
if __name__ == "__main__":
    success = asyncio.run(main())
    exit(0 if success else 1)
```
- **Purpose**: Validate MCP server functionality
- **Usage**: `python test_mcp_validation.py`
- **Tests**:
  - Direct in-memory connection
  - HTTP Inspector connection
  - Resources, tools, prompts availability

#### 8. **Orchestrator Validation Tests** - `/home/donbr/lila-graph/lila-mcp/test_orchestrators.py:210-211`
```python
if __name__ == "__main__":
    import sys
    sys.exit(main())
```
- **Purpose**: Validate multi-domain orchestrator system
- **Usage**: `python test_orchestrators.py`
- **Tests**:
  - Module imports
  - Agent registry functionality
  - MCP registry functionality
  - Orchestrator instantiation
  - Figma integration setup

### FastMCP Discovery

The FastMCP CLI discovers servers via module-level variables:

#### **Main Server** - `/home/donbr/lila-graph/lila-mcp/lila_mcp_server.py:765-766`
```python
_server_instance = LilaMCPServer()
mcp = _server_instance.app
```

#### **Simple Server** - `/home/donbr/lila-graph/lila-mcp/simple_lila_mcp_server.py:825`
```python
mcp = SimpleLilaMCPServer().app
```

**FastMCP Commands**:
- `fastmcp dev lila_mcp_server.py` - Start with Inspector
- `fastmcp dev simple_lila_mcp_server.py` - Start simplified version
- `fastmcp install lila_mcp_server.py` - Install to Claude Desktop

## Configuration Files

### Project Configuration

#### `pyproject.toml` - `/home/donbr/lila-graph/lila-mcp/pyproject.toml`
**Lines 1-77**

**Project Metadata**:
- Name: `lila-mcp-standalone`
- Version: `1.0.0`
- Python: `>=3.12`

**Core Dependencies** (Lines 10-32):
- `fastmcp>=2.12.3` - MCP framework
- `neo4j>=5.15.0` - Database connectivity
- `openai>=1.30.0`, `anthropic>=0.25.0` - LLM integration
- `pydantic>=2.6.0` - Data validation
- `python-dotenv>=1.0.0` - Configuration
- `click>=8.1.0` - CLI
- `claude-agent-sdk>=0.1.0` - Agent orchestration

**Dev Dependencies** (Lines 35-40):
- `pytest>=8.0.0`, `pytest-asyncio>=0.23.0` - Testing
- `black>=24.0.0`, `ruff>=0.3.0` - Code formatting

**Build Configuration** (Lines 64-76):
- Package includes: `*.py`, `agents/**/*.py`, `llm/**/*.py`, `graph/**/*.py`
- Build system: hatchling

#### `.mcp.json` - MCP Server Configuration
**Purpose**: FastMCP server metadata and capabilities declaration

#### `fastmcp.json` - FastMCP Configuration
**Purpose**: FastMCP-specific configuration

## Summary Statistics

### Public API Surface

**MCP Servers**: 2 implementations
- Production server with Neo4j backend
- Simplified server with mock data

**Resources**: 13 total
- 5 shared resources (personas, relationships, interactions)
- 8 additional resources in simplified server

**Tools**: 11 total
- 6 shared tools (relationship management, analysis, strategy)
- 5 additional tools in simplified server

**Prompts**: 3 total
- Attachment style assessment
- Emotional climate analysis
- Secure response generation

**Orchestrators**: 2 domain orchestrators
- Architecture analysis (5 phases)
- UX design workflow (6 phases)

**Agents**: 6 agent definitions
- 2 architecture agents (analyzer, doc-writer)
- 4 UX agents (researcher, IA architect, UI designer, prototype developer)

**CLI Tools**: 4 command-line utilities
- Data import/export (2)
- Server entry points (2)
- Test runners (2)

### Internal Implementation

**Package Modules**: 3 packages
- `ra_orchestrators/` - Orchestration framework
- `ra_agents/` - Agent registry and definitions
- `ra_tools/` - Tool integration layer

**Private Classes**: 7
- 2 MCP server implementations
- 2 data management utilities
- 3 framework components

**Total Lines of Code**: ~6,500 lines
- MCP servers: ~1,600 lines
- Data management: ~700 lines
- Orchestrators: ~1,200 lines
- Agents/Tools: ~350 lines
- Tests: ~380 lines

### Entry Points Summary

**Direct Entry Points**: 8
- 2 MCP server entry points
- 2 data management CLI tools
- 2 orchestrator runners
- 2 test runners

**Module-Level Exports**: 2
- FastMCP discovery via `mcp` variable

**Service Endpoints**: 2
- MCP server (port 8765)
- FastMCP Inspector (port 6274)
- Health check HTTP endpoint

---

**Analysis Date**: October 3, 2025
**Repository**: /home/donbr/lila-graph/lila-mcp
**Total Python Files**: 16
**Public Modules**: 16
**Internal Modules**: 3
**Configuration Files**: 4
