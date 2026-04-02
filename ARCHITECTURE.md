# Architecture — Advanced Trello MCP Server

## Project Structure

```
src/
├── index.ts                    # MCP Server entry point + 3 resources
├── types/
│   └── common.ts               # Shared TypeScript types
├── utils/
│   └── api.ts                  # HTTP helpers and reliability layer
└── tools/                      # Tool modules by Trello API area
    ├── boards.ts               # Boards API (1 tool)
    ├── lists.ts                # Lists API (10 tools)
    ├── cards.ts                # Cards API (12 tools)
    ├── labels.ts               # Labels API (8 tools)
    └── actions.ts              # Actions API (4 tools)
```

**Current tool count: 35**

---

## Architecture Principles

### 1. Separation of Concerns

- **`index.ts`** — server init, credentials, tool module registration, and 3 MCP resources (`board-info`, `lists-info`, `cards-info`) that make direct Trello API calls
- **`tools/`** — one module per Trello API area
- **`types/`** — shared TypeScript interfaces and Zod enums
- **`utils/`** — shared HTTP helpers

### 2. Modularity by API Area

Each module in `tools/` exports a single `register*Tools` function:

```typescript
// tools/labels.ts
export function registerLabelsTools(server: McpServer, credentials: TrelloCredentials) {
    server.tool('create-label', ...);
    server.tool('get-label', ...);
    // ...
}
```

### 3. Centralized Types

```typescript
// types/common.ts
export interface TrelloCredentials {
    apiKey: string;
    apiToken: string;
}

export const TrelloColorEnum = z.enum(['yellow', 'purple', 'blue', ...]);
```

### 4. HTTP Helpers

`utils/api.ts` has two layers:

**Reliability layer** (used by all handlers):
- `fetchWithRetry` — keep-alive HTTPS agent, sliding window rate limiter (80 req/10s), exponential backoff with jitter, 60s timeout

**High-level wrappers** (partially used):
- `trelloGet`, `trelloPost`, `trelloPut`, `trelloDelete` — wrap credentials validation, URL construction, `fetchWithRetry`, JSON parsing, and response formatting
- `createTrelloUrl`, `validateCredentials`, `createSuccessResponse`, `createErrorResponse`

Most handlers currently call `fetchWithRetry` directly and duplicate boilerplate (credentials check, URL construction, response formatting). Migration to `trelloGet/Post/Put/Delete` is planned (T03).

---

## File Sizes (current)

```
src/index.ts              99 lines
src/types/common.ts       ~60 lines
src/utils/api.ts          356 lines
src/tools/boards.ts       92 lines
src/tools/lists.ts        539 lines
src/tools/cards.ts        780 lines
src/tools/labels.ts       427 lines
src/tools/actions.ts      249 lines
```

---

## History

### Before (monolithic)

```
src/
└── index.ts  (2,408 lines)
    ├── 44 tools mixed together
    ├── inline types
    └── duplicated API logic
```

The original monolith is preserved at `src/index.original.ts`.

### After modular refactor (b6afe55, June 2025)

Modular structure was introduced. During the refactor 12 action-tools were not migrated from the monolith — the tool count dropped from 44 to 32. Documentation was not updated to reflect this.

### Since then

| Date | Change | Count |
| --- | --- | --- |
| Feb 2026 | `update-card` added | 33 |
| Mar 2026 | reliability layer, `get-card-attachments`, `download-card-attachments` | 35 |

---

## Adding New APIs

### 1. Create module

```typescript
// src/tools/members.ts
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { TrelloCredentials } from '../types/common.js';

export function registerMembersTools(server: McpServer, credentials: TrelloCredentials) {
    server.tool('get-member', ...);
}
```

### 2. Register in index

```typescript
// src/index.ts
import { registerMembersTools } from './tools/members.js';
registerMembersTools(server, credentials);
```

### 3. Build

```bash
npm run build
```

---

## Development Scripts

```bash
npm run build     # TypeScript compile + shebang injection
npm run compile   # TypeScript compile only (no shebang)
```

---

## Testing

No automated tests exist. Verification is manual via MCP client or `npm run compile` for type checking.

---

## Roadmap

Current: **35 tools** across 5 modules.

Planned (see `docs/2. specifications/S01_gap_closure.md`):
- Restore 12 lost action-tools → **48 tools**
- Add `get-card-comments`
- Add `due`/`start` to `update-card`
- Migrate handlers to `trelloGet/Post/Put/Delete`
