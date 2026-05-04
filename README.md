# @meridian-agent/langgraph-checkpoint-cloudflare-d1

A LangGraph.js checkpoint saver backed by [Cloudflare D1](https://developers.cloudflare.com/d1/).

Use this when you are running LangGraph.js inside a Cloudflare Worker and want checkpoints stored in a D1 database through the Worker `env.DB` binding.

## Installation

```bash
npm install @meridian-agent/langgraph-checkpoint-cloudflare-d1
```

You also need `@langchain/langgraph-checkpoint` in your project. This package is designed for Cloudflare Workers and expects a D1 binding.

## Usage

Configure a D1 binding in `wrangler.toml`:

```toml
[[d1_databases]]
binding = "DB"
database_name = "langgraph-checkpoints"
database_id = "your-database-id"
```

Then create the saver from the Worker environment binding:

```ts
import { CloudflareD1Saver } from "@meridian-agent/langgraph-checkpoint-cloudflare-d1";

export interface Env {
  DB: D1Database;
}

export default {
  async fetch(_request: Request, env: Env): Promise<Response> {
    const checkpointer = new CloudflareD1Saver(env.DB);

    // Pass `checkpointer` to your LangGraph graph compile/configuration.
    // const app = graph.compile({ checkpointer });

    return new Response("OK");
  },
};
```

## What It Stores

The saver creates two D1 tables automatically on first use:

- `checkpoints`
- `writes`

The schema and LangGraph-level behavior are based on the official `SqliteSaver` from `@langchain/langgraph-checkpoint-sqlite`, adapted for Cloudflare D1's async API.

Supported operations:

- `getTuple(config)`
- `list(config, options)`
- `put(config, checkpoint, metadata)`
- `putWrites(config, writes, taskId)`
- `deleteThread(threadId)`

## Notes

- 0.0.3 — fix setup() crashing on Cloudflare D1 because db.exec() splits on newlines; switched to batch() with prepare().
- D1 is provided by Cloudflare through `env.DB`; there is no connection string helper.
- Setup is async and runs lazily before saver operations.
- Writes use `db.batch()` for multi-statement operations.
- SQL is SQLite-compatible and intended to run on Cloudflare D1.

## Local Development

Install dependencies:

```bash
bun install
```

Typecheck:

```bash
bun run typecheck
```

Build:

```bash
bun run build
```

The build outputs ESM JavaScript and TypeScript declarations to `dist`.

## Publishing

Before publishing, verify the package contents:

```bash
npm pack --dry-run
```

Publish publicly:

```bash
npm publish --access public
```
