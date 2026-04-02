# Trello MCP: история `44 tools`, gap analysis и upstream drift

**Дата:** 2026-04-02

---

## Краткий вывод

`44 tools` в истории проекта были реальными. Это был не маркетинговый текст и не ошибка памяти, а фактический набор инструментов в монолитном `src/index.ts`.

Проблема в другом:

- во время модульного рефактора часть tool-ов не была перенесена
- документация при этом осталась с историческим count `44`
- позже появились новые card-tools, и счетчик вырос до `35`, но потерянные `12` action-tools так и не вернулись

---

## 1. Откуда взялись `44`

Хронология по git:

| Дата | Commit | Что произошло | Count |
| --- | --- | --- | --- |
| 2025-06-24 | `12b3fa8` | большой апгрейд относительно исходного open-source сервера | `40` |
| 2025-06-24 | `aa629f3` | добавлены 4 label-tools | `44` |
| 2025-06-24 | `dca357f` | `README.md` обновлен под `44` | `44` |
| 2025-06-24 | `b6afe55` | модульный рефактор, создан `src/index.original.ts` | фактически `32` |
| 2026-02-07 | `1d33124` | добавлен `update-card` | `33` |
| 2026-03-18 | `ad74061` | reliability + attachments в upstream | `35` |

Ключевой факт:

- файл [../../src/index.original.ts](../../src/index.original.ts) до сих пор хранит старый монолит с набором из `44 tools`

---

## 2. Что было в исторических `44`

Исторический монолит включал:

- boards: `1`
- lists: `10`
- cards: `11`
- labels: `8`
- actions: `14`

Итого: `44`.

Список был не “примерным”, а буквальным.

---

## 3. Что стало после модульного рефактора

Коммит `b6afe55` объявил модульную архитектуру и сохранил монолит в `src/index.original.ts`, но фактический перенос оказался неполным.

Сразу после рефактора в модульных файлах осталось:

- boards: `1`
- lists: `10`
- cards: `10`
- labels: `8`
- actions: `4`

Итого: `32`.

То есть count упал не потому, что API осознанно упростили, а потому что значительная часть action-surface не переехала в `src/tools/actions.ts`.

---

## 4. Какие tool-ы были отрезаны относительно `44`

Ниже список потерянных инструментов и практический смысл потери.

| Tool | Endpoint | Что умел | Что потеряли |
| --- | --- | --- | --- |
| `get-action-field` | `GET /actions/{id}/{field}` | забирал отдельное поле action без полного payload | более точный action introspection |
| `get-action-board` | `GET /actions/{id}/board` | получал board, связанный с action | переход от action к board |
| `get-action-card` | `GET /actions/{id}/card` | получал card, связанную с action | переход от action к card |
| `get-action-list` | `GET /actions/{id}/list` | получал list, связанную с action | переход от action к list |
| `get-action-member` | `GET /actions/{id}/member` | получал member для action | доступ к actor/member |
| `get-action-member-creator` | `GET /actions/{id}/memberCreator` | получал автора action | audit trail и attribution |
| `get-action-organization` | `GET /actions/{id}/organization` | получал организацию action | org context |
| `update-comment-action` | `PUT /actions/{id}/text` | обновлял comment action через dedicated endpoint | отдельный wrapper под comment text update |
| `create-action-reaction` | `POST /actions/{id}/reactions` | ставил reaction на action/comment | reaction write path |
| `get-action-reaction` | `GET /actions/{id}/reactions/{reactionId}` | читал конкретную reaction | reaction detail lookup |
| `delete-action-reaction` | `DELETE /actions/{id}/reactions/{reactionId}` | удалял reaction | reaction delete path |
| `get-action-reactions-summary` | `GET /actions/{id}/reactionsSummary` | агрегировал reaction summary | summary по emoji/reactions |

### Что это значит по сути

Потеря не ограничилась “минус 12 названий”.

Проект лишился почти всего расширенного action-graph:

1. traversal от action к связанным сущностям
2. fine-grained introspection по action fields
3. reaction CRUD и reaction summary
4. части comment-specific action API

### Важная оговорка про `update-comment-action`

Этот tool не полностью уникален по смыслу.

Почему:

- в историческом наборе уже был `update-action`
- `update-comment-action` был более узким wrapper под endpoint `PUT /actions/{id}/text`

То есть из 12 потерянных tool-ов минимум 1 имел частичное пересечение с тем, что осталось. Но остальные 11 дают реально отдельную capability.

---

## 5. Что добавилось позже

Относительно исторических `44` текущий код получил 3 новых tool-а:

| Tool | Откуда появился | Что делает |
| --- | --- | --- |
| `update-card` | `1d33124` | обновляет name/description карточки |
| `get-card-attachments` | март 2026 | читает attachments и маппит их к comment context |
| `download-card-attachments` | март 2026 | скачивает attachment files на диск и пишет `_manifest.json` |

Поэтому формула сейчас такая:

- было `44`
- потеряли `12`
- добавили `3`
- итог `35`

---

## 6. Общие хелперы рядом с tool-set

В документации про “модульную архитектуру” фигурирует идея общих API-хелперов.

### Что реально есть

В [../../src/utils/api.ts](../../src/utils/api.ts) существуют:

