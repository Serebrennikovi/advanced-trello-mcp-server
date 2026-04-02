# T03_unify_handlers

## Контекст

В `src/utils/api.ts` есть `trelloGet`, `trelloPost`, `trelloPut`, `trelloDelete` — они инкапсулируют проверку credentials, сборку URL, `fetchWithRetry`, парсинг ответа и формат success/error. Все handlers в `src/tools/*.ts` дублируют эту логику вручную.

Спека: [../2. specifications/S01_gap_closure.md](../2.%20specifications/S01_gap_closure.md)

## Что делаем

Перевести все handlers в пяти модулях на `trelloGet/Post/Put/Delete`:

- `src/tools/boards.ts`
- `src/tools/lists.ts`
- `src/tools/labels.ts`
- `src/tools/cards.ts`
- `src/tools/actions.ts`

После перевода в handlers не должно оставаться:

- инлайн-проверок `if (!credentials.apiKey || !credentials.apiToken)`
- инлайн-сборки URL через `new URL()` или строковую конкатенацию
- прямых вызовов `fetchWithRetry` (кроме attachment download — там нужны бинарные данные)
- инлайн `try/catch` с `createErrorResponse`

## Что не делаем

- Не меняем сигнатуры и имена tools
- Не меняем `src/utils/api.ts`
- Не трогаем attachment download в `cards.ts`
- Не добавляем новые tools — это чистый рефактор

## Файлы / модули

- `src/tools/boards.ts`
- `src/tools/lists.ts`
- `src/tools/labels.ts`
- `src/tools/cards.ts`
- `src/tools/actions.ts`

## Acceptance criteria

1. `npm run compile` проходит без ошибок
2. `grep -rn "= await fetch(" src/tools/` — пусто (кроме attachment download)
3. `grep -rn "apiKey && credentials" src/tools/` — пусто
4. Count tools не меняется

## Верификация

```bash
npm run compile
grep -rn "= await fetch(" src/tools/
grep -rn "apiKey && credentials" src/tools/
grep -rn "new URL(" src/tools/
```

## Какие документы обновить

- `ARCHITECTURE.md` — исправить блок про helpers: теперь реально используются
- `docs/CHANGELOG.md` — запись
