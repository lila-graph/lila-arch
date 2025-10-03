# API Reference

## Overview

This document provides comprehensive API reference documentation for the Lila MCP codebase, covering all public classes, functions, parameters, return types, and usage examples. The system consists of:

- **MCP Servers**: FastMCP-based servers providing psychological intelligence services
- **Orchestrators**: Multi-phase workflow coordinators for domain-specific tasks
- **Registries**: Component discovery and management systems
- **Integrations**: External tool and service wrappers
- **Data Management**: Neo4j import/export utilities

**Project**: Lila MCP - Multi-domain agent orchestration with psychological relationship modeling
**Last Updated**: 2025-10-03

---

## MCP Servers

### LilaMCPServer

**File**: `/home/donbr/lila-graph/lila-mcp/lila_mcp_server.py:29-779`

**Purpose**: Production MCP server exposing Lila's psychological relationship data and tools with Neo4j backend integration.

#### Class Definition

```python
class LilaMCPServer:
    """MCP server exposing Lila's psychological relationship data and tools with Neo4j integration."""
```

#### Initialization

```python
def __init__(self)
```

**Description**: Initialize the MCP server with Neo4j database connection.

**Parameters**: None

**Raises**:
- Connection errors logged but server continues with `self.driver = None` fallback

**Example**:
```python
from lila_mcp_server import LilaMCPServer
import asyncio

server = LilaMCPServer()
# Server initializes with:
# 1. FastMCP app creation
# 2. Neo4j connection setup
# 3. Resource registration
# 4. Tool registration
# 5. Prompt registration
# 6. Health check endpoint

asyncio.run(server.run_server())
```

**Environment Variables** (lines 49-51):
- `NEO4J_URI`: Default `bolt://localhost:7687`
- `NEO4J_USER`: Default `neo4j`
- `NEO4J_PASSWORD`: Required

**Internal Setup**:
- `_setup_database()` (line 46): Initializes Neo4j connection with error handling
- `_register_resources()` (line 41): Registers data endpoints
- `_register_tools()` (line 42): Registers action endpoints
- `_register_prompts()` (line 43): Registers AI prompt templates
- `_register_health_check()` (line 44): Registers HTTP health endpoint

#### Methods

##### close()

```python
def close(self)
```

**File**: `lila_mcp_server.py:64-67`

**Description**: Close Neo4j connection gracefully.

**Parameters**: None

**Returns**: None

**Example**:
```python
server = LilaMCPServer()
try:
    # Use server...
    pass
finally:
    server.close()
```

##### run_server()

```python
async def run_server(self, host: str = "localhost", port: int = 8765)
```

**File**: `lila_mcp_server.py:756-762`

**Description**: Run the MCP server with SSE transport.

**Parameters**:
- `host` (str): Server host address (default: "localhost")
- `port` (int): Server port number (default: 8765)

**Returns**: None

**Raises**: Logs errors and ensures cleanup via `finally` block

**Example**:
```python
import asyncio

async def main():
    server = LilaMCPServer()
    await server.run_server(host="0.0.0.0", port=8765)

asyncio.run(main())
```

#### Resources (Neo4j-backed)

##### neo4j://personas/all

**File**: `lila_mcp_server.py:72-114`

**Description**: Retrieve all personas with their psychological profiles.

**Returns**: JSON string with persona list

**Response Schema**:
```json
{
  "personas": [
    {
      "id": "string",
      "name": "string",
      "age": "integer",
      "role": "string",
      "attachment_style": "string",
      "personality": {
        "openness": "float (0-1)",
        "conscientiousness": "float (0-1)",
        "extraversion": "float (0-1)",
        "agreeableness": "float (0-1)",
        "neuroticism": "float (0-1)"
      }
    }
  ],
  "count": "integer",
  "last_updated": "ISO 8601 timestamp"
}
```

##### neo4j://personas/{persona_id}

**File**: `lila_mcp_server.py:116-161`

**Description**: Retrieve specific persona by ID with full psychological profile.

**Parameters**:
- `persona_id` (str): Unique persona identifier

**Returns**: JSON string with persona details

**Response Schema**: Includes additional fields beyond `/all`:
- `description`: Text description
- `trust_level`: Float (0-1)
- `communication_style`: String

##### neo4j://relationships/all

**File**: `lila_mcp_server.py:163-206`

**Description**: Retrieve all relationship data with psychological metrics.

**Returns**: JSON string with relationship list

**Response Schema**:
```json
{
  "relationships": [
    {
      "persona1_id": "string",
      "persona1_name": "string",
      "persona2_id": "string",
      "persona2_name": "string",
      "trust_level": "float (0-10)",
      "intimacy_level": "float (0-10)",
      "relationship_strength": "float (0-10)",
      "interaction_count": "integer",
      "relationship_type": "string",
      "emotional_valence": "float"
    }
  ],
  "count": "integer",
  "last_updated": "ISO 8601 timestamp"
}
```

##### neo4j://relationships/{persona1_id}/{persona2_id}

**File**: `lila_mcp_server.py:208-253`

**Description**: Get specific relationship metrics between two personas.

**Parameters**:
- `persona1_id` (str): First persona ID
- `persona2_id` (str): Second persona ID

**Returns**: JSON with relationship details including timestamps

##### neo4j://interactions/recent/{count}

**File**: `lila_mcp_server.py:255-295`

**Description**: Get recent interactions with psychological analysis.

**Parameters**:
- `count` (str): Number of interactions (max 50, default "10")

**Returns**: JSON with interaction list

#### Tools (Async)

##### update_relationship_metrics()

```python
async def update_relationship_metrics(
    persona1_id: str,
    persona2_id: str,
    trust_delta: float = 0.0,
    intimacy_delta: float = 0.0,
    strength_delta: float = 0.0
) -> str
```

**File**: `lila_mcp_server.py:301-360`

**Description**: Update relationship metrics between two personas with bounds checking (0-10 scale).

**Parameters**:
- `persona1_id` (str): First persona identifier
- `persona2_id` (str): Second persona identifier
- `trust_delta` (float): Change in trust level (-10 to +10, default 0.0)
- `intimacy_delta` (float): Change in intimacy level (-10 to +10, default 0.0)
- `strength_delta` (float): Change in relationship strength (-10 to +10, default 0.0)

**Returns**: JSON string with updated metrics

**Example**:
```python
result = await update_relationship_metrics(
    persona1_id="lila",
    persona2_id="don",
    trust_delta=0.5,
    intimacy_delta=0.3,
    strength_delta=0.4
)
```

**Response Schema**:
```json
{
  "success": true,
  "updated_relationship": {
    "participants": ["persona1_id", "persona2_id"],
    "participant_names": ["Name1", "Name2"],
    "trust_level": "float (0-10)",
    "intimacy_level": "float (0-10)",
    "relationship_strength": "float (0-10)",
    "changes": {
      "trust_delta": "float",
      "intimacy_delta": "float",
      "strength_delta": "float"
    }
  }
}
```

##### record_interaction()

```python
async def record_interaction(
    sender_id: str,
    recipient_id: str,
    content: str,
    emotional_valence: float = 0.0,
    relationship_impact: float = 0.0
) -> str
```

**File**: `lila_mcp_server.py:363-399`

**Description**: Record interaction between personas with psychological analysis.

**Parameters**:
- `sender_id` (str): Sender persona ID
- `recipient_id` (str): Recipient persona ID
- `content` (str): Interaction content/message
- `emotional_valence` (float): Emotional tone (-1 to +1, default 0.0)
- `relationship_impact` (float): Impact on relationship (default 0.0)

**Returns**: JSON with interaction ID and confirmation

**Side Effects**:
- Updates `last_interaction` timestamp
- Increments `interaction_count`
- Updates `emotional_valence` (rolling average)

##### analyze_persona_compatibility()

```python
async def analyze_persona_compatibility(
    persona1_id: str,
    persona2_id: str,
    relationship_type: str = "romantic"
) -> str
```

**File**: `lila_mcp_server.py:402-461`

**Description**: Assess relationship potential based on attachment styles.

**Parameters**:
- `persona1_id` (str): First persona ID
- `persona2_id` (str): Second persona ID
- `relationship_type` (str): Type of relationship (default "romantic")

**Returns**: JSON with compatibility analysis

**Compatibility Matrix** (lines 424-431):
- `(secure, secure)`: High compatibility
- `(secure, anxious)`: Good compatibility
- `(secure, avoidant)`: Moderate compatibility
- `(anxious, anxious)`: Challenging
- `(anxious, avoidant)`: Difficult
- `(avoidant, avoidant)`: Low compatibility

##### autonomous_strategy_selection()

```python
async def autonomous_strategy_selection(
    persona_id: str,
    conversation_context: str = "",
    situation_assessment: str = "",
    active_goals: str = "",
    attachment_style: str = "secure"
) -> str
```

**File**: `lila_mcp_server.py:464-513`

**Description**: AI-driven strategy selection based on attachment theory and context.

**Parameters**:
- `persona_id` (str): Persona identifier
- `conversation_context` (str): Current conversation context
- `situation_assessment` (str): Situation description
- `active_goals` (str): Active relationship goals
- `attachment_style` (str): Attachment style (default "secure")

**Returns**: JSON with selected strategy and reasoning