- `fetchWithRetry`
- `createSuccessResponse`
- `createErrorResponse`
- `validateCredentials`
- `createTrelloUrl`
- `trelloGet`
- `trelloPost`
- `trelloPut`
- `trelloDelete`

### Что реально используется

Фактическая картина такая:

- `fetchWithRetry` используется активно и повсеместно
- более высокоуровневые helpers `trelloGet/trelloPost/trelloPut/trelloDelete` почти не являются основным способом написания tool-handlers
- многие handlers по-прежнему вручную делают:
  - проверку credentials
  - сборку URL
  - `response.json()`
  - стандартный формат success/error ответа

### Почему это важно рядом с разбором tool-ов

Потому что `ARCHITECTURE.md` создает впечатление, что рефактор не только модульный, но и системно убрал дублирование через общие helpers.

По факту:

- модульность есть
- надежный низкоуровневый HTTP helper есть
- полная унификация handler-ов через общие helpers не завершена

Это не баг документации уровня “не тот count”. Это более глубокий mismatch между описанным и фактическим состоянием кодовой базы.

---

## 7. Что нового есть в upstream относительно нас

Сравнение на 2026-04-02:

- `fork/main` = `1d33124`
- `origin/main` = `ed0e1e6`

`origin/main` впереди на 5 коммитов:

1. `f80b123` — ветка с `due/start` support
2. `26c3b0b` — merge PR #3
3. `ad74061` — merge PR #4 reliability + attachments
4. `241d438` — README refresh
5. `ed0e1e6` — README formatting refresh

### Что полезного реально есть в текущем upstream

- reliability layer проведен через tool handlers
- attachment tools уже присутствуют
- README уже переписан под актуальный count `~35 tools`
- в `create-card` и `create-cards` добавлены `due` и `start`

### Важная тонкость

Merged PR #3 содержал `due/start` также для `update-card`, но в более позднем `ad74061` этот кусок фактически исчез.

Иными словами:

- история GitHub говорит “PR #3 merged”
- текущее состояние `origin/main` не сохраняет всю функциональность, которую нес merge commit `26c3b0b`

### Стоит ли это тянуть в форк и локаль

Да.

Причины:

1. `fork/main` отстает от фактического accepted state upstream
2. upstream уже содержит принятые и полезные вещи, которые нет смысла держать только как локальный side branch
3. проще поддерживать дальнейшую документацию от одного свежего baseline

Но тянуть надо осознанно:

- сравнить `src/tools/cards.ts`
- не потерять локальные operational-изменения
- отдельно решить, хотите ли вы вернуть `due/start` в `update-card`

---

## 8. Позволяют ли исторические `44 tools` смотреть комментарии по конкретной карточке

Короткий ответ: **напрямую нет**.

### Что в `44` для этого было

Были:

- `add-comment`
- `add-comments`
- `get-action`
- `get-action-field`
- `get-action-card`
- `update-action`
- `delete-action`
- `update-comment-action`

### Чего не было

Не было tool-а вида:

- `get-card-actions`
- `get-card-comments`
- `get-card-comment-actions`

То есть historical `44` не давали прямого пути:

`cardId -> список comment actions этой карточки`

### Что это означает practically

Через `44 tools` вы могли:

- создать comment на карточке
- обновить или удалить action/comment, если уже знаете `actionId`
- смотреть action detail, если уже знаете `actionId`

Но вы **не могли просто спросить сервер**:

> покажи все комментарии карточки `cardId=...`

Для этого в наборе не хватало list/read tool-а по card actions.

### Какой tool здесь реально нужен

Минимально полезный missing tool:

- `get-card-comments`

Реализация через Trello:

```text
GET /1/cards/{cardId}/actions?filter=commentCard
```

Опциональные параметры:

- `limit`
- `since`
- `before`
- `memberCreator`
- `fields`

### Почему его удобно добавить именно сейчас

Потому что текущий код уже использует этот endpoint внутри attachment workflow:

- `get-card-attachments`
- `download-card-attachments`

То есть сервер уже умеет ходить в:

```text
/1/cards/{cardId}/actions?filter=commentCard
```

Просто эта возможность не вынесена в отдельный MCP tool.

---

## 9. Практические рекомендации

### Если цель — восстановить исторический coverage

Приоритет на возврат:

1. `get-action-card`
2. `get-action-member-creator`
3. `create-action-reaction`
4. `delete-action-reaction`
5. `get-action-reactions-summary`

Именно они дают наиболее ощутимый возврат audit/comment workflow.

### Если цель — полезность для агентов сегодня

Приоритет другой:

1. добавить `get-card-comments`
2. решить вопрос с `due/start` в `update-card`
3. синхронизировать `ARCHITECTURE.md` и `README.md` с кодом
4. при необходимости вернуть только часть action-tools, а не все 12 сразу

---

## Итог

Главный вывод не в том, что “документация наврала”.

Главный вывод в том, что проект пережил два разных состояния:

1. богатый монолитный action-heavy сервер на `44 tools`
2. более надежный, но функционально урезанный модульный сервер на `35 tools`

Если это осознанный trade-off, его нужно честно описать в документации.  
Если нет, то список выше — это конкретный backlog на восстановление capability, а не абстрактная “рассинхронизация”.
