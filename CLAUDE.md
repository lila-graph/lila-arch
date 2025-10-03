# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**lila-arch** is a portable Repository Analyzer Framework for multi-domain AI agent orchestration. It provides a drop-in analysis toolkit with specialized agents for architecture analysis, UX/UI design, and future domains (DevOps, Testing).

**Key Principles:**
- Portable: Drop into any repository with `ra_` prefix to avoid collisions
- Timestamped outputs: Each run creates `ra_output/{domain}_{YYYYMMDD_HHMMSS}/`
- Extensible: Add new domain orchestrators in <1 day via `BaseOrchestrator`
- Agent-based: JSON-defined reusable agents in `ra_agents/{domain}/`

## Development Commands

### Environment Setup

```bash
# Python 3.11+ required (check: python3 --version)
# Install dependencies (uses uv if available, otherwise pip)
uv pip install -e .
# OR
pip install -e .
```

### Running Orchestrators

```bash
# Architecture analysis → ra_output/architecture_{timestamp}/
python -m ra_orchestrators.architecture_orchestrator

# UX design workflow → ra_output/ux_{timestamp}/
python -m ra_orchestrators.ux_orchestrator "Project Name"

# With timeout for long-running analyses
timeout 1800 python -m ra_orchestrators.architecture_orchestrator
```

### Agent Management

```bash
# List all available agents
python -c "from ra_agents.registry import AgentRegistry; print(AgentRegistry().discover_agents())"

# List agents for specific domain
python -c "from ra_agents.registry import AgentRegistry; print(AgentRegistry().discover_agents(domain='ux'))"
```

### Testing & Development

```bash
# Run from repository root (important for module imports)
cd /home/donbr/lila-graph/lila-arch
python -m ra_orchestrators.architecture_orchestrator

# Quick test of base orchestrator
python -c "from ra_orchestrators.base_orchestrator import BaseOrchestrator; print('Import successful')"
```

## Architecture

### Core Framework Pattern

**Three-layer architecture:**

1. **Orchestrators** (`ra_orchestrators/`) - Domain-specific workflows
   - `base_orchestrator.py` - Core framework with phase execution, cost tracking, progress visualization
   - `architecture_orchestrator.py` - Repository analysis orchestrator
   - `ux_orchestrator.py` - UX/UI design orchestrator
   - Each orchestrator inherits from `BaseOrchestrator` and implements:
     - `get_agent_definitions()` - Define specialized agents
     - `get_allowed_tools()` - Specify available tools
     - `run()` - Implement multi-phase workflow

2. **Agents** (`ra_agents/`) - JSON-defined reusable specialists
   - `registry.py` - Agent discovery and loading with caching
   - Agent JSON schema:
     ```json
     {
       "name": "agent_name",
       "description": "What this agent does",
       "prompt": "Detailed system prompt...",
       "tools": ["Read", "Write", "Grep", "Glob"],
       "model": "sonnet",
       "domain": "architecture|ux|custom",
       "version": "1.0.0"
     }
     ```
   - Current agents:
     - `architecture/analyzer` - Code analysis
     - `architecture/doc_writer` - Documentation (shared across domains)
     - `ux/ux_researcher` - User research and personas
     - `ux/ia_architect` - Information architecture
     - `ux/ui_designer` - Visual design specs
     - `ux/prototype_developer` - Interactive prototyping

3. **Tools** (`ra_tools/`) - External service integrations
   - `mcp_registry.py` - MCP server discovery and management
   - `figma_integration.py` - Figma MCP + REST API wrapper

### BaseOrchestrator Contract

All orchestrators implement this pattern (see `base_orchestrator.py:30-343`):

```python
class CustomOrchestrator(BaseOrchestrator):
    def __init__(self):
        super().__init__(
            domain_name="custom",
            output_base_dir=Path("ra_output"),
            use_timestamp=True  # Creates ra_output/custom_{timestamp}/
        )

        # Define subdirectories
        self.phase1_dir = self.output_dir / "01_analysis"
        self.create_output_structure()

    def get_agent_definitions(self) -> Dict[str, AgentDefinition]:
        # Return inline or registry-loaded agents
        return {"analyzer": AgentDefinition(...)}

    def get_allowed_tools(self) -> List[str]:
        return ["Read", "Write", "Grep", "Glob", "Bash"]

    async def run(self):
        await self.execute_phase("phase1", "analyzer", "Task description", self.client)
```

### Key Architectural Decisions

1. **Timestamped Outputs** - Prevents overwriting, enables comparison across runs
   - Format: `ra_output/{domain}_{YYYYMMDD_HHMMSS}/`
   - Disable with `use_timestamp=False` for legacy workflows