**Strategy Mappings** (lines 474-479):
- `secure`: emotional_bonding, vulnerable_disclosure, supportive_listening, trust_building
- `anxious`: reassurance_seeking, emotional_validation, secure_bonding, safety_creation
- `avoidant`: autonomous_connection, thoughtful_presence, respectful_distance, gradual_opening
- `exploratory`: growth_oriented_support, playful_engagement, curious_exploration, authentic_expression

##### assess_goal_progress()

```python
async def assess_goal_progress(
    persona_id: str,
    goals: str = "",
    recent_interactions: str = ""
) -> str
```

**File**: `lila_mcp_server.py:516-552`

**Description**: Assess progress toward relationship goals.

**Parameters**:
- `persona_id` (str): Persona identifier
- `goals` (str): Comma-separated goal list
- `recent_interactions` (str): Recent interaction data

**Returns**: JSON with goal progress assessment

##### generate_contextual_response()

```python
async def generate_contextual_response(
    persona_id: str,
    context: str,
    goals: str = "",
    constraints: str = ""
) -> str
```

**File**: `lila_mcp_server.py:555-608`

**Description**: Generate psychologically authentic response for persona.

**Parameters**:
- `persona_id` (str): Persona identifier
- `context` (str): Conversation context
- `goals` (str): Active goals (optional)
- `constraints` (str): Response constraints (optional)

**Returns**: JSON with generated response and psychological rationale

**Response Examples by Attachment Style**:
- `secure`: "I appreciate you sharing that with me. How can we work together on this?"
- `anxious`: "Thank you for telling me this. I want to make sure I understand how you're feeling."
- `avoidant`: "I hear what you're saying. Let me think about that for a moment."

#### Prompts

##### assess_attachment_style()

```python
def assess_attachment_style(
    persona_id: str,
    observation_period: str = "recent",
    behavioral_examples: str = ""
) -> str
```

**File**: `lila_mcp_server.py:614-643`

**Description**: Determine persona's attachment style from behavioral observations.

**Parameters**:
- `persona_id` (str): Persona to assess
- `observation_period` (str): Timeframe for observations (default "recent")
- `behavioral_examples` (str): Specific behavioral evidence

**Returns**: Formatted prompt string for LLM

**Analysis Framework**:
1. Emotional Regulation
2. Intimacy Comfort
3. Relationship Patterns
4. Communication Style
5. Response to Partner Distress

**Attachment Styles Considered**:
- Secure: Comfortable with intimacy and autonomy
- Anxious: Seeks closeness but fears abandonment
- Avoidant: Values independence over closeness
- Exploratory: Seeks authentic expression and growth

##### analyze_emotional_climate()

```python
def analyze_emotional_climate(
    conversation_text: str = "",
    interaction_id: str = "",
    participants: str = ""
) -> str
```

**File**: `lila_mcp_server.py:646-694`

**Description**: Evaluate conversation emotional dynamics and safety levels.

**Parameters**:
- `conversation_text` (str): Conversation to analyze
- `interaction_id` (str): Interaction identifier
- `participants` (str): Participant names/IDs

**Returns**: Formatted prompt string for LLM

**Assessment Dimensions** (1-10 scale):
1. **Safety Level**: Psychological safety for vulnerability
2. **Emotional Attunement**: Recognition of emotional needs
3. **Communication Quality**: Active listening indicators
4. **Power Dynamics**: Balance vs. imbalance
5. **Attachment Activation**: Security-building vs. threat responses

##### generate_secure_response()

```python
def generate_secure_response(
    scenario_description: str,
    personas: str,
    insecurity_triggers: str = "",
    growth_goals: str = ""
) -> str
```

**File**: `lila_mcp_server.py:697-739`

**Description**: Create attachment-security-building responses.

**Parameters**:
- `scenario_description` (str): Situation description
- `personas` (str): Involved personas
- `insecurity_triggers` (str): Known triggers (optional)
- `growth_goals` (str): Development goals (optional)

**Returns**: Formatted prompt string for LLM

**Secure Response Framework**:
1. **Emotional Safety First**: Validate emotions, create space for vulnerability
2. **Attunement and Understanding**: Reflect what you hear, ask curious questions
3. **Secure Base Behaviors**: Consistent responses, balance support with autonomy
4. **Repair and Growth**: Take responsibility, focus on connection

#### Health Check

**Endpoint**: `GET /health`

**File**: `lila_mcp_server.py:746-754`

**Description**: HTTP health check endpoint for container orchestration.

**Returns**: JSON with health status

**Response Schema**:
```json
{
  "status": "healthy",
  "service": "lila-mcp-server",
  "neo4j_connected": "boolean"
}
```

**Status Code**: 200 OK

**Usage**: Load balancers, Kubernetes probes, monitoring systems

#### Module-Level Exports

**File**: `lila_mcp_server.py:765-766`

```python
_server_instance = LilaMCPServer()
mcp = _server_instance.app  # FastMCP discovery
```

**Purpose**: Enable FastMCP CLI discovery via `fastmcp dev lila_mcp_server.py`

---

### SimpleLilaMCPServer

**File**: `/home/donbr/lila-graph/lila-mcp/simple_lila_mcp_server.py:33-830`

**Purpose**: Simplified MCP server with in-memory mock data for testing and development. Provides same interface as LilaMCPServer without requiring Neo4j.

#### Class Definition

```python
class SimpleLilaMCPServer:
    """MCP server with mock data for development and testing."""
```

#### Initialization

```python
def __init__(self)
```

**Description**: Initialize MCP server with mock data.

**Parameters**: None

**Mock Data** (lines 42-92):
- `self.mock_personas`: Dictionary of persona objects
- `self.mock_relationships`: Dictionary of relationship metrics
- `self.mock_interactions`: List of interaction records

**Default Personas**:
1. **Lila**: Age 28, Psychological Intelligence Agent, secure attachment
2. **Don**: Age 45, Software Developer, anxious attachment

**Example**:
```python
from simple_lila_mcp_server import SimpleLilaMCPServer

server = SimpleLilaMCPServer()
# Server ready with mock data, no Neo4j required
```

#### Key Differences from LilaMCPServer

1. **Data Storage**: In-memory dictionaries vs. Neo4j database
2. **Additional Resources** (lines 270-358):
   - `neo4j://emotional_climate/current`: Computed emotional safety metrics
   - `neo4j://attachment_styles/analysis`: Attachment compatibility analysis
   - `neo4j://goals/active`: Active relationship goals
   - `neo4j://psychological_insights/trends`: Psychological trends over time

3. **Additional Tools** (lines 609-631):
   - `commit_relationship_state()`: Explicit state persistence
   - `finalize_demo_session()`: Finalize all states at demo end

4. **Fallback Behavior**: No database errors, always returns data

#### Module-Level Export

**File**: `simple_lila_mcp_server.py:825`

```python
mcp = SimpleLilaMCPServer().app
```

**Usage**:
```bash
fastmcp dev simple_lila_mcp_server.py
```

---

## Orchestrators

### BaseOrchestrator

**File**: `/home/donbr/lila-graph/lila-mcp/orchestrators/base_orchestrator.py:26-296`

**Purpose**: Abstract base class providing common functionality for all domain-specific orchestrators.

#### Class Definition

```python
class BaseOrchestrator(ABC):
    """Base class for domain-specific orchestrators with common workflow patterns."""
```

#### Initialization

```python
def __init__(
    self,
    domain_name: str,
    output_base_dir: Path = Path("outputs"),
    show_tool_details: bool = True
)
```

**File**: `base_orchestrator.py:39-61`

**Description**: Initialize base orchestrator with domain configuration.

**Parameters**:
- `domain_name` (str): Name of the domain (e.g., 'architecture', 'ux', 'devops')
- `output_base_dir` (Path): Base directory for all outputs (default: `Path("outputs")`)
- `show_tool_details` (bool): Whether to display detailed tool usage (default: `True`)

**Instance Variables**:
- `self.domain_name` (str): Domain identifier
- `self.output_base_dir` (Path): Base output directory
- `self.output_dir` (Path): Domain-specific output directory
- `self.show_tool_details` (bool): Tool visibility toggle
- `self.total_cost` (float): Cumulative API cost
- `self.phase_costs` (Dict[str, float]): Per-phase cost tracking
- `self.completed_phases` (List[str]): Completed phase names

**Example**:
```python
from orchestrators.base_orchestrator import BaseOrchestrator

class CustomOrchestrator(BaseOrchestrator):
    def __init__(self):
        super().__init__(
            domain_name="custom",
            output_base_dir=Path("outputs"),
            show_tool_details=True
        )
```

#### Core Methods

##### create_output_structure()

```python
def create_output_structure(self, subdirs: Optional[List[str]] = None)
```

**File**: `base_orchestrator.py:62-72`

**Description**: Create output directory structure.

**Parameters**:
- `subdirs` (Optional[List[str]]): List of subdirectory names to create

**Returns**: None

**Side Effects**: Creates directories on filesystem

**Example**:
```python
orchestrator.create_output_structure(
    subdirs=["docs", "diagrams", "reports"]
)
```

##### display_message()

```python
def display_message(self, msg, show_tools: bool = True)
```

**File**: `base_orchestrator.py:74-115`

**Description**: Display message content with full visibility into tool usage.

**Parameters**:
- `msg`: Message to display (AssistantMessage, UserMessage, or ResultMessage)
- `show_tools` (bool): Whether to show tool usage details (default: True)

