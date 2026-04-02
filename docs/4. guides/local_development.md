# Local Development

**Репозиторий:** `/Users/is/personal/Projects/10_TrelloMCP`

---

## Что это за проект

TypeScript MCP-сервер для Trello. Запускается через stdio transport и предназначен для MCP-клиентов вроде Cursor.

---

## Требования

- Node.js 18+
- `npm`
- `TRELLO_API_KEY`
- `TRELLO_API_TOKEN`

---

## Установка

```bash
npm install
```

---

## Сборка

Полная сборка:

```bash
npm run build
```

Только TypeScript:

```bash
npm run compile
```

На 2026-04-02 `npm run compile` проходит успешно.

---

## Конфигурация окружения

Перед запуском должны быть заданы:

```bash
export TRELLO_API_KEY="..."
export TRELLO_API_TOKEN="..."
```

---

## Что важно знать

- Проект не содержит автоматических тестов.
- Большая часть tool-handlers бьет в реальный Trello API.
- Надежный HTTP-слой находится в `src/utils/api.ts`.
- Ветка `fork/main` не равна актуальному `origin/main`; перед серьезной доработкой сравнивать remotes обязательно.

---

## Быстрая проверка после изменений

Минимум:

```bash
npm run compile
git status --short
```

Если менялся tool-set или docs:

- обновить `docs/HANDOFF.md`
- обновить `docs/CHANGELOG.md`
- проверить, не устарели ли `README.md` и `ARCHITECTURE.md`
