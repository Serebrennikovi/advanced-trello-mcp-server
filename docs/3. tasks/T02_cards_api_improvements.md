# T02_cards_api_improvements

## Контекст

Два gap-а в Cards API, которые логично закрыть вместе:

1. Нет инструмента для чтения комментариев карточки по `cardId`. Endpoint уже используется внутри `get-card-attachments`.
2. `update-card` не поддерживает `due`/`start` — поля были в PR #3, потерялись при merge PR #4.

Спека: [../2. specifications/S01_gap_closure.md](../2.%20specifications/S01_gap_closure.md)

## Что делаем

**`get-card-comments`** — новый tool в `src/tools/cards.ts`:

- параметры: `cardId` (required), `limit` (optional, default 50), `since` (optional ISO datetime), `before` (optional ISO datetime)
- endpoint: `GET /1/cards/{cardId}/actions?filter=commentCard`
- возвращает массив action-объектов с `id`, `date`, `memberCreator`, `data.text`
- использует `trelloGet`

**`update-card`** — расширить существующий tool:

- добавить `due` — опциональный ISO 8601 datetime или `null` (сброс)
- добавить `start` — опциональный ISO 8601 datetime или `null` (сброс)
- передавать в тело PUT только если указаны

## Что не делаем

- Не трогаем `create-card` / `create-cards` — `due`/`start` там уже есть
- Не добавляем `dueComplete` или другие поля карточки
- Не добавляем пагинацию для `get-card-comments`

## Файлы / модули

- `src/tools/cards.ts`

## Acceptance criteria

1. `npm run compile` проходит без ошибок
2. `get-card-comments` зарегистрирован, принимает `cardId` + опциональные фильтры
3. `update-card` принимает `due` и `start`, при `null` сбрасывает
4. Count cards tools: **13**, общий count: **36**

## Верификация

```bash
npm run compile
grep -c "server\.tool(" src/tools/cards.ts
```

## Какие документы обновить

- `ARCHITECTURE.md` — count 36
- `README.md` — добавить `get-card-comments`, обновить `update-card`
- `docs/HANDOFF.md` — tool count
- `docs/CHANGELOG.md` — запись