**Returns**: None

**Side Effects**: Prints to stdout

**Message Handling**:
- **AssistantMessage** (lines 81-100): Displays TextBlock and ToolUseBlock
- **UserMessage** (lines 102-107): Displays ToolResultBlock (truncated to 200 chars)
- **ResultMessage** (lines 109-115): Displays completion banner and cost

**Tool-Specific Display**:
- Read/Grep/Glob: Shows `file_path` or `pattern`
- Write: Shows `file_path` with ✍️ emoji
- Bash: Shows `command`

##### display_phase_header()

```python
def display_phase_header(
    self,
    phase_number: int,
    phase_name: str,
    emoji: str = "📋"
)
```

**File**: `base_orchestrator.py:117-127`

**Description**: Display formatted phase header.

**Parameters**:
- `phase_number` (int): Phase number (1-indexed)
- `phase_name` (str): Name of the phase
- `emoji` (str): Emoji to display (default: "📋")

**Returns**: None

**Example Output**:
```
======================================================================
📋 PHASE 1: Component Inventory
======================================================================
```

##### track_phase_cost()

```python
def track_phase_cost(self, phase_name: str, cost: float)
```

**File**: `base_orchestrator.py:129-137`

**Description**: Track cost for a specific phase.

**Parameters**:
- `phase_name` (str): Name of the phase
- `cost` (float): Cost in USD

**Returns**: None

**Side Effects**: Updates `self.phase_costs` and `self.total_cost`

##### mark_phase_complete()

```python
def mark_phase_complete(self, phase_name: str)
```

**File**: `base_orchestrator.py:139-145`

**Description**: Mark a phase as completed.

**Parameters**:
- `phase_name` (str): Name of the completed phase

**Returns**: None

**Side Effects**: Appends to `self.completed_phases`

##### verify_outputs()

```python
async def verify_outputs(self, expected_files: List[Path]) -> bool
```

**File**: `base_orchestrator.py:147-170`

**Description**: Verify all expected outputs were created.

**Parameters**:
- `expected_files` (List[Path]): List of expected file paths

**Returns**:
- `True` if all files exist
- `False` if any files missing

**Side Effects**: Prints verification results to stdout

**Example**:
```python
expected = [
    Path("docs/01_component_inventory.md"),
    Path("docs/02_architecture_diagrams.md")
]
all_created = await orchestrator.verify_outputs(expected)
```

##### display_summary()

```python
def display_summary(self)
```

**File**: `base_orchestrator.py:172-187`

**Description**: Display orchestrator run summary with costs and phase information.

**Parameters**: None

**Returns**: None

**Displays**:
- Domain name
- Output directory (absolute path)
- Number of completed phases
- Total cost in USD
- Cost breakdown per phase

##### execute_phase()

```python
async def execute_phase(
    self,
    phase_name: str,
    agent_name: str,
    prompt: str,
    client: ClaudeSDKClient
)
```

**File**: `base_orchestrator.py:216-240`

**Description**: Execute a single phase of the workflow.

**Parameters**:
- `phase_name` (str): Name of the phase
- `agent_name` (str): Name of the agent to use
- `prompt` (str): Prompt for the agent
- `client` (ClaudeSDKClient): Claude SDK client instance

**Returns**: None

**Side Effects**:
- Sends query to Claude API
- Displays streaming messages
- Tracks phase cost
- Marks phase complete

**Example**:
```python
await self.execute_phase(
    phase_name="Component Inventory",
    agent_name="analyzer",
    prompt="Analyze the codebase...",
    client=self.client
)
```

##### create_client_options()

```python
def create_client_options(
    self,
    permission_mode: str = "acceptEdits",
    cwd: str = "."
) -> ClaudeAgentOptions
```

**File**: `base_orchestrator.py:242-264`

**Description**: Create Claude SDK client options.

**Parameters**:
- `permission_mode` (str): Permission mode ("acceptEdits" or "ask", default: "acceptEdits")
- `cwd` (str): Current working directory (default: ".")

**Returns**: `ClaudeAgentOptions` configured for this orchestrator

**Permission Modes**:
- `"acceptEdits"`: Auto-approve Read/Write/Grep/Glob, prompt for Bash
- `"ask"`: Prompt for every tool use

**Example**:
```python
options = orchestrator.create_client_options(
    permission_mode="acceptEdits",
    cwd="/path/to/project"
)
```

##### run_with_client()

```python
async def run_with_client(self)
```

**File**: `base_orchestrator.py:266-295`

**Description**: Run orchestrator with automatic client setup and teardown.

**Parameters**: None

**Returns**: `True` on success

**Raises**: Re-raises exceptions after cleanup

**Workflow**:
1. Display orchestrator header
2. Create client options
3. Open async context with ClaudeSDKClient
4. Store client reference
5. Call subclass `run()` method
6. Display summary
7. Clean up client reference (in finally block)

**Example**:
```python
orchestrator = ArchitectureOrchestrator()
await orchestrator.run_with_client()
```

#### Abstract Methods

**Subclasses must implement**:

##### get_agent_definitions()

```python
@abstractmethod
def get_agent_definitions(self) -> Dict[str, AgentDefinition]
```

**File**: `base_orchestrator.py:189-196`

**Description**: Get agent definitions for this orchestrator.

**Returns**: Dictionary mapping agent names to AgentDefinition objects

**Example Implementation**:
```python
def get_agent_definitions(self) -> Dict[str, AgentDefinition]:
    return {
        "analyzer": AgentDefinition(
            description="Code analyzer",
            prompt="Analyze code structure...",
            tools=["Read", "Grep", "Glob", "Write"],
            model="sonnet"
        )
    }
```

##### get_allowed_tools()

```python
@abstractmethod
def get_allowed_tools(self) -> List[str]
```

**File**: `base_orchestrator.py:198-205`

**Description**: Get list of allowed tools for this orchestrator.

**Returns**: List of tool names

**Example Implementation**:
```python
def get_allowed_tools(self) -> List[str]:
    return ["Read", "Write", "Grep", "Glob", "Bash"]
```

##### run()

```python
@abstractmethod
async def run(self)
```

**File**: `base_orchestrator.py:207-214`

**Description**: Execute the orchestrator workflow.

**Returns**: None

**Example Implementation**:
```python
async def run(self):
    await self.phase_1_analysis()
    await self.phase_2_documentation()
    await self.phase_3_synthesis()
```

---

### ArchitectureOrchestrator

**File**: `/home/donbr/lila-graph/lila-mcp/orchestrators/architecture_orchestrator.py:20-313`

**Purpose**: Orchestrator for comprehensive repository architecture analysis.

#### Class Definition

```python
class ArchitectureOrchestrator(BaseOrchestrator):
    """Orchestrator for comprehensive repository architecture analysis."""
```

**Inherits**: `BaseOrchestrator`

#### Initialization

```python
def __init__(
    self,
    output_base_dir: Path = Path("repo_analysis"),
    show_tool_details: bool = True
)
```

**File**: `architecture_orchestrator.py:23-54`

**Description**: Initialize architecture orchestrator.

**Parameters**:
- `output_base_dir` (Path): Base directory for analysis outputs (default: `Path("repo_analysis")`)
- `show_tool_details` (bool): Whether to display detailed tool usage (default: `True`)

**Output Structure**:
- `self.docs_dir`: Documentation directory
- `self.diagrams_dir`: Architecture diagrams directory
- `self.reports_dir`: Analysis reports directory

**Example**:
```python
from orchestrators.architecture_orchestrator import ArchitectureOrchestrator

orchestrator = ArchitectureOrchestrator(
    output_base_dir=Path("repo_analysis"),
    show_tool_details=True
)
```

#### Agent Definitions

**File**: `architecture_orchestrator.py:56-101`

##### analyzer

**Description**: Analyzes code structure, patterns, and architecture

**Prompt**: Code analyzer expert focusing on systematic examination

**Tools**: `["Read", "Grep", "Glob", "Write", "Bash"]`

**Model**: `sonnet`

##### doc-writer

**Description**: Writes comprehensive technical documentation

**Prompt**: Technical documentation expert focusing on clarity

**Tools**: `["Read", "Write", "Grep", "Glob"]`

**Model**: `sonnet`

#### Allowed Tools

**File**: `architecture_orchestrator.py:103-109`

```python
["Read", "Write", "Grep", "Glob", "Bash"]
```

#### Workflow Phases

##### Phase 1: Component Inventory

**File**: `architecture_orchestrator.py:111-142`

**Method**: `async def phase_1_component_inventory(self)`

**Agent**: analyzer

**Output**: `{docs_dir}/01_component_inventory.md`

**Analyzes**:
1. All Python modules and their purposes
2. Key classes and functions with descriptions
3. Public API surface vs internal implementation
4. Entry points and main interfaces

##### Phase 2: Architecture Diagrams

**File**: `architecture_orchestrator.py:144-181`

**Method**: `async def phase_2_architecture_diagrams(self)`

**Agent**: analyzer

**Output**: `{diagrams_dir}/02_architecture_diagrams.md`

**Generates Mermaid diagrams**:
1. System architecture (layered view)
2. Component relationships
3. Class hierarchies
4. Module dependencies

##### Phase 3: Data Flows

**File**: `architecture_orchestrator.py:183-218`

**Method**: `async def phase_3_data_flows(self)`

