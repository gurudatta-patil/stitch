# Ghost-Bridge

An MCP tool for Claude Code that enables seamless, autonomous cross-language execution.

> Request a Python library capability from inside a TypeScript project — Ghost-Bridge generates the isolated venv, the Python sidecar, and a fully-typed TypeScript client automatically.

## How it works

```
Claude Code  →  MCP tool: generate_ghost_bridge
                  ↓
          creates .ghost-bridge/.venv
          pip installs dependencies
          generates <bridge>.py  (JSON-RPC sidecar)
          generates <bridge>.ts  (typed async client)
```

The TypeScript client spawns Python as a child process and communicates via JSON-RPC over stdin/stdout — no HTTP, no ports, no zombies.

## Quick start (coming in Phase 2)

```ts
import { ImageProcessorBridge } from '.ghost-bridge/bridges/image_processor';

const bridge = new ImageProcessorBridge();
const { base64 } = await bridge.processImage({ path: './photo.jpg' });
bridge.destroy();
```

## Project status

Currently in Phase 1 — core IPC mechanics. See the roadmap for the full task breakdown.
