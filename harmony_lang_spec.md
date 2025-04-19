# HarmonySpace DSL Specification

## Overview

HarmonySpace is a high-level declarative language for designing agent-based systems. It allows developers to define complex agent interactions, information flows, and coordination patterns using a YAML-like syntax. The language abstracts from the details of implementing the agent framework, while retaining its powerful capabilities for building complex agent systems.

## Core Principles

1. **Agent-Centric Design**: Agents are first-class citizens that encapsulate capabilities, knowledge, and behavior.
2. **Flow-Based Programming**: Systems are defined by information flows between agents and components.
3. **Declarative Configuration**: System topology and behavior are defined in a declarative manner.
4. **Human Integration**: First-class support for human-in-the-loop interactions.
5. **Composability**: Components can be composed to create more complex systems.

## Language Components

### 1. System Definition

All HarmonySpace systems start with a system definition that establishes the overall context and configuration.

```yaml
system:
  name: "Customer Support System"
  description: "Multi-agent system for handling customer inquiries"
  version: "1.0"
  config:
    default_llm: "gpt-4-turbo"
    persistence:
      type: "postgres"  # Options: memory, redis, postgres, mongodb
      connection: "postgresql://user:pass@localhost:5432/harmony_db"
    streaming: true
    execution_mode: "async"  # Options: sync, async
```

### 2. Agent Definition

Agents are the primary actors in the system, each with their own capabilities and responsibilities.

```yaml
agents:
  - id: "router"
    type: "react"  # Options: react, structured_output, custom
    description: "Routes customer queries to appropriate specialized agents"
    llm:
      model: "gpt-4-turbo"
      temperature: 0.2
    memory:
      type: "conversation_history"
      max_tokens: 8000
    system_prompt: |
      You are a routing agent that analyzes customer inquiries and directs them
      to the most appropriate specialized agent.
    tools:
      - name: "route_to_agent"
        description: "Route the inquiry to a specialized agent"
        parameters:
          agent_id: 
            type: "string"
            description: "ID of the agent to handle this inquiry"
          reason:
            type: "string" 
            description: "Reason for selecting this agent"

  - id: "tech_support"
    type: "react"
    description: "Handles technical product issues"
    system_prompt: |
      You are a technical support specialist who helps customers
      resolve technical issues with our products.
    tools:
      - name: "search_knowledge_base"
        # Tool definition...
      - name: "escalate_to_human"
        # Tool definition...
```

### 3. Flow Definition

Flows define how information and control moves between agents in the system.

```yaml
flows:
  - name: "main_flow"
    description: "Main customer support flow"
    entry_point: "router"
    
    connections:
      - from: "router"
        to: "tech_support"
        condition: "state.agent_selection.agent_id == 'tech_support'"
      
      - from: "router"
        to: "billing_support"
        condition: "state.agent_selection.agent_id == 'billing_support'"
      
      - from: "tech_support"
        to: "router"
        condition: "state.needs_routing == true"
      
      - from: "tech_support"
        to: "END"
        condition: "state.query_resolved == true"
```

### 4. State Definition

Defines the structure of state that flows through the system.

```yaml
state:
  schema:
    messages:
      type: "list"
      description: "Conversation history"
    
    current_agent:
      type: "string"
      description: "Currently active agent"
    
    agent_selection:
      type: "object"
      properties:
        agent_id:
          type: "string"
        reason:
          type: "string"
    
    query_resolved:
      type: "boolean"
      default: false
```

### 5. Human-in-the-Loop Integration

Defines points where human intervention is required or possible.

```yaml
human_integration:
  - id: "expert_review"
    description: "Expert reviews solution before presenting to customer"
    type: "breakpoint"  # Options: breakpoint, time_travel, edit_state, review_tool_calls
    trigger:
      node: "tech_support"
      condition: "state.solution_confidence < 0.8"
    actions:
      - edit_solution
      - approve
      - reject
```

### 6. Subgraphs

Allows composition of complex systems from reusable components.

```yaml
subgraphs:
  - id: "knowledge_retrieval"
    description: "Knowledge retrieval subsystem"
    file: "knowledge_retrieval.harmony"  # Import from external file
    config:
      embedding_model: "text-embedding-3-large"
      
  - id: "customer_verification"
    description: "Customer identity verification"
    # Inline definition
    agents:
      - id: "identity_checker"
        # Agent definition...
    flows:
      # Flow definition...
```