**Agent**: analyzer

**Output**: `{docs_dir}/03_data_flows.md`

**Documents**:
1. Simple query flow
2. Interactive client session flow
3. Tool permission callback flow
4. MCP server communication flow
5. Message parsing and routing

##### Phase 4: API Documentation

**File**: `architecture_orchestrator.py:220-239`

**Method**: `async def phase_4_api_documentation(self)`

**Agent**: doc-writer

**Output**: `{docs_dir}/04_api_reference.md`

**Documents**:
1. All public functions and classes
2. Parameters, return types, and examples
3. Usage patterns and best practices
4. Configuration options

##### Phase 5: Final Synthesis

**File**: `architecture_orchestrator.py:241-281`

**Method**: `async def phase_5_synthesis(self)`

**Agent**: doc-writer

**Output**: `{output_dir}/README.md`

**Creates**:
- Overview and quick start
- Architecture summary
- Component overview
- Data flows
- References to detailed docs

#### Run Method

**File**: `architecture_orchestrator.py:283-301`

```python
async def run(self):
    """Run comprehensive repository analysis in phases."""
    await self.phase_1_component_inventory()
    await self.phase_2_architecture_diagrams()
    await self.phase_3_data_flows()
    await self.phase_4_api_documentation()
    await self.phase_5_synthesis()

    # Verify all outputs
    expected_files = [
        self.docs_dir / "01_component_inventory.md",
        self.diagrams_dir / "02_architecture_diagrams.md",
        self.docs_dir / "03_data_flows.md",
        self.docs_dir / "04_api_reference.md",
        self.output_dir / "README.md",
    ]
    await self.verify_outputs(expected_files)
```

#### Entry Point

**File**: `architecture_orchestrator.py:304-312`

```python
async def main():
    """Run comprehensive repository analysis."""
    orchestrator = ArchitectureOrchestrator()
    await orchestrator.run_with_client()

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

**Usage**:
```bash
python -m orchestrators.architecture_orchestrator
```

---

### UXOrchestrator

**File**: `/home/donbr/lila-graph/lila-mcp/orchestrators/ux_orchestrator.py:21-619`

**Purpose**: Orchestrator for comprehensive UX/UI design workflow.

#### Class Definition

```python
class UXOrchestrator(BaseOrchestrator):
    """Orchestrator for comprehensive UX/UI design workflow."""
```

**Inherits**: `BaseOrchestrator`

#### Initialization

```python
def __init__(
    self,
    project_name: str = "Project",
    output_base_dir: Path = Path("outputs"),
    show_tool_details: bool = True
)
```

**File**: `ux_orchestrator.py:24-54`

**Description**: Initialize UX orchestrator.

**Parameters**:
- `project_name` (str): Name of the project being designed (default: "Project")
- `output_base_dir` (Path): Base directory for design outputs (default: `Path("outputs")`)
- `show_tool_details` (bool): Whether to display detailed tool usage (default: `True`)

**Output Structure** (6-phase workflow):
- `self.research_dir`: `01_research/` - User research and personas
- `self.ia_dir`: `02_ia/` - Information architecture
- `self.design_dir`: `03_design/` - Visual design specs
- `self.prototypes_dir`: `04_prototypes/` - Interactive prototypes
- `self.api_contracts_dir`: `05_api_contracts/` - API specifications
- `self.design_system_dir`: `06_design_system/` - Design system docs

**Example**:
```python
from orchestrators.ux_orchestrator import UXOrchestrator

orchestrator = UXOrchestrator(
    project_name="MyApp",
    output_base_dir=Path("ux_outputs")
)
```

#### Agent Definitions

**File**: `ux_orchestrator.py:66-162`

##### ux_researcher

**Description**: Conducts user research, creates personas, analyzes user journeys

**Tools**: `["Read", "Write", "Grep", "Glob", "WebSearch"]`

**Model**: `sonnet`

##### ia_architect

**Description**: Designs information architecture, sitemaps, navigation structures

**Tools**: `["Read", "Write", "Grep", "Glob"]`

**Model**: `sonnet`

##### ui_designer

**Description**: Creates visual designs and high-fidelity mockups

**Tools**: `["Read", "Write", "Grep", "Glob"]`

**Model**: `sonnet`

**Special Note**: Uses Figma MCP tools when available, otherwise creates detailed design specs

##### prototype_developer

**Description**: Creates interactive prototypes and validates user flows

**Tools**: `["Read", "Write", "Grep", "Glob", "Bash"]`

**Model**: `sonnet`

#### Allowed Tools

**File**: `ux_orchestrator.py:157-163`

```python
["Read", "Write", "Grep", "Glob", "Bash", "WebSearch"]
```

#### Workflow Phases

The UX orchestrator implements a comprehensive 6-phase design workflow. Each phase builds on previous outputs.

**Example Usage**:
```python
import asyncio
from orchestrators.ux_orchestrator import UXOrchestrator

async def main():
    orchestrator = UXOrchestrator(project_name="MyApp")
    await orchestrator.run_with_client()

asyncio.run(main())
```

---

### CrossOrchestratorCommunication

**File**: `/home/donbr/lila-graph/lila-mcp/orchestrators/base_orchestrator.py:298-343`

**Purpose**: Mixin for orchestrators that need cross-domain communication.

#### Class Definition

```python
class CrossOrchestratorCommunication:
    """Mixin for orchestrators that need to communicate with each other."""
```

#### Initialization

```python
def __init__(self)
```

**File**: `base_orchestrator.py:301-303`

**Description**: Initialize cross-orchestrator communication.

**Instance Variables**:
- `self.orchestrator_registry` (Dict[str, BaseOrchestrator]): Registry of orchestrator instances

#### Methods

##### register_orchestrator()

```python
def register_orchestrator(self, name: str, orchestrator: BaseOrchestrator)
```

**File**: `base_orchestrator.py:305-312`

**Description**: Register an orchestrator for cross-domain communication.

**Parameters**:
- `name` (str): Name of the orchestrator
- `orchestrator` (BaseOrchestrator): Orchestrator instance

**Returns**: None

**Example**:
```python
ux_orch = UXOrchestrator()
arch_orch = ArchitectureOrchestrator()

ux_orch.register_orchestrator("architecture", arch_orch)
```

##### invoke_orchestrator()

```python
async def invoke_orchestrator(
    self,
    orchestrator_name: str,
    phase_name: str,
    context: Dict[str, Any]
) -> Any
```

**File**: `base_orchestrator.py:314-343`

**Description**: Invoke another orchestrator for cross-domain validation.

**Parameters**:
- `orchestrator_name` (str): Name of the orchestrator to invoke
- `phase_name` (str): Name of the phase to execute
- `context` (Dict[str, Any]): Context data to pass

**Returns**: Result from the orchestrator

**Raises**: `ValueError` if orchestrator not registered

**Example**:
```python
result = await ux_orch.invoke_orchestrator(
    orchestrator_name="architecture",
    phase_name="validate_design",
    context={"design_file": "designs/mockup.md"}
)
```

**Usage Pattern**: Mix in alongside BaseOrchestrator for cross-domain workflows:
```python
class CrossDomainOrchestrator(BaseOrchestrator, CrossOrchestratorCommunication):
    def __init__(self):
        BaseOrchestrator.__init__(self, domain_name="cross")
        CrossOrchestratorCommunication.__init__(self)
```

---

## Registries

### AgentRegistry

**File**: `/home/donbr/lila-graph/lila-mcp/agents/registry.py:10-100`

**Purpose**: Registry for discovering and loading agent definitions from JSON files.

#### Class Definition

```python
class AgentRegistry:
    """Registry for discovering and loading agent definitions from JSON files."""
```

#### Initialization

```python
def __init__(self, agents_dir: Path = Path(__file__).parent)
```

**File**: `registry.py:13-20`

**Description**: Initialize agent registry.

**Parameters**:
- `agents_dir` (Path): Base directory containing agent definitions (default: package directory)

**Instance Variables**:
- `self.agents_dir` (Path): Base directory for agent definitions
- `self._cache` (Dict[str, AgentDefinition]): Cache for loaded agents

**Directory Structure**:
```
agents/
├── __init__.py
├── registry.py
├── ux/
│   ├── ux_researcher.json
│   ├── ia_architect.json
│   └── ui_designer.json
└── architecture/
    ├── analyzer.json
    └── doc_writer.json
```

**Example**:
```python
from agents.registry import AgentRegistry

registry = AgentRegistry()
```

#### Methods

##### discover_agents()

```python
def discover_agents(self, domain: Optional[str] = None) -> Dict[str, str]
```

**File**: `registry.py:22-43`

**Description**: Discover all available agent definition files.

**Parameters**:
- `domain` (Optional[str]): Optional domain filter (e.g., 'ux', 'architecture')

**Returns**: Dictionary mapping agent names to file paths

**Behavior**:
- Scans subdirectories for `*.json` files
- Skips directories starting with `_`
- If domain specified, only scans that domain's directory

**Example**:
```python
# Discover all agents
all_agents = registry.discover_agents()
# {'ux_researcher': '/path/to/ux/ux_researcher.json', ...}

