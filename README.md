# Land Scout Pod

> Autonomous SCOUT Worker Pod for Domain: `land`.

An edge-native, serverless micro-agent running 24/7 on Cloudflare Workers + D1, executing assigned domain tasks and communicating over the asynchronous Tincan event mesh.

- **Pod ID**: `land-scout-pod`
- **Role**: `scout`
- **Domain**: `land`
- **Database**: `land-scout-pod-db`

---

## 🏗️ Architecture

```
 ┌────────────────────────────────────────────────────────┐
 │            Central Orchestration Mesh / Hub            │
 │            (HTTP / Webhook / Event Mesh)               │
 └─────────────┬────────────────────────────▲─────────────┘
               │ task.assigned              │ heartbeat
               │ (Event Envelope)           │ task.progress / task.completed
               ▼                            │
 ┌──────────────────────────────────────────┴─────────────┐
 │         Land Scout Pod (Cloudflare Worker)          │
 │                                                        │
 │  ┌──────────────┐   ┌───────────────┐  ┌─────────────┐ │
 │  │ / (Cockpit)  │   │  Event Mesh   │  │  Cron / 5m  │ │
 │  │ Dark Mode UI │   │  Client Node  │  │  Keepalive  │ │
 │  └──────────────┘   └───────┬───────┘  └──────┬──────┘ │
 │                             │                 │        │
 │                     ┌───────▼─────────────────▼──────┐ │
 │                     │ Pluggable Domain Task Handler  │ │
 │                     │       (src/handler.ts)         │ │
 │                     └───────────────┬────────────────┘ │
 │                                     │                  │
 │                             ┌───────▼───────┐          │
 │                             │ Cloudflare D1 │          │
 │                             │ / SQLite WAL  │          │
 │                             └───────────────┘          │
 └────────────────────────────────────────────────────────┘
```

---

## 🛠️ Implementing Domain Logic

Implement this pod's specialized functionality in [`src/handler.ts`](src/handler.ts). The chassis automatically calls `handle(task, context)` whenever an assigned task arrives:

```typescript
import { TaskRecord, HandlerContext, TaskHandler } from "./types.js";

export class LandScoutPodHandler implements TaskHandler {
  public name = "land-scout-pod-handler";

  public canHandle(topic: string, _spec: any): boolean {
    return true; // Match topics this pod specializes in
  }

  public async handle(task: TaskRecord, context: HandlerContext): Promise<any> {
    // 1. Perform domain work (API calls, data pipelines, LLM prompts)
    const result = { taskId: task.taskId, status: "completed" };

    // 2. Report completion back to central hub over event mesh
    await context.publishEvent("task.completed", "hub", {
      taskId: task.taskId,
      workerPod: context.podConfig.id,
      event: "completed",
      result,
      timestamp: context.now,
    });

    return result;
  }
}
```

---

## 🚀 Deployment

### 1. Apply Database Schema
```bash
just cf-schema land-scout-pod-db
```

### 2. Deploy Worker to Cloudflare
```bash
just cf-deploy
```

Your live web Cockpit UI will be available at `https://land-scout-pod.<account>.workers.dev`.

---

## 🧪 Local Testing

```bash
# Run unit test suite
npm test

# Type check
npm run check

# Run local development server
npm run dev
```

---

## 📜 License
MIT License. Free to use, adapt, and build upon.
