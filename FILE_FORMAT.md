# Open Agent JSON File Format Specification

> Visual, production-ready AI workflows — portable as JSON

## Overview

Open Agent JSON is a declarative JSON format for defining AI agent workflows powered by LangGraph.js. It enables meta-programming of complex agent systems through configuration rather than code, supporting multi-agent orchestration, model integration, and interactive workflows.

## File Structure

An Open Agent JSON file consists of the following top-level sections:

```json
{
  "config": { },
  "stateAnnotation": { },
  "annotation": { },
  "models": [ ],
  "mcpServers": { },
  "nodes": [ ],
  "edges": [ ],
  "stateGraph": { }
}
```

---

## 1. Config Section

Global configuration for the workflow execution environment.

### Schema

```typescript
{
  "config": {
    "info": {
      "version": string,
      "description": string,
      "copyright": string,
      "author": string
    },
    "recursionLimit": number,
    "eventEmitter": {
      "defaultMaxListeners": number
    }
  }
}
```

### Fields

- **info**: Metadata about the workflow
  - **version**: Semantic version (e.g., "1.0.0")
  - **description**: Human-readable workflow description
  - **copyright**: Copyright notice
  - **author**: Author name

- **recursionLimit**: Maximum depth for recursive workflow execution (default: 25)

- **eventEmitter**: EventEmitter configuration
  - **defaultMaxListeners**: Max concurrent event listeners (default: 10)

### Example

```json
{
  "config": {
    "info": {
      "version": "1.0.0",
      "description": "Career counselor workflow",
      "copyright": "Copyright 2026",
      "author": "Akira Kudo"
    },
    "recursionLimit": 25,
    "eventEmitter": {
      "defaultMaxListeners": 10
    }
  }
}
```

---

## 2. State Annotation Section

Defines the root state annotation name used throughout the workflow.

### Schema

```typescript
{
  "stateAnnotation": {
    "name": string,
    "type": "Annotation.Root"
  }
}
```

### Fields

- **name**: Identifier for the state annotation (referenced by nodes)
- **type**: Must be `"Annotation.Root"` (fixed value)

### Example

```json
{
  "stateAnnotation": {
    "name": "TestWorkflowState",
    "type": "Annotation.Root"
  }
}
```

---

## 3. Annotation Section

Defines state fields with types, reducers, and default values.

### Schema

```typescript
{
  "annotation": {
    "<fieldName>": {
      "type": string,
      "reducer": string,
      "default": any
    }
  }
}
```

### Fields

- **fieldName**: State field identifier
  - **type**: TypeScript type expression (e.g., `"string"`, `"string[]"`, `"BaseMessage[]"`)
  - **reducer**: Function expression for merging state updates (e.g., `"(x, y) => x.concat(y)"`)
  - **default**: Initial value for the field

### Common Reducer Patterns

| Pattern | Reducer | Use Case |
|---------|---------|----------|
| Concatenate | `(x, y) => x.concat(y)` | Arrays (messages, logs) |
| Replace | `(x, y) => y || x` | Single values (status, counters) |
| Merge | `(x, y) => ({ ...x, ...y })` | Objects |
| Override | `(x, y) => y !== undefined ? y : x` | Nullable values |

### Example

```json
{
  "annotation": {
    "messages": {
      "type": "string[]",
      "reducer": "(x, y) => x.concat(y)",
      "default": []
    },
    "userName": {
      "type": "string",
      "reducer": "(x, y) => y || x",
      "default": ""
    },
    "userApproval": {
      "type": "boolean | null",
      "reducer": "(x, y) => y !== undefined ? y : x",
      "default": null
    }
  }
}
```

---

## 4. Models Section

Defines LLM models used by workflow nodes.

### Schema

```typescript
{
  "models": [
    {
      "id": string,
      "type": "OpenAI" | "Anthropic" | "Ollama",
      "config": {
        "model": string,
        "temperature": number,
        ...
      },
      "systemPrompt": string,
      "bindA2AServers": boolean,
      "bindMcpServers": boolean
    }
  ]
}
```