# Discover UX agents only
ux_agents = registry.discover_agents(domain='ux')
# {'ux_researcher': '/path/to/ux/ux_researcher.json', ...}
```

##### load_agent()

```python
def load_agent(
    self,
    agent_name: str,
    domain: Optional[str] = None
) -> Optional[AgentDefinition]
```

**File**: `registry.py:45-80`

**Description**: Load an agent definition from JSON file.

**Parameters**:
- `agent_name` (str): Name of the agent (without .json extension)
- `domain` (Optional[str]): Optional domain to search in

**Returns**:
- `AgentDefinition` object if found
- `None` if not found

**Caching**: Uses cache key `"{domain}/{agent_name}"` to avoid re-loading

**JSON Schema**:
```json
{
  "description": "Agent description",
  "prompt": "System prompt for agent",
  "tools": ["Read", "Write", "Grep"],
  "model": "sonnet"
}
```

**Example**:
```python
# Load specific agent
agent = registry.load_agent('ux_researcher', domain='ux')

# Use agent
if agent:
    print(f"Description: {agent.description}")
    print(f"Tools: {agent.tools}")
```

##### load_domain_agents()

```python
def load_domain_agents(self, domain: str) -> Dict[str, AgentDefinition]
```

**File**: `registry.py:82-99`

**Description**: Load all agents for a specific domain.

**Parameters**:
- `domain` (str): Domain name (e.g., 'ux', 'architecture')

**Returns**: Dictionary mapping agent names to AgentDefinition objects

**Example**:
```python
# Load all UX agents
ux_agents = registry.load_domain_agents('ux')

for name, agent in ux_agents.items():
    print(f"{name}: {agent.description}")
```

**Use Case**: Orchestrators use this to load all agents for their domain

---

### MCPRegistry

**File**: `/home/donbr/lila-graph/lila-mcp/tools/mcp_registry.py:8-153`

**Purpose**: Registry for discovering and managing MCP server connections.

#### Class Definition

```python
class MCPRegistry:
    """Registry for discovering and managing MCP server connections."""
```

#### Initialization

```python
def __init__(self)
```

**File**: `mcp_registry.py:11-14`

**Description**: Initialize MCP registry and discover servers.

**Instance Variables**:
- `self.available_servers` (Dict[str, Dict[str, Any]]): Server registry

**Auto-discovery**: Calls `discover_mcp_servers()` during initialization

**Example**:
```python
from tools.mcp_registry import MCPRegistry

registry = MCPRegistry()
```

#### Registered Servers

**File**: `mcp_registry.py:26-51`

##### figma

- **Available**: False (requires configuration)
- **Description**: Figma MCP Server for design context
- **Tools**: `["figma_get_file", "figma_get_components"]`
- **Config Required**: `FIGMA_ACCESS_TOKEN`

##### v0

- **Available**: False (requires configuration)
- **Description**: Vercel v0 MCP Server for UI generation
- **Tools**: `["v0_generate_ui", "v0_generate_from_image", "v0_chat_complete"]`
- **Config Required**: `V0_API_KEY`

##### sequential-thinking

- **Available**: True (assumed available)
- **Description**: Advanced reasoning MCP tool
- **Tools**: `["sequentialthinking"]`
- **Config Required**: None

##### playwright

- **Available**: False
- **Description**: Browser automation MCP tool
- **Tools**: `["browser_navigate", "browser_click", "browser_snapshot"]`
- **Config Required**: None

#### Methods

##### discover_mcp_servers()

```python
def discover_mcp_servers(self) -> Dict[str, Dict[str, Any]]
```

**File**: `mcp_registry.py:16-53`

**Description**: Auto-discover available MCP servers.

**Returns**: Dictionary of available MCP servers with their capabilities

**Server Schema**:
```python
{
  "server_name": {
    "available": "boolean",
    "description": "string",
    "tools": ["tool_name", ...],
    "config_required": "boolean"
  }
}
```

##### is_server_available()

```python
def is_server_available(self, server_name: str) -> bool
```

**File**: `mcp_registry.py:55-67`

**Description**: Check if an MCP server is available.

**Parameters**:
- `server_name` (str): Name of the MCP server

**Returns**: `True` if server is available

**Example**:
```python
if registry.is_server_available('figma'):
    print("Figma MCP server is available")
else:
    print("Figma MCP server not configured")
```

##### get_server_tools()

```python
def get_server_tools(self, server_name: str) -> List[str]
```

**File**: `mcp_registry.py:69-81`

**Description**: Get list of tools provided by an MCP server.

**Parameters**:
- `server_name` (str): Name of the MCP server

**Returns**: List of tool names (empty list if server not found)

**Example**:
```python
figma_tools = registry.get_server_tools('figma')
# ['figma_get_file', 'figma_get_components']
```

##### validate_tool_availability()

```python
def validate_tool_availability(self, tool_name: str) -> bool
```

**File**: `mcp_registry.py:83-96`

**Description**: Validate if a specific tool is available.

**Parameters**:
- `tool_name` (str): Name of the tool

**Returns**: `True` if tool is available from any server

**Example**:
```python
if registry.validate_tool_availability('figma_get_file'):
    # Use Figma tool
    pass
else:
    # Use fallback
    pass
```

##### get_configuration_requirements()

```python
def get_configuration_requirements(
    self,
    server_name: str
) -> Optional[Dict[str, Any]]
```

**File**: `mcp_registry.py:98-128`

**Description**: Get configuration requirements for an MCP server.

**Parameters**:
- `server_name` (str): Name of the MCP server

**Returns**:
- Configuration dictionary if server requires config
- `None` if no config required or server not found

**Response Schema**:
```python
{
  "required_env": ["ENV_VAR_NAME", ...],
  "optional_env": ["ENV_VAR_NAME", ...],
  "setup_instructions": "string"
}
```

**Example**:
```python
config = registry.get_configuration_requirements('figma')
# {
#   "required_env": ["FIGMA_ACCESS_TOKEN"],
#   "optional_env": ["FIGMA_FILE_ID"],
#   "setup_instructions": "Get access token from..."
# }
```

##### get_fallback_options()

```python
def get_fallback_options(self, tool_name: str) -> List[str]
```

**File**: `mcp_registry.py:130-152`

**Description**: Get fallback options if a tool is unavailable.

**Parameters**:
- `tool_name` (str): Name of the requested tool

**Returns**: List of alternative approaches

**Fallback Mappings**:
- `figma_get_file`:
  - "Create design specifications in markdown"
  - "Use Mermaid diagrams for wireframes"
  - "Document design manually with screenshots"
- `v0_generate_ui`:
  - "Write component specifications"
  - "Create HTML/CSS mockups"
  - "Use alternative design-to-code tools (Builder.io, Anima)"

**Example**:
```python
fallbacks = registry.get_fallback_options('figma_get_file')
for option in fallbacks:
    print(f"- {option}")
```

---

## Integrations

### FigmaIntegration

**File**: `/home/donbr/lila-graph/lila-mcp/tools/figma_integration.py:7-157`

**Purpose**: Wrapper for Figma MCP server and REST API integration.

#### Class Definition

```python
class FigmaIntegration:
    """Wrapper for Figma MCP server and REST API integration."""
```

#### Initialization

```python
def __init__(self)
```

**File**: `figma_integration.py:10-13`

**Description**: Initialize Figma integration.

**Environment Variables**:
- `FIGMA_ACCESS_TOKEN`: Figma personal access token (optional)

**Instance Variables**:
- `self.access_token` (str): Token from environment
- `self.mcp_available` (bool): MCP server availability status

**Example**:
```python
import os
from tools.figma_integration import FigmaIntegration

# Set token
os.environ['FIGMA_ACCESS_TOKEN'] = 'your_token_here'

# Initialize
figma = FigmaIntegration()
```

#### Methods

##### is_available()

```python
def is_available(self) -> bool
```

**File**: `figma_integration.py:15-21`

**Description**: Check if Figma integration is available.

**Returns**: `True` if token exists or MCP is available

**Example**:
```python
if figma.is_available():
    # Use Figma integration
    context = figma.get_design_context(file_id)
else:
    # Use manual design approach
    print(figma.get_setup_instructions())
```

##### get_design_context()

```python
def get_design_context(self, file_id: str) -> Optional[Dict[str, Any]]
```

**File**: `figma_integration.py:23-57`

**Description**: Get design context from Figma file.

**Parameters**:
- `file_id` (str): Figma file ID

**Returns**:
- Design context dictionary if available
- Error dictionary with fallback instructions if unavailable

**Response Schema (Error)**:
```python
{
  "error": "Figma integration not configured",
  "fallback": "Use manual design specifications",
  "instructions": [
    "Set FIGMA_ACCESS_TOKEN environment variable",
    "Or configure Figma MCP server",
    "Or create manual design docs"
  ]
}
```

**Response Schema (Stub)**:
```python
{
  "file_id": "string",
  "message": "Figma integration stub - implement actual API calls",
  "next_steps": [
    "Install Figma MCP server: npm install -g @figma/mcp-server",
    "Configure in Claude Code MCP settings",
    "Or use Figma REST API directly"
  ]
}
```

**Note**: Current implementation is a stub. Future implementation would:
1. Try Figma MCP server if available
2. Fall back to Figma REST API
3. Return design context (components, styles, etc.)

##### export_to_code()

```python
def export_to_code(
    self,
    component_id: str,
    framework: str = "react"
) -> Optional[str]
```

**File**: `figma_integration.py:59-84`

**Description**: Export Figma component to code.

**Parameters**:
- `component_id` (str): Figma component ID
- `framework` (str): Target framework (default: "react")

**Returns**:
- Generated code string if available
- `None` if unavailable

**Supported Frameworks**: react, vue, svelte, etc.

**Note**: Stub implementation. Future integration with:
- Figma MCP → Anima API
- Figma MCP → Builder.io
- Direct code generation

##### create_component()

```python
def create_component(self, spec: Dict[str, Any]) -> Optional[str]
```

**File**: `figma_integration.py:86-101`

**Description**: Create a Figma component from specification.

**Parameters**:
- `spec` (Dict[str, Any]): Component specification dictionary

**Returns**:
- Component ID if created
- `None` if MCP unavailable

**Note**: Stub implementation for future Figma API integration

##### get_setup_instructions()

```python
def get_setup_instructions(self) -> str
```

**File**: `figma_integration.py:103-156`

**Description**: Get setup instructions for Figma integration.

**Returns**: Markdown-formatted setup instructions

**Instructions Include**:

**Option 1: Figma MCP Server (Recommended)**
- Install command: `npm install -g @figma/mcp-server`
- Configuration for Claude Code
- Token setup from Figma Settings

**Option 2: Figma REST API**
- Environment variable setup
- API documentation link
- Base URL: `https://api.figma.com/v1`

