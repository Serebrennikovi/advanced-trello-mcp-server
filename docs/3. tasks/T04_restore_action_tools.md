# T04_restore_action_tools

## Контекст

При модульном рефакторе (`b6afe55`) из монолита в `src/tools/actions.ts` перенесли только 4 из 14 action-tools. 12 потерянных tools покрывают action-graph traversal, reaction CRUD и comment-specific API. Референсная реализация хранится в `src/index.original.ts`.

Спека: [../2. specifications/S01_gap_closure.md](../2.%20specifications/S01_gap_closure.md)
Аудит: [../5. unsorted/2026-04-02_trello_mcp_history_and_tool_gap_analysis.md](../5.%20unsorted/2026-04-02_trello_mcp_history_and_tool_gap_analysis.md)

## Что делаем

Восстановить 12 tools в `src/tools/actions.ts`, используя `trelloGet/Post/Put/Delete`:

| Tool | Endpoint |
| --- | --- |
| `get-action-field` | `GET /actions/{id}/{field}` |
| `get-action-board` | `GET /actions/{id}/board` |
| `get-action-card` | `GET /actions/{id}/card` |
| `get-action-list` | `GET /actions/{id}/list` |
| `get-action-member` | `GET /actions/{id}/member` |
| `get-action-member-creator` | `GET /actions/{id}/memberCreator` |
| `get-action-organization` | `GET /actions/{id}/organization` |
| `update-comment-action` | `PUT /actions/{id}/text` |
| `create-action-reaction` | `POST /actions/{id}/reactions` |
| `get-action-reaction` | `GET /actions/{id}/reactions/{reactionId}` |
| `delete-action-reaction` | `DELETE /actions/{id}/reactions/{reactionId}` |
| `get-action-reactions-summary` | `GET /actions/{id}/reactionsSummary` |

## Что не делаем

- Не копируем из `src/index.original.ts` дословно — там старый стиль без helpers
- Не добавляем tools за пределами списка выше
- Не трогаем существующие 4 action-tools

## Файлы / модули

- `src/tools/actions.ts`

## Acceptance criteria

1. `npm run compile` проходит без ошибок
2. `grep -c "server\.tool(" src/tools/actions.ts` → **16**
3. Общий count tools: **48**
4. Ни один новый handler не использует голый `fetch` или инлайн URL

## Верификация

```bash
npm run compile
grep -c "server\.tool(" src/tools/actions.ts
grep -c "server\.tool(" src/tools/*.ts
```

## Какие документы обновить

- `ARCHITECTURE.md` — count 48, таблица модулей
- `README.md` — полный список Actions tools
- `docs/HANDOFF.md` — tool count, статус
- `docs/CHANGELOG.md` — запись
