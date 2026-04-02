# Upstream Sync Guide

**Актуально на:** 2026-04-02

---

## Remote topology

В этом репозитории:

- `origin` = upstream `adriangrahldev/advanced-trello-mcp-server`
- `fork` = личный форк `Serebrennikovi/advanced-trello-mcp-server`

Это важно, потому что по привычке легко принять `origin` за свой форк, а здесь наоборот.

---

## Текущее расхождение

На 2026-04-02:

- `origin/main` = `ed0e1e6`
- `fork/main` = `1d33124`
- `fork/main` отстает от `origin/main` на 5 коммитов

Коммиты, которых нет в `fork/main`:

1. `f80b123` — due/start support PR branch
2. `26c3b0b` — merge PR #3
3. `ad74061` — merge PR #4 reliability + attachments
4. `241d438` — README refresh
5. `ed0e1e6` — README formatting refresh

---

## Что полезного есть в upstream

- reliability layer протянут через tool handlers
- attachment tools уже влиты
- README уже обновлен под `~35 tools`
- есть поддержка `due/start` в `create-card` и `create-cards`

### Важная оговорка

Merged `PR #3` исторически содержал `due/start` также и для `update-card`, но в более позднем `ad74061` эта часть фактически исчезла. Значит, sync с upstream полезен, но не решает эту часть автоматически.

---

## Рекомендация

Да, upstream стоит подтянуть в форк и локальную базовую ветку.

Причины:

1. `fork/main` сейчас не отражает фактическое accepted state upstream
2. на upstream уже есть полезные и принятые изменения, которые вы все равно учитываете локально
3. текущая документация и reasoning проще поддерживать относительно одного свежего baseline

---

## Предпочтительный порядок

1. Обновить локальные refs:

```bash
git fetch --all --prune
```

2. Переключиться на локальный `main`

3. Подтянуть в него `origin/main`

4. Отдельно решить, как синхронизировать `fork/main`

5. Только потом переносить локальные незалитые изменения поверх свежего baseline

---

## Что сравнивать после sync

- `src/tools/cards.ts`
- `src/utils/api.ts`
- `README.md`
- counts tool-ов
- не потерялись ли ваши локальные operational-файлы и правки

Если нужна историческая подложка, см. [../5. unsorted/2026-04-02_trello_mcp_history_and_tool_gap_analysis.md](../5.%20unsorted/2026-04-02_trello_mcp_history_and_tool_gap_analysis.md).