**Option 3: Manual Design Specs**
- Markdown specifications
- Mermaid diagrams
- Component screenshots
- Design tokens documentation

**Example**:
```python
if not figma.is_available():
    print(figma.get_setup_instructions())
```

---

## Data Management

### Neo4jDataImporter

**File**: `/home/donbr/lila-graph/lila-mcp/import_data.py:22-465`

**Purpose**: Imports psychological intelligence data and schema into Neo4j for MCP standalone.

#### Class Definition

```python
class Neo4jDataImporter:
    """Imports psychological intelligence data and schema into Neo4j for MCP standalone."""
```

#### Initialization

```python
def __init__(
    self,
    uri: str,
    user: str,
    password: str,
    max_retries: int = 30
)
```

**File**: `import_data.py:25-32`

**Description**: Initialize Neo4j connection with retry logic.

**Parameters**:
- `uri` (str): Neo4j connection URI (e.g., "bolt://localhost:7687")
- `user` (str): Neo4j username
- `password` (str): Neo4j password
- `max_retries` (int): Maximum connection retry attempts (default: 30)

**Instance Variables**:
- `self.uri` (str): Connection URI
- `self.user` (str): Username
- `self.password` (str): Password
- `self.max_retries` (int): Retry limit
- `self.driver`: Neo4j driver instance

**Retry Logic**: Waits 2 seconds between retries for container startup

**Example**:
```python
from import_data import Neo4jDataImporter

importer = Neo4jDataImporter(
    uri="bolt://localhost:7687",
    user="neo4j",
    password="your_password",
    max_retries=30
)
```

#### Methods

##### close()

```python
def close(self)
```

**File**: `import_data.py:51-54`

**Description**: Close Neo4j connection.

**Parameters**: None

**Returns**: None

##### clear_database()

```python
def clear_database(self)
```

**File**: `import_data.py:56-61`

**Description**: Clear all existing data from the database.

**Parameters**: None

**Returns**: None

**Cypher Query**: `MATCH (n) DETACH DELETE n`

**Warning**: Destructive operation - removes all nodes and relationships

##### load_schema()

```python
def load_schema(self, schema_path: Path)
```

**File**: `import_data.py:63-120`

**Description**: Load schema constraints, indexes, and persona data from JSON file.

**Parameters**:
- `schema_path` (Path): Path to schema JSON file

**Returns**: None

**Creates**:
1. **Constraints** (lines 77-89):
   - `PersonaAgent.persona_id` UNIQUE
   - `PersonaAgent.name` UNIQUE
   - `Memory.memory_id` UNIQUE
   - `Goal.goal_id` UNIQUE

2. **Indexes** (lines 93-98):
   - `PersonaAgent.attachment_style`
   - `Memory.memory_type`
   - `Goal.goal_type`
   - `RELATIONSHIP.relationship_type`

3. **Persona Data**: Calls `_load_family_graph_data()` to import personas and relationships

**Example**:
```python
from pathlib import Path

schema_file = Path("graphs/lila-graph-schema-v8.json")
importer.load_schema(schema_file)
```

##### import_seed_data()

```python
def import_seed_data(self, seed_data_path: Path)
```

**File**: `import_data.py:282-310`

**Description**: Import seed data from Cypher file.

**Parameters**:
- `seed_data_path` (Path): Path to Cypher seed data file

**Returns**: None

**Behavior**:
- Splits Cypher script on semicolons
- Executes each statement sequentially
- Skips empty statements
- Logs execution progress

**Example**:
```python
seed_file = Path("seed_data.cypher")
importer.import_seed_data(seed_file)
```

##### create_default_personas()

```python
def create_default_personas(self)
```

**File**: `import_data.py:312-377`

**Description**: Create default personas if no data imported.

**Parameters**: None

**Returns**: None

**Creates**:
1. **Lila**: AI Research Assistant, age 28, secure attachment
   - Openness: 0.8
   - Conscientiousness: 0.75
   - Extraversion: 0.65
   - Agreeableness: 0.85
   - Neuroticism: 0.3

2. **Alex**: Software Engineer, age 45, secure attachment
   - Openness: 0.7
   - Conscientiousness: 0.8
   - Extraversion: 0.4
   - Agreeableness: 0.7
   - Neuroticism: 0.45

3. **Relationship**: Friendship between Lila and Alex
   - Trust: 7.5
   - Intimacy: 6.5
   - Strength: 7.0

##### verify_import()

```python
def verify_import(self)
```

**File**: `import_data.py:379-407`

**Description**: Verify data was imported successfully.

**Parameters**: None

**Returns**: None

**Verification Checks**:
- Count of PersonaAgent nodes
- Count of RELATIONSHIP edges
- Count of Memory nodes
- Count of Goal nodes

**Output**: Prints verification summary to stdout

**Example**:
```python
importer.verify_import()
# ✓ Found 5 personas
# ✓ Found 8 relationships
# ✓ Found 12 memories
# ✓ Found 7 goals
```

#### CLI Usage

**File**: `import_data.py:409-465`

**Entry Point**: `main()`

**Command**:
```bash
python import_data.py [OPTIONS]
```

**Arguments**:
- `--seed-data`: Seed data Cypher file (default: `seed_data.cypher`)
- `--schema`: Schema JSON file (default: `graphs/lila-graph-schema-v8.json`)
- `--uri`: Neo4j URI (default: `bolt://localhost:7687`)
- `--user`: Neo4j username (default: `neo4j`)
- `--password`: Neo4j password (or use `NEO4J_PASSWORD` env var)
- `--create-defaults`: Create default personas if no seed data found

**Example**:
```bash
# Import with schema
python import_data.py \
  --schema graphs/lila-graph-schema-v8.json \
  --create-defaults

# Import with seed data
python import_data.py \
  --seed-data seed_data.cypher \
  --schema graphs/lila-graph-schema-v8.json \
  --uri bolt://localhost:7687 \
  --user neo4j \
  --password mypassword
```

---

### Neo4jDataExporter

**File**: `/home/donbr/lila-graph/lila-mcp/export_data.py:21-294`

**Purpose**: Exports psychological intelligence data from Neo4j for MCP standalone seeding.

#### Class Definition

```python
class Neo4jDataExporter:
    """Exports psychological intelligence data from Neo4j for MCP standalone seeding."""
```

#### Initialization

```python
def __init__(self, uri: str, user: str, password: str)
```

**File**: `export_data.py:24-26`

**Description**: Initialize Neo4j connection.

**Parameters**:
- `uri` (str): Neo4j connection URI
- `user` (str): Neo4j username
- `password` (str): Neo4j password

**Instance Variables**:
- `self.driver`: Neo4j driver instance

**Example**:
```python
from export_data import Neo4jDataExporter

exporter = Neo4jDataExporter(
    uri="bolt://localhost:7687",
    user="neo4j",
    password="your_password"
)
```

#### Methods

##### close()

```python
def close(self)
```

**File**: `export_data.py:28-30`

**Description**: Close Neo4j connection.

**Parameters**: None

**Returns**: None

##### export_personas()

```python
def export_personas(self) -> List[Dict[str, Any]]
```

**File**: `export_data.py:32-62`

**Description**: Export all PersonaAgent nodes with their psychological profiles.

**Parameters**: None

**Returns**: List of persona dictionaries

**Exported Fields**:
- `persona_id`, `name`, `age`, `role`, `description`
- `attachment_style`
- `openness`, `conscientiousness`, `extraversion`, `agreeableness`, `neuroticism`
- `trust_level`, `relationship_history`, `communication_style`
- `created_at`, `updated_at`

**Example**:
```python
personas = exporter.export_personas()
print(f"Exported {len(personas)} personas")
```

##### export_relationships()

```python
def export_relationships(self) -> List[Dict[str, Any]]
```

**File**: `export_data.py:64-89`

**Description**: Export all relationships between personas.

**Parameters**: None

**Returns**: List of relationship dictionaries

**Exported Fields**:
- `persona1_id`, `persona2_id`
- `trust_level`, `intimacy_level`, `relationship_strength`
- `interaction_count`, `last_interaction`
- `relationship_type`, `emotional_valence`
- `created_at`, `updated_at`