### Fields

- **id**: Unique identifier for referencing in nodes
- **type**: Model provider (`"OpenAI"`, `"Anthropic"`, `"Ollama"`)
- **config**: Provider-specific configuration
  - **model**: Model name (e.g., `"gpt-4o-mini"`, `"claude-sonnet-3-5-20241022"`)
  - **temperature**: Sampling temperature (0.0-2.0)
  - Additional provider-specific options

- **systemPrompt** (optional): Default system message
- **bindA2AServers** (optional): Enable Agent-to-Agent (A2A) tool binding
- **bindMcpServers** (optional): Enable MCP server tool binding

### Example

```json
{
  "models": [
    {
      "id": "adviceModel",
      "type": "OpenAI",
      "config": {
        "model": "gpt-4o-mini",
        "temperature": 0.7
      },
      "systemPrompt": "You are a career counselor."
    },
    {
      "id": "orchestrator",
      "type": "Anthropic",
      "config": {
        "model": "claude-sonnet-3-5-20241022",
        "temperature": 0.3
      },
      "bindA2AServers": true
    }
  ]
}
```

---

## 5. MCP Servers Section

Configuration for Model Context Protocol (MCP) servers providing external tools.

### Schema

```typescript
{
  "mcpServers": {
    "<serverName>": {
      "command": string,
      "args": string[]
    }
  }
}
```

### Fields

- **serverName**: Identifier for the MCP server
  - **command**: Executable command (e.g., `"npx"`, `"node"`)
  - **args**: Command arguments array