2. **Agent Registry Pattern** - JSON-based agent definitions enable:
   - Cross-domain agent sharing (e.g., `doc_writer` used in both architecture and UX)
   - Version tracking per agent
   - Lazy loading with caching (`registry.py:55-80`)

3. **Phase Execution** - `BaseOrchestrator.execute_phase()` provides:
   - Progress tracking with tool visibility
   - Cost monitoring per phase
   - Output verification
   - Error handling with partial result preservation

4. **Tool Visibility** - `show_tool_details=True` displays all Read/Write/Grep operations for transparency

## MCP Server Configuration

This repo includes `.mcp.json` with:
- `mcp-server-time` - Time/timezone utilities
- `sequential-thinking` - Advanced reasoning
- `ai-docs-server` - MCP/FastMCP/LangGraph/Anthropic docs
- `ai-docs-server-full` - Full documentation variants

Additional recommended MCP servers:
- Figma MCP - Design context (configure in `ra_tools/figma_integration.py`)
- Playwright - Browser automation for prototyping

## Adding a New Domain Orchestrator

Target: <1 day implementation

### Step 1: Create Orchestrator

```python
# ra_orchestrators/devops_orchestrator.py
from .base_orchestrator import BaseOrchestrator

class DevOpsOrchestrator(BaseOrchestrator):
    def __init__(self):
        super().__init__(domain_name="devops")
        self.infra_dir = self.output_dir / "01_infrastructure"
        self.cicd_dir = self.output_dir / "02_cicd"
        self.create_output_structure()

    # Implement get_agent_definitions(), get_allowed_tools(), run()
```

### Step 2: Define Agents

```bash
mkdir -p ra_agents/devops
# Create analyzer.json, iac_generator.json, etc.
```

### Step 3: Run

```bash
python -m ra_orchestrators.devops_orchestrator
# Output: ra_output/devops_{timestamp}/
```

## Output Structure

### Architecture Analysis
```
ra_output/architecture_{timestamp}/
├── docs/
│   ├── 01_component_inventory.md
│   ├── 03_data_flows.md
│   └── 04_api_reference.md
├── diagrams/
│   └── 02_architecture_diagrams.md
└── README.md
```

### UX Design
```
ra_output/ux_{timestamp}/
├── 01_research/user_research.md
├── 02_ia/information_architecture.md
├── 03_design/visual_design.md
├── 04_prototypes/interactive_prototypes.md
├── 05_api_contracts/api_specifications.md
└── 06_design_system/design_system.md
```

## Common Patterns

### Loading Agents from Registry

```python
from ra_agents.registry import AgentRegistry

registry = AgentRegistry()
agent = registry.load_agent("doc_writer", domain="architecture")
# Agents are cached after first load
```

### Cross-Domain Agent Reuse

```python
# UX orchestrator using architecture domain's doc_writer
doc_writer = registry.load_agent("doc_writer", domain="architecture")
```

### Programmatic Orchestrator Usage

```python
from ra_orchestrators.architecture_orchestrator import ArchitectureOrchestrator

orchestrator = ArchitectureOrchestrator(
    output_base_dir=Path("custom_output"),
    show_tool_details=True,
    use_timestamp=False  # Use fixed directory name
)
await orchestrator.run_with_client()
```

## Important Constraints

1. **Always run from repo root** - Module imports require `python -m ra_orchestrators.{name}`
2. **Agent write mandate** - All agent prompts must include: "IMPORTANT: When asked to write to a file, ALWAYS use the Write tool to create the actual file. Do not just describe what you would write."
3. **Phase independence** - Each phase should be self-contained when possible for error recovery
4. **Output verification** - Always verify expected files were created using `verify_output()`

## Troubleshooting

**Import errors:**
```bash
# Ensure running from repo root
cd /home/donbr/lila-graph/lila-arch
python -m ra_orchestrators.architecture_orchestrator
```

**Agent not writing files:**
Check agent prompt includes write mandate (see Constraints #2 above)

**Timestamp collisions:**
```python
import time
orchestrator1 = ArchitectureOrchestrator()
time.sleep(2)  # Ensure different timestamp
orchestrator2 = ArchitectureOrchestrator()
```

**MCP tools unavailable:**
Check `.mcp.json` and Claude Code MCP server settings

## Distribution Model

This framework is designed to be:
1. **Cloned/submoduled** into target repositories
2. **Added to .gitignore** via `ra_*` prefix
3. **Run from target repo root** for analysis
4. **Committed selectively** (outputs optional, framework itself typically excluded)

## Related Documentation

- `README.md` - User-facing installation and usage guide
- `ra_orchestrators/claude-agents-research.md` - Comprehensive agent pattern research (832 lines)
- `ra_orchestrators/base_orchestrator.py:1-343` - Core framework implementation