**Example**:
```python
relationships = exporter.export_relationships()
print(f"Exported {len(relationships)} relationships")
```

##### export_memories()

```python
def export_memories(self) -> List[Dict[str, Any]]
```

**File**: `export_data.py:91-113`

**Description**: Export memory nodes associated with personas.

**Parameters**: None

**Returns**: List of memory dictionaries

**Exported Fields**:
- `persona_id`, `memory_id`
- `content`, `memory_type`
- `importance_score`, `emotional_valence`
- `participants`, `created_at`

##### export_goals()

```python
def export_goals(self) -> List[Dict[str, Any]]
```

**File**: `export_data.py:115-139`

**Description**: Export goal nodes associated with personas.

**Parameters**: None

**Returns**: List of goal dictionaries

**Exported Fields**:
- `persona_id`, `goal_id`
- `goal_type`, `description`
- `progress`, `target_persona`
- `priority`, `status`
- `created_at`, `updated_at`

##### generate_cypher_script()

```python
def generate_cypher_script(
    self,
    personas: List[Dict],
    relationships: List[Dict],
    memories: List[Dict],
    goals: List[Dict]
) -> str
```

**File**: `export_data.py:141-243`

**Description**: Generate Cypher import script from exported data.

**Parameters**:
- `personas` (List[Dict]): Exported persona data
- `relationships` (List[Dict]): Exported relationship data
- `memories` (List[Dict]): Exported memory data
- `goals` (List[Dict]): Exported goal data

**Returns**: Multi-line Cypher script string

**Script Structure**:
1. Header comments
2. Clear existing data
3. CREATE statements for personas
4. CREATE statements for relationships
5. CREATE statements for memories
6. CREATE statements for goals

**Escaping**: Handles quotes and newlines in string values

**Example**:
```python
personas = exporter.export_personas()
relationships = exporter.export_relationships()
memories = exporter.export_memories()
goals = exporter.export_goals()

script = exporter.generate_cypher_script(
    personas, relationships, memories, goals
)

with open("seed_data.cypher", "w") as f:
    f.write(script)
```

#### CLI Usage

**File**: `export_data.py:245-294`

**Entry Point**: `main()`

**Command**:
```bash
python export_data.py [OPTIONS]
```

**Arguments**:
- `--output`: Output Cypher file (default: `seed_data.cypher`)
- `--uri`: Neo4j URI (default: `bolt://localhost:7687`)
- `--user`: Neo4j username (default: `neo4j`)
- `--password`: Neo4j password (or use `NEO4J_PASSWORD` env var)

**Example**:
```bash
# Export to default file
python export_data.py

# Export to custom file
python export_data.py \
  --output custom_seed.cypher \
  --uri bolt://localhost:7687 \
  --user neo4j \
  --password mypassword
```

**Workflow**:
1. Connect to Neo4j
2. Export personas
3. Export relationships
4. Export memories
5. Export goals
6. Generate Cypher script
7. Write to output file
8. Display summary

---

## Configuration

### Environment Variables

#### Neo4j Connection

**Required for LilaMCPServer and data management**:

```bash
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your_password_here
```

**Optional**:
```bash
NEO4J_TIMEOUT=30
NEO4J_MAX_RETRY_TIME=60
```

