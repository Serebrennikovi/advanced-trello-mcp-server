# Передача работы: Trello MCP Server

**Дата:** 2026-04-02
**Статус:** T01 done; T02–T04 в работе
**Архитектура:** см. [../ARCHITECTURE.md](../ARCHITECTURE.md)
**История:** см. [CHANGELOG.md](CHANGELOG.md)
**Последнее обновление:** 2026-04-02 — T01 закрыт: docs/ закоммичена, ARCHITECTURE.md и README.md синхронизированы с кодом

---

## Текущее состояние

Проект представляет собой TypeScript MCP-сервер для Trello со stdio transport, модульной структурой и кастомным HTTP-слоем для надежных запросов к Trello API.

### Что есть в коде сейчас

| Компонент | Статус | Комментарий |
| --- | --- | --- |
| MCP server | `done` | `src/index.ts`, stdio transport, 3 resources |
| Tool-set | `35 tools` | boards 1, lists 10, cards 12, labels 8, actions 4 |
| Reliability layer | `done` | `fetchWithRetry`, keep-alive, timeout, rate-limit, retries |
| Attachment workflow | `done` | `get-card-attachments`, `download-card-attachments` |
| Автотесты | `missing` | в репозитории нет test suite и test config |
| Операционная документация | `done` | `docs/` закоммичена, `README.md` и `ARCHITECTURE.md` синхронизированы с кодом (T01) |

### Важная историческая справка

- Исторические `44 tools` были реальными, но только в монолитном `src/index.ts`.
- Во время модульного рефактора (`b6afe55`) проект фактически сократился до `32 tools`, однако документация осталась с claim про `44`.
- Сейчас и локальная рабочая ветка, и актуальный `origin/main` содержат `35 tools`.

Подробности: [2026-04-02_trello_mcp_history_and_tool_gap_analysis.md](5.%20unsorted/2026-04-02_trello_mcp_history_and_tool_gap_analysis.md).

---

## Критически важно

| Что | Статус | Пояснение |
| --- | --- | --- |
| `origin` и `fork` | важно | `origin` указывает на upstream `adriangrahldev/advanced-trello-mcp-server`, `fork` на личный форк `Serebrennikovi/advanced-trello-mcp-server` |
| Drift веток | важно | по состоянию на 2026-04-02 `fork/main` отстает от `origin/main` на 5 коммитов |
| Корневая документация | актуальна | синхронизирована в T01 (2026-04-02) |
| `44 tools` | неактуально для модульного кода | это исторический count монолита, а не текущего state проекта |
| Общие API-хелперы | внедрены частично | `src/utils/api.ts` содержит reusable helpers, но модули в основном используют только `fetchWithRetry`, а не весь набор оберток |

---

## Документация

### Структура папок

| Папка | Назначение |
| --- | --- |
| `1. business requirements/` | Бизнес-требования и входящие запросы |
| `2. specifications/` | Технические спецификации и RFC |
| `3. tasks/` | Задачи в работе и архив |
| `4. guides/` | Локальные правила и инструкции |
| `5. unsorted/` | Исследования, заметки, неразложенные материалы |
| `6. backlog/` | Отложенные инициативы |

### Ключевые документы

| Документ | Описание |
| --- | --- |
| [HANDOFF.md](HANDOFF.md) | Этот документ |
| [CHANGELOG.md](CHANGELOG.md) | История изменений документации и значимых решений |
| [../README.md](../README.md) | Пользовательский обзор проекта и quick start |
| [../ARCHITECTURE.md](../ARCHITECTURE.md) | Историческое описание модульной архитектуры |
| [2026-04-02_trello_mcp_history_and_tool_gap_analysis.md](5.%20unsorted/2026-04-02_trello_mcp_history_and_tool_gap_analysis.md) | Аудит истории `44 tools`, потерь после рефактора и расхождений с upstream |

---

## Следующие шаги

Спека: [2. specifications/S01_gap_closure.md](2.%20specifications/S01_gap_closure.md)

| Задача | Файл | Статус |
| --- | --- | --- |
| T01: docs housekeeping | [Done/T01_docs_housekeeping_done.md](3.%20tasks/Done/T01_docs_housekeeping_done.md) | ✅ done |
| T02: cards API improvements | [T02_cards_api_improvements.md](3.%20tasks/T02_cards_api_improvements.md) | в очереди |
| T03: unify handlers | [T03_unify_handlers.md](3.%20tasks/T03_unify_handlers.md) | в очереди |
| T04: restore 12 action-tools | [T04_restore_action_tools.md](3.%20tasks/T04_restore_action_tools.md) | в очереди |

---

## Upstream и ветки

На 2026-04-02:

- `origin/main` указывает на `ed0e1e6`  
- `fork/main` указывает на `1d33124`  
- локальная рабочая ветка `feat/reliability-batch-attachments` содержит ваши коммиты поверх `fork/main`

Подробный разбор с рекомендацией по sync: [4. guides/upstream_sync_guide.md](4.%20guides/upstream_sync_guide.md).

---

## Запуск

```bash
npm install
npm run build
```

Для быстрой проверки TypeScript:

```bash
npm run compile
```

Для запуска нужен `TRELLO_API_KEY` и `TRELLO_API_TOKEN`.

Подробнее: [4. guides/local_development.md](4.%20guides/local_development.md).
