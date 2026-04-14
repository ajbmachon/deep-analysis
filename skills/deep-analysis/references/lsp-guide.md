# LSP Usage Guide

When instructing agents, include LSP guidance if the project has an LSP server configured. Available operations:

| Operation | Use for | Example |
|-----------|---------|---------|
| `documentSymbol` | Quick inventory of all classes/methods in a file | Map file contents without reading every line |
| `workspaceSymbol` | Find classes by name across workspace | Find all *Command classes, all *Interface types |
| `goToDefinition` | Trace where a type/method is defined | Verify import references |
| `goToImplementation` | Find concrete implementations of interfaces | Map interface -> implementation pairs |
| `findReferences` | Find all usages of a symbol | Gauge coupling, find consumers |
| `incomingCalls` | What calls this method? | Trace who dispatches a command |
| `outgoingCalls` | What does this method call? | Map dependencies of a handler |
| `hover` | Get type info + docblock | Quick type verification |

## LSP Budget Per Agent Type

- **Wave 1 overview**: 10-15 LSP calls (strategic — key interfaces, hotspot files)
- **Wave 2 deep-dive**: 20-40 LSP calls (thorough — every file in the package)
- **Wave 3 verification**: 10-20 LSP calls (targeted — verify specific claims)

LSP provides precision that grep cannot — use it for type relationships, interface implementations, and call chains. If LSP is not available, fall back to grep and file reading.
