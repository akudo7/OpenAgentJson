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
    },
    "skills": {
      "enabled": boolean,
      "skillsPath": string,
      "backend": {
        "virtualMode": boolean,
        "rootDir": string
      }
    },
    "store": {
      "type": "InMemoryStore"
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

- **skills** (optional): Skills integration configuration
  - **enabled**: Enable skills support (default: false)
  - **skillsPath**: Relative path to skills directory (e.g., "skills")
  - **backend**: Filesystem backend configuration
    - **virtualMode**: Enable virtual filesystem mode (default: true)
    - **rootDir**: Root directory for filesystem operations (default: ".")

- **store** (optional): Key-value store configuration
  - **type**: Store implementation (`"InMemoryStore"`)

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
    },
    "skills": {
      "enabled": true,
      "skillsPath": "skills",
      "backend": {
        "virtualMode": true,
        "rootDir": "."
      }
    },
    "store": {
      "type": "InMemoryStore"
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
      "bindMcpServers": boolean,
      "bindSystemSkills": boolean
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
- **bindSystemSkills** (optional): Enable skills tool binding

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
      "useMcpServers": boolean,
      "useSystemSkills": boolean
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

Special node for handling tool calls (A2A/MCP/Skills).

**Fields:**
- **id**: Unique node identifier
- **type**: Must be `"ToolNode"`
- **useA2AServers**: Enable A2A server tools
- **useMcpServers**: Enable MCP server tools
- **useSystemSkills**: Enable skills tools

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
  "useA2AServers": true,
  "useSystemSkills": true
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

### Skills Integration

Open Agent JSON supports skills that provide file system operations through a virtual or real filesystem backend. Skills come in two types: **System Skills** (built-in) and **Custom Skills** (user-defined).

**Configuration:**

- Enable skills in `config.skills` with `enabled: true`
- Set `skillsPath` to the directory containing skill definitions
- Configure `backend.virtualMode` for virtual filesystem simulation
- Set `backend.rootDir` for the root directory of filesystem operations

**Model Integration:**

- Use `"bindSystemSkills": true` in model config to bind skills as tools
- Skills are automatically available to the model as callable tools

**Tool Node Integration:**

- Use `"useSystemSkills": true` in ToolNode to handle skill tool calls
- Skills execute filesystem operations and return results to the workflow

#### System Skills (Built-in)

SceneGraphManager provides the following 7 core built-in skills:

| Skill Name | Description | Key Features |
| ------------ | ------------- | ---------- |
| `read_file` | Read file contents from the filesystem | Supports offset/limit for large files, line-by-line reading |
| `write_file` | Write content to a file | Overwrite protection, creates parent directories |
| `edit_file` | String replacement in files | Uniqueness validation, replace_all option |
| `glob_files` | Pattern-based file search | Supports glob patterns (e.g., `**/*.ts`), recursive search |
| `grep_search` | Content search with regex support | Regex patterns, context lines (-A/-B/-C), file filtering |
| `bash_command` | Shell command execution | Safety checks, timeout support, environment variables |
| `web_fetch` | HTTP content fetching | GET/POST requests, header support, JSON/text responses |

**Tool Details:**

1. **read_file**
   - Read file contents with optional offset and limit
   - Supports reading specific line ranges for large files
   - Returns content with line numbers for easy navigation
   - Example: `read_file({ file_path: "config.json", offset: 0, limit: 100 })`

2. **write_file**
   - Write or overwrite file contents
   - Automatically creates parent directories if needed
   - Includes overwrite protection for existing files
   - Example: `write_file({ file_path: "output.txt", content: "data" })`

3. **edit_file**
   - Perform exact string replacement in files
   - Validates uniqueness to prevent unintended replacements
   - Supports `replace_all` option for multiple occurrences
   - Example: `edit_file({ file_path: "app.ts", old_string: "old", new_string: "new" })`

4. **glob_files**
   - Search for files using glob patterns
   - Supports recursive patterns like `**/*.ts` or `src/**/*.json`
   - Returns sorted list of matching file paths
   - Example: `glob_files({ pattern: "**/*.md", path: "." })`

5. **grep_search**
   - Search file contents using regex patterns
   - Supports context lines (-A, -B, -C) and case-insensitive search
   - Filter by file type or glob pattern
   - Example: `grep_search({ pattern: "function.*test", glob: "**/*.ts" })`

6. **bash_command**
   - Execute shell commands with safety checks
   - Supports timeout configuration and environment variables
   - Captures stdout and stderr output
   - Example: `bash_command({ command: "npm test", timeout: 30000 })`

7. **web_fetch**
   - Fetch content from HTTP/HTTPS URLs
   - Supports custom headers and POST data
   - Returns JSON or text responses
   - Example: `web_fetch({ url: "https://api.example.com/data", method: "GET" })`

#### Custom Skills

Custom skills extend functionality beyond filesystem operations. They are defined in the `skillsPath` directory with the following structure:

**Directory Structure:**

```text
skills/
├── skill-name/
│   ├── SKILL.md          # Skill metadata and instructions
│   └── implementation.ts # Skill implementation (optional)
```

**SKILL.md Format:**

```markdown
---
name: skill-name
description: Brief description of what the skill does
---

# Skill Title

## Overview
Detailed description of the skill's purpose and capabilities.

## Instructions
Step-by-step instructions for the AI agent to use this skill.

## Examples
Usage examples with expected inputs and outputs.
```

**Custom Skill Examples:**

For example implementations, see the [kudosflow](https://github.com/akudo7/kudosflow) repository:

- **arxiv-search**: Search arXiv preprint repository for research papers
- **langgraph-docs**: Fetch and use LangGraph.js documentation

**Example Configuration:**

```json
{
  "config": {
    "skills": {
      "enabled": true,
      "skillsPath": "skills",
      "backend": {
        "virtualMode": true,
        "rootDir": "."
      }
    },
    "store": {
      "type": "InMemoryStore"
    }
  },
  "models": [
    {
      "id": "skillsModel",
      "type": "openai",
      "config": {
        "model": "gpt-5.2",
        "temperature": 0.7
      },
      "bindSystemSkills": true,
      "systemPrompt": "You are an assistant with filesystem access and custom skills."
    }
  ],
  "nodes": [
    {
      "id": "agent",
      "handler": {
        "parameters": [
          {
            "name": "state",
            "parameterType": "state",
            "stateType": "typeof StateAnnotation.State"
          },
          {
            "name": "model",
            "parameterType": "model",
            "modelRef": "skillsModel"
          }
        ],
        "function": "const messages = state.messages;\nconst response = await model.invoke(messages);\nreturn { messages: [response] };"
      }
    },
    {
      "id": "tools",
      "type": "ToolNode",
      "useSystemSkills": true
    }
  ]
}
```

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
