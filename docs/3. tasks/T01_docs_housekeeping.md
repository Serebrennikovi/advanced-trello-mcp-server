# T01_docs_housekeeping

## Контекст

Папка `docs/` создана 2026-04-02 и не закоммичена. `ARCHITECTURE.md` и `README.md` содержат устаревшие claims: count 44, несуществующий `npm run dev`, неверное описание `index.ts` и helpers.

Спека: [../2. specifications/S01_gap_closure.md](../2.%20specifications/S01_gap_closure.md)

## Что делаем

1. Закоммитить `docs/` в ветку `feat/reliability-batch-attachments`
2. Обновить `ARCHITECTURE.md`:
   - count → **35** (текущее до выполнения T02–T04)
   - убрать `npm run dev`
   - исправить описание `index.ts` — он регистрирует 3 MCP resources и делает API вызовы
   - исправить блок про helpers — `fetchWithRetry` используется везде, `trelloGet/Post/Put/Delete` — частично, boilerplate дублируется
   - убрать или сделать честным блок про testing
3. Обновить `README.md` под актуальный count и список tools

## Что не делаем

- Не обновляем counts под будущие задачи — только текущее состояние
- Не переписываем документы с нуля

## Файлы / модули

- `docs/` — коммит
- `ARCHITECTURE.md`
- `README.md`

## Acceptance criteria

1. `git status` не показывает `docs/` как untracked
2. `ARCHITECTURE.md` не содержит `44` и `npm run dev`
3. `README.md` отражает актуальный список tools

## Верификация

```bash
git status
grep "44\|npm run dev" ARCHITECTURE.md
```

## Какие документы обновить

- `docs/CHANGELOG.md` — запись
