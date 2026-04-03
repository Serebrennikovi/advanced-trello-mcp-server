# Trello MCP Server — Changelog

Все заметные изменения в документации и важных операционных решениях фиксируются здесь.

Формат основан на [Keep a Changelog](https://keepachangelog.com/ru/1.0.0/).

---

## [Не выпущено]

### Добавлено

- 2026-04-02 — T02: добавлен `get-card-comments` (GET /cards/{id}/actions?filter=commentCard, параметры: cardId, limit, since, before)
- 2026-04-02 — T02: `update-card` расширен полями `due` и `start` (ISO 8601 datetime или null для сброса)

### Исправлено

- 2026-04-03 — T02: `get-card-comments` error path унифицирован с `update-card` — `response.json().catch(()=>({}))` заменён на `response.text()` (plain-text ошибки Trello больше не теряются)

### Изменено

- 2026-04-02 — T02: count cards tools 12→13, общий count 35→36 (ARCHITECTURE.md, README.md, HANDOFF.md обновлены)

---

## [2026-04-02 T01]

### Добавлено

- 2026-04-02 — создана папка `docs/` с базовой структурой: `business requirements`, `specifications`, `tasks`, `guides`, `unsorted`, `backlog`
- 2026-04-02 — создан [HANDOFF.md](HANDOFF.md) как точка входа по текущему состоянию проекта, branch drift и следующим шагам
- 2026-04-02 — создана исследовательская записка [2026-04-02_trello_mcp_history_and_tool_gap_analysis.md](5.%20unsorted/2026-04-02_trello_mcp_history_and_tool_gap_analysis.md) с аудитом происхождения `44 tools`, потерянных action-tools, общих хелперов и отличий upstream
- 2026-04-02 — добавлены гайды: [doc_conventions.md](4.%20guides/doc_conventions.md), [local_development.md](4.%20guides/local_development.md), [specifications_guide.md](4.%20guides/specifications_guide.md), [task_decomposition_guide.md](4.%20guides/task_decomposition_guide.md), [upstream_sync_guide.md](4.%20guides/upstream_sync_guide.md)
- 2026-04-02 — T01: добавлена спека S01_gap_closure и задачи T01–T04

### Изменено

- 2026-04-02 — проект получил документированную структуру рабочей документации вместо разрозненных корневых файлов
- 2026-04-02 — T01: ARCHITECTURE.md синхронизирован с кодом (count 44→35, убран npm run dev, исправлено описание index.ts и helpers)
- 2026-04-02 — T01: README.md обновлён под актуальный tool-set (35 tools, списки карточек и actions)

---

## Шаблон записи

```markdown
- YYYY-MM-DD — краткое описание изменения
```

**Категории:**

| Категория | Когда использовать |
| --- | --- |
| `Добавлено` | Новые документы, функции, разделы |
| `Изменено` | Рефакторинг, изменение поведения, перепаковка |
| `Исправлено` | Исправление багов или ошибок в документации |
| `Удалено` | Удаленные файлы, разделы, функции |

---

Актуальное состояние проекта см. в [HANDOFF.md](HANDOFF.md).
