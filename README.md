# Open Agent JSON

Visual, production-ready AI workflows — portable as JSON

## Overview

Open Agent JSON enables you to define complex multi-agent systems and LLM-powered workflows using simple JSON configuration files instead of writing code. It provides a standardized format for:

- Multi-agent orchestration with LangGraph.js
- LLM model integration (OpenAI, Anthropic, Ollama)
- Agent-to-Agent (A2A) communication
- Model Context Protocol (MCP) server integration
- FilesystemBackend skills integration (System and Custom Skills)
- Interactive workflows with user interrupts
- Conditional routing and state management

## Key Features

- **Declarative Workflow Definition**: Define nodes, edges, and state transitions in JSON
- **Multi-Model Support**: Seamlessly integrate OpenAI, Anthropic, and Ollama models
- **A2A Protocol**: Enable agent-to-agent communication for complex multi-agent systems
- **MCP Integration**: Connect external tools via Model Context Protocol servers
- **Skills Integration (System and Custom Skills)**: FilesystemBackend skills support for file system operations
- **State Management**: Flexible state annotations with custom reducers
- **Interactive Flows**: Built-in support for user interrupts and approval gates

## Quick Example

```json
{
  "config": {
    "info": {
      "version": "1.0.0",
      "description": "Simple greeting workflow"
    }
  },
  "stateAnnotation": {
    "name": "GreetingState",
    "type": "Annotation.Root"
  },
  "annotation": {
    "messages": {
      "type": "string[]",
      "reducer": "(x, y) => x.concat(y)",
      "default": []
    }
  },
  "models": [],
  "nodes": [
    {
      "id": "greet",
      "handler": {
        "parameters": [
          {
            "name": "state",
            "parameterType": "state",
            "stateType": "typeof GreetingState.State"
          }
        ],
        "function": "const name = interrupt('What is your name?');\nreturn { messages: [`Hello ${name}!`] };"
      }
    }
  ],
  "edges": [
    { "from": "__start__", "to": "greet" },
    { "from": "greet", "to": "__end__" }
  ],
  "stateGraph": {
    "annotationRef": "GreetingState",
    "config": {
      "checkpointer": { "type": "MemorySaver" }
    }
  }
}
```

## Documentation

For complete file format specification, see [FILE_FORMAT.md](FILE_FORMAT.md).

The documentation covers:
- Complete schema reference for all sections
- Model configuration (OpenAI, Anthropic, Ollama)
- Node types (Function Nodes, Tool Nodes)
- Edge types (Normal, Conditional)
- State management patterns
- A2A and MCP integration
- Skills integration (System Skills and Custom Skills)
- Working examples and best practices

## Examples

Explore real-world implementations:

- **[kudosflow](https://github.com/akudo7/kudosflow)**: Example workflows including interactive flows, model integration, and A2A orchestration
- **[a2a-server](https://github.com/akudo7/a2a-server)**: A2A server implementation for multi-agent systems

## Technology Stack

This specification is powered by:
- **LangGraph.js**: Workflow orchestration framework
- **SceneGraphManager v2.0.0**: Private library for JSON-to-workflow compilation
- **LangChain**: LLM integration and tooling

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

**Note:** This specification references the SceneGraphManager v2.0.0 library (kudos-scene-graph-manager), which is a private component. For licensing and access inquiries, please contact [Akira Kudo](https://www.linkedin.com/in/akira-kudo-4b04163/).

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## Author

**Akira Kudo**
- LinkedIn: [akira-kudo-4b04163](https://www.linkedin.com/in/akira-kudo-4b04163/)
- GitHub: [@akudo7](https://github.com/akudo7)

## Related Projects

- [LangGraph.js](https://langchain-ai.github.io/langgraphjs/) - Official LangGraph.js documentation
- [kudosflow](https://github.com/akudo7/kudosflow) - Example workflows
- [a2a-server](https://github.com/akudo7/a2a-server) - A2A server implementation
