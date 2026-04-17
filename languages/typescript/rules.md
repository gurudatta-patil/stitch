# Language Rules - TypeScript (Client / Source)

TypeScript is the **source** (parent process). It spawns the target sidecar and provides a typed async API.

---

## Environment

| Concern | Rule |
|---------|------|
| Runtime | Node.js ≥ 18 (required for `crypto.randomUUID()` without import) |
| Spawn mode | `child_process.spawn` with `detached: false` |
| stdio | `['pipe', 'pipe', 'pipe']` - all three piped. Never `'inherit'` for stdin or stdout; it breaks the EOF watchdog. |
| Path resolution | Always use `path.join` + `os.platform()` check. Never string-concat paths. |

---

## Startup Contract

1. Spawn child process.
2. Wait for `{"ready": true}` line on stdout before sending any requests.
3. Expose a `ready: Promise<void>` or equivalent to callers.

---

## Shutdown / Signal Rules

All four hooks are **mandatory** in every generated client. Use the `killChild` helper - not a bare `proc.kill()` - for cross-platform safety:

```ts
function killChild(proc: ChildProcess): void {
  if (proc.killed || proc.exitCode !== null) return;
  if (process.platform === 'win32') {
    proc.kill();                       // TerminateProcess; no SIGTERM on Windows
  } else {
    proc.kill('SIGTERM');
    setTimeout(() => {
      if (!proc.killed) proc.kill('SIGKILL');
    }, 2000).unref();                  // .unref() prevents timer from keeping Node alive
  }
}

const kill = () => killChild(proc);
process.on('exit',              kill);
process.on('SIGINT',            () => { kill(); process.exit(0); });
process.on('SIGTERM',           () => { kill(); process.exit(0); });
process.on('uncaughtException', (e) => { kill(); throw e; });
```

- `SIGINT` = user hits **Ctrl+C**. Child must die before Node exits.
- `.unref()` on the escalation timer is mandatory - without it a dying process hangs for 2 s.
- On Windows, `SIGTERM` is not a real signal; `proc.kill()` with no argument calls `TerminateProcess` which is instant.

---

## stdout Framing (Chunked-Stream Safe)

**Never parse `data` events directly.** Node may split a large JSON object across multiple events.

```ts
let buffer = '';

child.stdout.on('data', (chunk: Buffer) => {
  buffer += chunk.toString('utf8');
  const lines = buffer.split('\n');
  buffer = lines.pop()!;           // keep the incomplete tail
  for (const line of lines) {
    if (!line.trim()) continue;
    try { handleMessage(JSON.parse(line)); }
    catch { process.stderr.write(`[bridge] bad line: ${line}\n`); }
  }
});
```

---

## ID Strategy

- Use `randomUUID()` from Node's built-in `crypto` module - **not** an incrementing counter.
- Counter IDs are fine for sequential calls but silently corrupt results under concurrency.
- Store pending calls in `Map<string, {resolve, reject}>` keyed by UUID.
- On `error` key in response: reject with `new Error(msg.error.message)` and attach `traceback` as a property.

---

## Cross-Platform Path Helper (mandatory in every client)

```ts
import os from 'os';
import path from 'path';

function resolveChildExecutable(venvRoot: string, execName: string): string {
  return os.platform() === 'win32'
    ? path.join(venvRoot, 'Scripts', `${execName}.exe`)
    : path.join(venvRoot, 'bin', execName);
}
```