### Example

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/workspace"]
    }
  }
}
```

---

## 6. Nodes Section

Defines executable workflow nodes (states in the graph).

### Schema

```typescript
{
  "nodes": [
    {
      "id": string,
      "type": "ToolNode" | undefined,
      "handler": {
        "parameters": [
          {
            "name": string,
            "parameterType": "state" | "model",
            "stateType": string,      // for parameterType: "state"
            "modelRef": string         // for parameterType: "model"
          }
        ],
        "function": string
      },
      "useA2AServers": boolean,
      "useMcpServers": boolean
    }
  ]
}
```

### Node Types

#### Function Node

Standard node executing custom JavaScript logic.

**Fields:**
- **id**: Unique node identifier
- **handler**: Node execution configuration
  - **parameters**: Array of parameter definitions
    - **name**: Parameter variable name
    - **parameterType**: `"state"` or `"model"`
    - **stateType**: State annotation type (for state params)
    - **modelRef**: Model ID reference (for model params)
  - **function**: JavaScript function body as string

#### Tool Node

Special node for handling tool calls (A2A/MCP).

**Fields:**
- **id**: Unique node identifier
- **type**: Must be `"ToolNode"`
- **useA2AServers**: Enable A2A server tools
- **useMcpServers**: Enable MCP server tools

### Special Functions

- **interrupt(message)**: Pauses workflow execution and prompts user input

### Examples

#### Function Node

```json
{
  "id": "askName",
  "handler": {
    "parameters": [
      {
        "name": "state",
        "parameterType": "state",
        "stateType": "typeof TestWorkflowState.State"
      }
    ],
    "function": "const userInput = interrupt('What is your name?');\nconst userName = String(userInput).trim();\nreturn { messages: [`Hello ${userName}!`], userName: userName };"
  }
}
```

#### Function Node with Model

```json
{
  "id": "generateAdvice",
  "handler": {
    "parameters": [
      {
        "name": "state",
        "parameterType": "state",
        "stateType": "typeof AgentState.State"
      },
      {
        "name": "model",
        "parameterType": "model",
        "modelRef": "adviceModel"
      }
    ],
    "function": "const response = await model.invoke([{ role: 'user', content: `Advice for ${state.userJob}` }]);\nreturn { advice: response.content };"
  }
}
```

#### Tool Node

```json
{
  "id": "tools",
  "type": "ToolNode",
  "useA2AServers": true
}
```

---

## 7. Edges Section

Defines workflow transitions between nodes.

### Schema

```typescript
{
  "edges": [
    {
      "from": string,
      "to": string,
      "type": "normal" | "conditional",
      "condition": {
        "name": string,
        "handler": {
          "parameters": [ ],
          "output": string,
          "function": string
        },
        "possibleTargets": string[]
      }
    }
  ]
}
```

### Edge Types

#### Normal Edge

Direct transition from one node to another.

**Fields:**
- **from**: Source node ID (or `"__start__"` for entry)
- **to**: Target node ID (or `"__end__"` for exit)

#### Conditional Edge

Dynamic routing based on runtime logic.

**Fields:**
- **from**: Source node ID
- **type**: Must be `"conditional"`
- **condition**: Condition configuration
  - **name**: Condition identifier
  - **handler**: Evaluation logic
    - **parameters**: Parameter definitions (same as node parameters)
    - **output**: Return type (`"string"` for target node ID)
    - **function**: JavaScript function body
  - **possibleTargets**: Array of possible target node IDs

### Examples

#### Normal Edge

```json
{
  "from": "__start__",
  "to": "askName"
}
```

#### Conditional Edge

```json
{
  "from": "orchestrator",
  "type": "conditional",
  "condition": {
    "name": "routeBasedOnPhase",
    "handler": {
      "parameters": [
        {
          "name": "state",
          "parameterType": "state",
          "stateType": "typeof AgentState.State"
        }
      ],
      "output": "string",
      "function": "if (state.currentPhase === 'research') return 'researchNode';\nreturn 'evaluationNode';"
    },
    "possibleTargets": ["researchNode", "evaluationNode"]
  }
}
```

---

## 8. State Graph Section

Configures the LangGraph StateGraph instance.

### Schema

```typescript
{
  "stateGraph": {
    "annotationRef": string,
    "config": {
      "checkpointer": {
        "type": "MemorySaver" | "SqliteSaver"
      },
      "store": {
        "type": "InMemoryStore"
      }
    }
  }
}
```

### Fields

- **annotationRef**: Reference to `stateAnnotation.name`
- **config**: Graph configuration
  - **checkpointer**: State persistence backend
    - **type**: `"MemorySaver"` (in-memory) or `"SqliteSaver"` (persistent)
  - **store**: Key-value store configuration
    - **type**: `"InMemoryStore"`

### Example

```json
{
  "stateGraph": {
    "annotationRef": "TestWorkflowState",
    "config": {
      "checkpointer": {
        "type": "MemorySaver"
      }
    }
  }
}
```

---

## Complete Examples

### Simple Interactive Workflow

```json
{
  "config": {
    "info": {
      "version": "1.0.0",
      "description": "Name and job collector",
      "copyright": "Copyright 2026",
      "author": "Akira Kudo"
    },
    "recursionLimit": 25,
    "eventEmitter": {
      "defaultMaxListeners": 10
    }
  },
  "stateAnnotation": {
    "name": "InterruptWorkflowState",
    "type": "Annotation.Root"
  },
  "annotation": {
    "messages": {
      "type": "string[]",
      "reducer": "(x, y) => x.concat(y)",
      "default": []
    },
    "userName": {
      "type": "string",
      "reducer": "(x, y) => y || x",
      "default": ""
    }
  },
  "models": [],
  "nodes": [
    {
      "id": "askName",
      "handler": {
        "parameters": [
          {
            "name": "state",
            "parameterType": "state",
            "stateType": "typeof InterruptWorkflowState.State"
          }
        ],
        "function": "const userInput = interrupt('What is your name?');\nreturn { messages: [`Hello ${userInput}!`], userName: String(userInput) };"
      }
    }
  ],
  "edges": [
    {
      "from": "__start__",
      "to": "askName"
    },
    {
      "from": "askName",
      "to": "__end__"
    }
  ],
  "stateGraph": {
    "annotationRef": "InterruptWorkflowState",
    "config": {
      "checkpointer": {
        "type": "MemorySaver"
      }
    }
  }
}
```

### Model Integration Workflow

```json
{
  "config": {
    "info": {
      "version": "1.0.0",
      "description": "Career counselor with AI advice",
      "copyright": "Copyright 2026",
      "author": "Akira Kudo"
    },
    "recursionLimit": 25,
    "eventEmitter": {
      "defaultMaxListeners": 10
    }
  },
  "stateAnnotation": {
    "name": "CounselorState",
    "type": "Annotation.Root"
  },
  "annotation": {
    "messages": {
      "type": "string[]",
      "reducer": "(x, y) => x.concat(y)",
      "default": []
    },
    "userJob": {
      "type": "string",
      "reducer": "(x, y) => y || x",
      "default": ""
    },
    "advice": {
      "type": "string",
      "reducer": "(x, y) => y || x",
      "default": ""
    }
  },
  "models": [
    {
      "id": "adviceModel",
      "type": "OpenAI",
      "config": {
        "model": "gpt-4o-mini",
        "temperature": 0.7
      },
      "systemPrompt": "You are a career counselor."
    }
  ],
  "nodes": [
    {
      "id": "askJob",
      "handler": {
        "parameters": [
          {
            "name": "state",
            "parameterType": "state",
            "stateType": "typeof CounselorState.State"
          }
        ],
        "function": "const userJob = interrupt('What is your occupation?');\nreturn { messages: ['Generating advice...'], userJob: String(userJob) };"
      }
    },
    {
      "id": "generateAdvice",
      "handler": {
        "parameters": [
          {
            "name": "state",
            "parameterType": "state",
            "stateType": "typeof CounselorState.State"
          },
          {
            "name": "model",
            "parameterType": "model",
            "modelRef": "adviceModel"
          }
        ],
        "function": "const response = await model.invoke([{ role: 'user', content: `Advice for ${state.userJob}` }]);\nreturn { advice: response.content, messages: ['Advice generated'] };"
      }
    }
  ],
  "edges": [
    {
      "from": "__start__",
      "to": "askJob"
    },
    {
      "from": "askJob",
      "to": "generateAdvice"
    },
    {
      "from": "generateAdvice",
      "to": "__end__"
    }
  ],
  "stateGraph": {
    "annotationRef": "CounselorState",
    "config": {
      "checkpointer": {
        "type": "MemorySaver"
      }
    }
  }
}
```

---

## Advanced Features

### Agent-to-Agent (A2A) Communication

Open Agent JSON supports multi-agent orchestration where agents communicate via A2A protocol.

**Client Workflow:**
- Uses `"bindA2AServers": true` in model config
- Accesses remote agent tools via model invocation
- Tool node with `"useA2AServers": true` handles responses

**Server Workflow:**
- Exposed as A2A endpoint
- Receives messages from client workflows
- Returns structured responses

See [kudosflow](https://github.com/akudo7/kudosflow) and [a2a-server](https://github.com/akudo7/a2a-server) for examples.

---

## Best Practices

1. **State Design**: Keep state flat and minimal; use reducers to manage updates
2. **Node Granularity**: One node per logical step; avoid monolithic handlers
3. **Error Handling**: Wrap async operations in try-catch blocks
4. **Interrupts**: Use sparingly; prefer state-driven flows
5. **Model Selection**: Match model capability to task complexity
6. **Versioning**: Update `config.info.version` on breaking changes

---

## See Also

- [kudosflow Examples](https://github.com/akudo7/kudosflow)
- [a2a-server Implementation](https://github.com/akudo7/a2a-server)
- [LangGraph.js Documentation](https://langchain-ai.github.io/langgraphjs/)

**Note:** This specification is based on the SceneGraphManager v2.0.0 library (kudos-scene-graph-manager), which is a private component. For licensing and access inquiries, please contact Akira Kudo.
