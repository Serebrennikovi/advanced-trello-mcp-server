# S01_gap_closure

**Задачи:** T01 ✅ | T02 ⬜ | T03 ⬜ | T04 ⬜

## Контекст

База: локальная ветка `feat/reliability-batch-attachments` (поверх `fork/main` = `1d33124`).

После модульного рефактора (`b6afe55`, июнь 2025) проект потерял 12 action-tools из исходных 44. Reliability layer (`fetchWithRetry`, rate limiter, keep-alive) добавлен Codex в марте 2026 и работает. Текущий tool count: **35** (boards 1, lists 10, cards 12, labels 8, actions 4). Документация (`ARCHITECTURE.md`, `README.md`) частично устарела: неверные counts, несуществующий скрипт `npm run dev`, ложные claims про унификацию handlers.

Подробный аудит: [../5. unsorted/2026-04-02_trello_mcp_history_and_tool_gap_analysis.md](../5.%20unsorted/2026-04-02_trello_mcp_history_and_tool_gap_analysis.md)

---

## Проблема

1. **Документация врёт** — `ARCHITECTURE.md` говорит 44 tools и `npm run dev`, которого нет.
2. **Отсутствует `get-card-comments`** — инструмента "покажи комментарии карточки" нет ни в 44, ни в 35. Endpoint уже вызывается внутри `get-card-attachments`.
3. **`update-card` не поддерживает `due`/`start`** — поля были в PR #3, потерялись при merge PR #4 в upstream.
4. **Handlers дублируют boilerplate** — проверка credentials, сборка URL, `response.json()`, формат ответа пишутся вручную в каждом handler, хотя `trelloGet/Post/Put/Delete` в `src/utils/api.ts` это уже инкапсулируют.
5. **12 action-tools потеряны** — выпал весь action-graph: traversal к связанным сущностям, reaction CRUD, fine-grained introspection.

---

## Цель

Закрыть функциональные и документационные gaps до состояния, когда:

- документация соответствует коду
- агент может читать комментарии карточки по `cardId`
- `update-card` поддерживает `due`/`start`
- handlers не дублируют boilerplate
- action-graph восстановлен

---

## Не-цели

- Синк `fork/main` с `origin/main` — отдельное решение, не входит в эту спеку.
- Добавление новых API (members, organizations, checklists) — за пределами scope.
- Изменение transport или конфигурации MCP server.
- Изменение reliability layer — он работает.

---

## Изменения по файлам

| Файл | Изменение |
| --- | --- |
| `ARCHITECTURE.md` | Обновить counts, убрать `npm run dev`, исправить описание `index.ts` и статус helpers |
| `README.md` | Обновить counts и список tools |
| `src/tools/cards.ts` | Добавить `get-card-comments`; добавить `due`/`start` в `update-card`; перевести handlers на `trelloGet/Post/Put/Delete` |
| `src/tools/lists.ts` | Перевести handlers на `trelloGet/Post/Put/Delete` |
| `src/tools/labels.ts` | Перевести handlers на `trelloGet/Post/Put/Delete` |
| `src/tools/boards.ts` | Перевести handlers на `trelloGet/Post/Put/Delete` |
| `src/tools/actions.ts` | Перевести handlers на `trelloGet/Post/Put/Delete`; восстановить 12 потерянных tools |
| `src/utils/api.ts` | Без изменений (helpers уже есть) |
| `docs/HANDOFF.md` | Обновить статус и counts после выполнения задач |
| `docs/CHANGELOG.md` | Зафиксировать каждое изменение |

---

## Изменения по tool-set

База сравнения: локальная ветка `feat/reliability-batch-attachments`, **35 tools**.

### Добавляются

| Tool | Модуль | Endpoint |
| --- | --- | --- |
| `get-card-comments` | `cards.ts` | `GET /1/cards/{cardId}/actions?filter=commentCard` |
| `get-action-field` | `actions.ts` | `GET /actions/{id}/{field}` |
| `get-action-board` | `actions.ts` | `GET /actions/{id}/board` |
| `get-action-card` | `actions.ts` | `GET /actions/{id}/card` |
| `get-action-list` | `actions.ts` | `GET /actions/{id}/list` |
| `get-action-member` | `actions.ts` | `GET /actions/{id}/member` |
| `get-action-member-creator` | `actions.ts` | `GET /actions/{id}/memberCreator` |
| `get-action-organization` | `actions.ts` | `GET /actions/{id}/organization` |
| `update-comment-action` | `actions.ts` | `PUT /actions/{id}/text` |
| `create-action-reaction` | `actions.ts` | `POST /actions/{id}/reactions` |
| `get-action-reaction` | `actions.ts` | `GET /actions/{id}/reactions/{reactionId}` |
| `delete-action-reaction` | `actions.ts` | `DELETE /actions/{id}/reactions/{reactionId}` |
| `get-action-reactions-summary` | `actions.ts` | `GET /actions/{id}/reactionsSummary` |

### Расширяются (без нового tool name)

| Tool | Что меняется |
| --- | --- |
| `update-card` | Добавляются параметры `due` (datetime) и `start` (datetime) |

### Итог после всех изменений

**48 tools** (35 + 13 новых).

---

## Acceptance criteria

1. `npm run compile` проходит без ошибок
2. `get-card-comments` принимает `cardId` и возвращает массив comment actions
3. `update-card` принимает `due` и `start` как опциональные параметры
4. 12 восстановленных action-tools зарегистрированы в сервере и отвечают на валидный `actionId`
5. Ни один handler не содержит inline проверки credentials или inline сборки Trello URL — только вызов `trelloGet/Post/Put/Delete`
6. `ARCHITECTURE.md` содержит count **48** и не упоминает `npm run dev`
7. `README.md` содержит актуальный список tools

---

## Верификация

```bash
# Компиляция
npm run compile

# Подсчёт зарегистрированных tools
grep -c "server\.tool(" src/tools/*.ts

# Проверка отсутствия голого fetch в handlers
grep -rn "= await fetch(" src/tools/

# Проверка отсутствия inline credential check в handlers
grep -rn "apiKey && credentials" src/tools/
```

---

## Документы, которые нужно обновить

- `ARCHITECTURE.md` — counts, скрипты, описание helpers
- `README.md` — список tools и counts
- `docs/HANDOFF.md` — статус, tool count
- `docs/CHANGELOG.md` — запись по каждой задаче