**Details**:
- `NEO4J_URI`: Connection string (bolt://, neo4j://, or neo4j+s://)
- `NEO4J_USER`: Database username (default: neo4j)
- `NEO4J_PASSWORD`: Required for authentication
- `NEO4J_TIMEOUT`: Query timeout in seconds
- `NEO4J_MAX_RETRY_TIME`: Maximum retry duration for connection

**Used In**:
- `lila_mcp_server.py:49-51`
- `simple_lila_mcp_server.py:105-107`
- `import_data.py:428`
- `export_data.py:260`

#### Figma Integration

**Optional for UX design workflows**:

```bash
FIGMA_ACCESS_TOKEN=your_token_here
FIGMA_FILE_ID=your_file_id
```

**Setup**:
1. Go to Figma Settings → Personal Access Tokens
2. Generate new token with required scopes
3. Set environment variable

**Used In**:
- `tools/figma_integration.py:12`
- `tools/mcp_registry.py:118-119`

#### Vercel v0 Integration

**Optional for UI generation**:

```bash
V0_API_KEY=your_api_key_here
```

**Setup**:
1. Get API key from Vercel v0 dashboard
2. Set environment variable

**Used In**:
- `tools/mcp_registry.py:123`

### Configuration Files

#### .env File

**File**: `.env` (not committed to git)

**Template**: `.env.example`

```bash
# Neo4j Configuration
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your_password_here

# Optional: Neo4j Advanced Settings
NEO4J_TIMEOUT=30
NEO4J_MAX_RETRY_TIME=60

# Optional: Figma Integration
FIGMA_ACCESS_TOKEN=your_token_here
FIGMA_FILE_ID=your_file_id

# Optional: Vercel v0 Integration
V0_API_KEY=your_api_key_here
```

**Loading**: Use `python-dotenv` to load from `.env` file

```python
from dotenv import load_dotenv
load_dotenv()

import os
neo4j_uri = os.getenv("NEO4J_URI")
```

#### Docker Compose Configuration

**File**: `docker-compose.yml`

**Neo4j Service**:
```yaml
neo4j:
  environment:
    NEO4J_AUTH: "neo4j/${NEO4J_PASSWORD:-passw0rd}"
    NEO4J_PLUGINS: '["apoc"]'
    NEO4J_dbms_security_auth__enabled: "true"
```

**MCP Server Service**:
```yaml
lila-mcp-server:
  environment:
    NEO4J_URI: bolt://neo4j:7687
    NEO4J_USER: ${NEO4J_USER:-neo4j}
    NEO4J_PASSWORD: ${NEO4J_PASSWORD:-password}
```

---

## Usage Patterns and Best Practices

### Starting an MCP Server

#### Production Server with Neo4j

```python
from lila_mcp_server import LilaMCPServer
import asyncio
import os

# Set environment variables
os.environ['NEO4J_URI'] = 'bolt://localhost:7687'
os.environ['NEO4J_USER'] = 'neo4j'
os.environ['NEO4J_PASSWORD'] = 'your_password'

async def main():
    server = LilaMCPServer()
    try:
        await server.run_server(host="0.0.0.0", port=8765)
    finally:
        server.close()

asyncio.run(main())
```

#### Development Server with Mock Data

```python
from simple_lila_mcp_server import SimpleLilaMCPServer
import asyncio

async def main():
    server = SimpleLilaMCPServer()
    await server.run_server()

asyncio.run(main())
```

**Or using FastMCP CLI**:
```bash
fastmcp dev simple_lila_mcp_server.py
```

### Running an Orchestrator

#### Architecture Analysis

```python
from orchestrators.architecture_orchestrator import ArchitectureOrchestrator
import asyncio

async def main():
    orchestrator = ArchitectureOrchestrator(
        output_base_dir=Path("repo_analysis"),
        show_tool_details=True
    )
    await orchestrator.run_with_client()

asyncio.run(main())
```

**Or via CLI**:
```bash
python -m orchestrators.architecture_orchestrator
```

#### UX Design Workflow

```python
from orchestrators.ux_orchestrator import UXOrchestrator
import asyncio

async def main():
    orchestrator = UXOrchestrator(
        project_name="MyApp",
        output_base_dir=Path("ux_outputs")
    )
    await orchestrator.run_with_client()

asyncio.run(main())
```

### Creating a Custom Orchestrator

```python
from pathlib import Path
from typing import Dict, List
from orchestrators.base_orchestrator import BaseOrchestrator
from claude_agent_sdk import AgentDefinition

class CustomOrchestrator(BaseOrchestrator):
    def __init__(self):
        super().__init__(
            domain_name="custom",
            output_base_dir=Path("outputs"),
            show_tool_details=True
        )

        # Create output structure
        self.docs_dir = self.output_dir / "docs"
        self.create_output_structure()

    def get_agent_definitions(self) -> Dict[str, AgentDefinition]:
        return {
            "analyzer": AgentDefinition(
                description="Custom analyzer",
                prompt="You are a custom analyzer...",
                tools=["Read", "Write", "Grep", "Glob"],
                model="sonnet"
            )
        }

    def get_allowed_tools(self) -> List[str]:
        return ["Read", "Write", "Grep", "Glob", "Bash"]

    async def run(self):
        # Phase 1
        await self.execute_phase(
            phase_name="Analysis",
            agent_name="analyzer",
            prompt="Analyze the system...",
            client=self.client
        )

        # Verify outputs
        expected = [self.docs_dir / "analysis.md"]
        await self.verify_outputs(expected)

# Run it
async def main():
    orchestrator = CustomOrchestrator()
    await orchestrator.run_with_client()

import asyncio
asyncio.run(main())
```

### Using Agent Registry

```python
from agents.registry import AgentRegistry

# Initialize registry
registry = AgentRegistry()

# Discover all agents
all_agents = registry.discover_agents()
print(f"Found {len(all_agents)} agents")

# Load specific agent
ux_researcher = registry.load_agent('ux_researcher', domain='ux')
if ux_researcher:
    print(f"Description: {ux_researcher.description}")
    print(f"Tools: {ux_researcher.tools}")

# Load all agents for a domain
ux_agents = registry.load_domain_agents('ux')
for name, agent in ux_agents.items():
    print(f"{name}: {agent.description}")
```

### Using MCP Registry

```python
from tools.mcp_registry import MCPRegistry

# Initialize registry
mcp = MCPRegistry()

# Check server availability
if mcp.is_server_available('figma'):
    tools = mcp.get_server_tools('figma')
    print(f"Figma tools: {tools}")
else:
    # Get configuration requirements
    config = mcp.get_configuration_requirements('figma')
    print(f"Required: {config['required_env']}")

    # Get fallback options
    fallbacks = mcp.get_fallback_options('figma_get_file')
    for option in fallbacks:
        print(f"- {option}")
```

### Using Figma Integration

```python
from tools.figma_integration import FigmaIntegration
import os

# Set up
os.environ['FIGMA_ACCESS_TOKEN'] = 'your_token'

# Initialize
figma = FigmaIntegration()

# Check availability
if figma.is_available():
    # Get design context
    context = figma.get_design_context(file_id="abc123")

    # Export to code
    code = figma.export_to_code(
        component_id="comp456",
        framework="react"
    )
else:
    # Show setup instructions
    print(figma.get_setup_instructions())
```

### Importing Data to Neo4j

```python
from pathlib import Path
from import_data import Neo4jDataImporter

# Initialize importer
importer = Neo4jDataImporter(
    uri="bolt://localhost:7687",
    user="neo4j",
    password="your_password",
    max_retries=30
)

try:
    # Clear existing data (optional)
    importer.clear_database()

    # Load schema and create constraints
    schema_file = Path("graphs/lila-graph-schema-v8.json")
    importer.load_schema(schema_file)

    # Import seed data (optional)
    seed_file = Path("seed_data.cypher")
    if seed_file.exists():
        importer.import_seed_data(seed_file)
    else:
        # Create default personas
        importer.create_default_personas()

    # Verify import
    importer.verify_import()

finally:
    importer.close()
```

**Or via CLI**:
```bash
python import_data.py \
  --schema graphs/lila-graph-schema-v8.json \
  --create-defaults \
  --uri bolt://localhost:7687 \
  --user neo4j \
  --password your_password
```

### Exporting Data from Neo4j

```python
from pathlib import Path
from export_data import Neo4jDataExporter

# Initialize exporter
exporter = Neo4jDataExporter(
    uri="bolt://localhost:7687",
    user="neo4j",
    password="your_password"
)

try:
    # Export all data
    personas = exporter.export_personas()
    relationships = exporter.export_relationships()
    memories = exporter.export_memories()
    goals = exporter.export_goals()

    # Generate Cypher script
    script = exporter.generate_cypher_script(
        personas, relationships, memories, goals
    )

    # Write to file
    output_file = Path("seed_data.cypher")
    with open(output_file, "w") as f:
        f.write(script)

    print(f"Exported to {output_file}")

finally:
    exporter.close()
```

**Or via CLI**:
```bash
python export_data.py \
  --output seed_data.cypher \
  --uri bolt://localhost:7687 \
  --user neo4j \
  --password your_password
```

### Best Practices

#### 1. Error Handling

**Always use try-finally for resource cleanup**:

```python
server = LilaMCPServer()
try:
    await server.run_server()
finally:
    server.close()
```

#### 2. Environment Variables

**Use .env files for configuration**:

```python
from dotenv import load_dotenv
load_dotenv()  # Load from .env file

import os
neo4j_password = os.getenv("NEO4J_PASSWORD")
if not neo4j_password:
    raise ValueError("NEO4J_PASSWORD not set")
```

#### 3. Cost Tracking

**Monitor API costs in orchestrators**:

```python
# Orchestrator automatically tracks costs
orchestrator = ArchitectureOrchestrator()
await orchestrator.run_with_client()

# Access cost data
print(f"Total cost: ${orchestrator.total_cost:.4f}")
for phase, cost in orchestrator.phase_costs.items():
    print(f"  {phase}: ${cost:.4f}")
```

#### 4. Output Verification

**Always verify expected outputs were created**:

```python
expected_files = [
    Path("docs/analysis.md"),
    Path("diagrams/architecture.md")
]
all_created = await orchestrator.verify_outputs(expected_files)
if not all_created:
    print("Some outputs missing!")
```

#### 5. Permission Modes

**Use appropriate permission modes**:

```python
# Development: Ask for all tools
options = orchestrator.create_client_options(
    permission_mode="ask"
)

# Production: Auto-accept safe edits
options = orchestrator.create_client_options(
    permission_mode="acceptEdits"
)
```

#### 6. Caching

**Leverage registries for performance**:

```python
# Agent registry caches loaded agents
registry = AgentRegistry()
agent = registry.load_agent('analyzer')  # Cached after first load
```

#### 7. Mock Data for Testing

**Use SimpleLilaMCPServer for development**:

```python
# No Neo4j required
server = SimpleLilaMCPServer()
# Server has mock personas and relationships
```

#### 8. Cross-Orchestrator Communication

**Use mixin for multi-domain workflows**:

```python
class MultiDomainOrchestrator(
    BaseOrchestrator,
    CrossOrchestratorCommunication
):
    def __init__(self):
        BaseOrchestrator.__init__(self, domain_name="multi")
        CrossOrchestratorCommunication.__init__(self)

        # Register other orchestrators
        self.register_orchestrator("ux", UXOrchestrator())
        self.register_orchestrator("arch", ArchitectureOrchestrator())

    async def run(self):
        # Invoke other orchestrators
        result = await self.invoke_orchestrator(
            "ux",
            "design_phase",
            context={"requirement": "data"}
        )
```

### Common Pitfalls

#### 1. Forgetting to Close Connections

**Bad**:
```python
server = LilaMCPServer()
await server.run_server()
# Connection never closed
```

**Good**:
```python
server = LilaMCPServer()
try:
    await server.run_server()
finally:
    server.close()
```

#### 2. Not Checking Environment Variables

**Bad**:
```python
password = os.getenv("NEO4J_PASSWORD")
# May be None, causing connection errors
```

**Good**:
```python
password = os.getenv("NEO4J_PASSWORD")
if not password:
    raise ValueError("NEO4J_PASSWORD environment variable not set")
```

#### 3. Hardcoding File Paths

**Bad**:
```python
output_file = "docs/analysis.md"  # Relative path
```

**Good**:
```python
output_file = self.docs_dir / "analysis.md"  # Path object
# Or use absolute path
output_file = Path("/absolute/path/to/docs/analysis.md")
```

#### 4. Not Implementing Abstract Methods

**Bad**:
```python
class MyOrchestrator(BaseOrchestrator):
    def __init__(self):
        super().__init__(domain_name="my")
    # Missing: get_agent_definitions(), get_allowed_tools(), run()
```

**Good**:
```python
class MyOrchestrator(BaseOrchestrator):
    def __init__(self):
        super().__init__(domain_name="my")

    def get_agent_definitions(self):
        return {...}

    def get_allowed_tools(self):
        return [...]

    async def run(self):
        # Implement workflow
        pass
```

#### 5. Ignoring Return Types

**Bad**:
```python
result = await update_relationship_metrics(...)
# Result is JSON string, not dict
data = result["trust_level"]  # TypeError
```

**Good**:
```python
import json

result = await update_relationship_metrics(...)
data = json.loads(result)
trust = data["updated_relationship"]["trust_level"]
```

---

## Summary

This API reference provides comprehensive documentation for the Lila MCP codebase, covering:

1. **MCP Servers**: LilaMCPServer (production) and SimpleLilaMCPServer (development)
2. **Orchestrators**: BaseOrchestrator (framework), ArchitectureOrchestrator, UXOrchestrator
3. **Registries**: AgentRegistry (agent discovery), MCPRegistry (MCP server management)
4. **Integrations**: FigmaIntegration (design tools)
5. **Data Management**: Neo4jDataImporter, Neo4jDataExporter

### Key Features

- **Type Hints**: All parameters and returns documented with types
- **Examples**: Practical code examples for every API
- **File References**: Exact file paths and line numbers
- **Best Practices**: Recommended usage patterns and common pitfalls
- **Configuration**: Complete environment variable documentation

### Quick Reference

**File Locations**:
- MCP Servers: `/home/donbr/lila-graph/lila-mcp/lila_mcp_server.py`, `simple_lila_mcp_server.py`
- Orchestrators: `/home/donbr/lila-graph/lila-mcp/orchestrators/`
- Registries: `/home/donbr/lila-graph/lila-mcp/agents/registry.py`, `tools/mcp_registry.py`
- Data Management: `/home/donbr/lila-graph/lila-mcp/import_data.py`, `export_data.py`

**Environment Setup**:
```bash
# Required
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your_password

# Optional
FIGMA_ACCESS_TOKEN=your_token
V0_API_KEY=your_key
```

**Quick Start**:
```bash
# Start MCP server
python lila_mcp_server.py

# Run architecture analysis
python -m orchestrators.architecture_orchestrator

# Import data
python import_data.py --schema graphs/lila-graph-schema-v8.json --create-defaults
```

For additional information, see:
- Component Inventory: `repo_analysis/docs/01_component_inventory.md`
- Architecture Diagrams: `repo_analysis/diagrams/02_architecture_diagrams.md`
- Data Flows: `repo_analysis/docs/03_data_flows.md`