## Advanced Features

### 1. Conditional Edges and Branching

```yaml
flows:
  - name: "support_flow"
    connections:
      - from: "issue_analyzer"
        to:
          - node: "simple_resolver"
            condition: "state.complexity_score < 3"
          - node: "complex_resolver"
            condition: "state.complexity_score >= 3"
```

### 2. Parallel Execution

```yaml
flows:
  - name: "research_flow"
    parallel:
      - from: "research_coordinator"
        to: ["web_searcher", "doc_analyzer", "code_examiner"]
        wait_for_all: true
        reduce_to: "research_synthesizer"
        reducer: "merge_research_results"
```

### 3. Streaming Configuration

```yaml
system:
  config:
    streaming:
      enabled: true
      mode: "tokens"  # Options: tokens, messages, events, values
      nodes: ["assistant", "researcher"]  # Only stream from these nodes, or all if not specified
```

### 4. Persistence and Storage

```yaml
system:
  config:
    persistence:
      type: "postgres"
      connection: "postgresql://user:pass@localhost:5432/harmony_db"
      ttl: 86400  # Time-to-live in seconds
      prefetch: true
```

### 5. Interruption Handling

```yaml
system:
  config:
    interruption:
      enabled: true
      resumable: true
      checkpoint_interval: 5  # seconds
```

## Examples

### Simple Customer Support System

```yaml
system:
  name: "Basic Customer Support"
  config:
    persistence:
      type: "memory"
    streaming: true

agents:
  - id: "support_agent"
    type: "react"
    system_prompt: "You are a helpful customer support agent..."
    tools:
      - name: "search_knowledge_base"
        # Tool definition...
      - name: "escalate_to_human"
        # Tool definition...

flows:
  - name: "support_flow"
    entry_point: "support_agent"
    connections:
      - from: "support_agent"
        to: "END"
        condition: "state.query_resolved == true"
      - from: "support_agent"
        to: "human_support"
        condition: "state.escalated == true"

human_integration:
  - id: "human_support"
    type: "wait_user_input"
    description: "Human customer support agent takes over"
```

### Multi-Agent Research Team

```yaml
system:
  name: "Research Team"
  config:
    persistence:
      type: "redis"
      connection: "redis://localhost:6379/0"

agents:
  - id: "supervisor"
    type: "react"
    system_prompt: "You are the research team supervisor..."
    # Additional configuration...
  
  - id: "researcher"
    type: "react"
    system_prompt: "You are a researcher who finds relevant information..."
    # Additional configuration...
  
  - id: "writer"
    type: "react"
    system_prompt: "You are a writer who synthesizes research into clear reports..."
    # Additional configuration...

flows:
  - name: "research_flow"
    entry_point: "supervisor"
    connections:
      - from: "supervisor"
        to: "researcher"
        condition: "state.needs_research == true"
      
      - from: "researcher"
        to: "writer"
        condition: "state.research_complete == true"
      
      - from: "writer"
        to: "supervisor"
        condition: "state.draft_complete == true"
      
      - from: "supervisor"
        to: "END"
        condition: "state.report_approved == true"

human_integration:
  - id: "report_review"
    type: "review_tool_calls"
    trigger:
      node: "supervisor"
      condition: "state.report_ready_for_review == true"
```

## Compilation Process

HarmonySpace DSL is translated into executable code through a multi-stage compilation process:

1. **Parsing**: The YAML-like DSL is parsed into an intermediate representation
2. **Validation**: The system definition is validated for consistency and completeness
3. **Code Generation**: Executable code is generated from the validated definition
4. **Execution**: The generated code is executed to create the runtime system

## Extension Points

HarmonySpace supports extension through:

1. **Custom Agent Types**: Define new agent architectures beyond the built-in types
2. **Custom State Types**: Define application-specific state schemas
3. **Custom Reducers**: Define specialized functions for combining parallel results
4. **Integration Hooks**: Connect with external systems and services

## Harmony Space To Example Mapping

| HarmonySpace Concept | Implementation |
|----------------------|--------------------------|
| Agent Definition | StateGraph with ReAct or custom nodes |
| Flow Connections | Graph edges and conditional edges |
| State Schema | TypedDict or Pydantic models |
| Human Integration | Breakpoints, wait_for_user_input, etc. |
| Subgraphs | Nested StateGraph objects |
| Parallel Execution | Fan-out pattern with reducer |