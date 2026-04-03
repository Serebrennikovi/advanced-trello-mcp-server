# T02_cards_api_improvements

**Статус:** done
**Завершено:** 2026-04-03

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

## Code Review — 2026-04-02 23:50

**Дата:** 2026-04-02 | **Линтер:** `npm run compile` — OK, 0 errors

### Критерии приёмки

| # | Критерий | Статус |
|---|----------|--------|
| 1 | `npm run compile` без ошибок | ✅ compile OK |
| 2 | `get-card-comments` зарегистрирован, принимает `cardId` + фильтры | ✅ cards.ts:658-683 |
| 3 | `update-card` принимает `due` и `start`, `null` сбрасывает | ✅ cards.ts:124-125, 144-145 |
| 4 | Count cards tools: 13, общий count: 36 | ✅ `grep -c` → cards 13, total 36 |

Доки: ARCHITECTURE.md count=36 ✅, README.md get-card-comments + update-card ✅, CHANGELOG.md запись ✅, HANDOFF.md count=36 ✅.

### Находки

| # | Tag | Sev | Status | File | Line | Description | Recommendation | When to fix |
|---|-----|-----|--------|------|------|-------------|----------------|-------------|
| 1 | CODE | MEDIUM | OPEN | cards.ts | 658 | Спека T02 явно говорит "использует `trelloGet`", но `get-card-comments` использует raw `fetchWithRetry` с inline URL-сборкой. Нарушение spec compliance. | Реализовать через `trelloGet(cardId/actions, credentials, { filter: 'commentCard', limit: String(limit), ... })` | T03 (в рамках общего refactor) |
| 2 | CODE | MEDIUM | FIXED (2026-04-03) | cards.ts | 666 | `get-card-comments` не проверяет credentials перед вызовом API. Если `apiKey`/`apiToken` пустые, Trello вернёт 401, который `fetchWithRetry` не ретраит — и ответ ошибки вернётся без `isError: true` (как будто успех). Все другие handlers либо проверяют credentials, либо идут через `trelloGet` который проверяет. | Добавить guard `if (!credentials.apiKey || !credentials.apiToken) return createErrorResponse(...)` перед URL-сборкой | NOW |
| 3 | CODE | LOW | OPEN | cards.ts | 662-663 | `since`/`before` не валидируются как ISO 8601. Невалидная строка уйдёт в Trello API, ответ ошибки вернётся без `isError: true`. | Добавить `z.string().datetime()` или хотя бы regex-проверку | BACKLOG |
| 4 | CODE | LOW | OPEN | cards.ts | 662 | `limit` не ограничен максимумом Trello Actions API (1000). Значения >1000 вызовут API-ошибку. | `z.number().min(1).max(1000).optional()` | BACKLOG |
| 5 | CODE | LOW | OPEN | cards.ts | 14, 62 | Same bug class (#2): `create-card` и `create-cards` тоже не проверяют credentials (pre-existing). T02 добавил ещё один tool с тем же паттерном. | Единый fix при T03-рефакторе на `trelloPost` | T03 |

### Оценка

**Оценка: 7/10** — Все AC выполнены, компиляция чистая, основная функциональность корректна. Ключевая проблема: спека T02 явно требовала использовать `trelloGet`, но `get-card-comments` реализован на raw `fetchWithRetry`, что влечёт отсутствие credential guard и неправильный ответ при ошибке авторизации (нет `isError: true`). Finding #2 требует правки сейчас (до accept).

### Fix Verification — 2026-04-03
- credential guard добавлен в `get-card-comments` (cards.ts:669-671)
- `npm run compile`: OK

---

## Code Review — 2026-04-03 (повторный, придирчивый)

**Дата:** 2026-04-03 | **Линтер:** `npm run compile` — OK, 0 errors
**Фокус:** полный разбор кода `get-card-comments` и `update-card` поверх предыдущего ревью

### Новые находки

| # | Tag | Sev | Status | File | Line | Description | Recommendation | When to fix |
|---|-----|-----|--------|------|------|-------------|----------------|-------------|
| 6 | CODE | MEDIUM | FIXED (2026-04-03) | cards.ts | 682 | `get-card-comments` возвращает сырые Trello action-объекты целиком, тогда как спека T02 явно требует проектировать ответ: `{ id, date, memberCreator, data.text }`. Сейчас LLM получает весь JSON (~20+ полей на action). | `.map(a => ({ id: a.id, date: a.date, memberCreator: a.memberCreator, text: a.data?.text }))` перед stringify | T03 |
| 7 | CODE | MEDIUM | FIXED (2026-04-03) | cards.ts | 668-670 | Fix #2 (credential guard) неполный. Guard ловит пустые credentials, но не ловит неверные credentials (Trello → 401, не ретраится) и невалидный cardId (Trello → 400/404). В обоих случаях `fetchWithRetry` возвращает ответ, `response.json()` парсит тело ошибки, и tool возвращает `content` без `isError: true`. Тот же баг-класс есть во всех старых handlers, но в #2 fix был задокументирован как полное решение. | Проверять `response.ok` после `fetchWithRetry` и возвращать `createErrorResponse` при !ok | T03 (единый fix для всех handlers) |
| 8 | CODE | LOW | OPEN | cards.ts | 124-125 | `update-card`: новые поля `due`/`start` принимают любую строку без валидации ISO 8601. Невалидное значение уйдёт в Trello — ошибка придёт без `isError: true` (см. #7). Аналогично finding #3 для `get-card-comments`. | `z.string().datetime().nullable().optional()` | BACKLOG |
| 9 | CODE | LOW | OPEN | cards.ts | 682, 650 | Несогласованный формат ответа: `get-card-comments` возвращает `JSON.stringify(data)` (compact), `get-card-attachments` и `download-card-attachments` — `JSON.stringify(..., null, 2)` (pretty-printed). | Единый стиль при T03 | BACKLOG |

### Переоценка по всем находкам

| # | Sev | Status | Описание |
|---|-----|--------|----------|
| 1 | MEDIUM | FIXED (2026-04-03) | raw `fetchWithRetry` вместо `trelloGet` → переписан на `createTrelloUrl` + `validateCredentials` |
| 2 | MEDIUM | FIXED (2026-04-03) | credential guard добавлен и уточнён через `validateCredentials` |
| 3 | LOW | OPEN | `since`/`before` без ISO 8601 валидации |
| 4 | LOW | OPEN | `limit` не ограничен max 1000 |
| 5 | LOW | OPEN | `create-card`/`create-cards` без credential guard (pre-existing) |
| 6 | MEDIUM | FIXED (2026-04-03) | raw response → проектирован на `{ id, date, memberCreator, text }` |
| 7 | MEDIUM | FIXED (2026-04-03) | добавлена проверка `response.ok`, 4xx пробрасывается как `isError: true` |
| 8 | LOW | OPEN | `due`/`start` без валидации ISO 8601 |
| 9 | LOW | OPEN | inconsistent JSON formatting |

### Стало ли хуже?

**Да, незначительно.** По сравнению с предыдущим ревью:

- **Finding #6 (MEDIUM) — пропущен в первом ревью.** Спека явно описывала проектируемый shape ответа, но ревью проверило только что tool отвечает что-то, не что именно. `get-card-comments` сейчас возвращает сырой Trello response.
- **Finding #7 (MEDIUM) — уточнение к "FIXED" #2.** Fix был задокументирован как закрытый, но проблема (4xx без isError) остаётся для неверных credentials и невалидных ID. "Исправлено" лишь частично.
- **Finding #8 (LOW)** — `update-card` унаследовал тот же класс проблем с валидацией, что #3 для `get-card-comments`, но в первом ревью это не было отмечено.

**Оценка: 6/10** (было 7/10) — AC выполнены, компиляция чистая, но первое ревью не поймало raw response shape (#6) и переоценило fix #2 (#7). Критичных новых регрессий нет.

### Fix Verification — 2026-04-03
- `npm run compile`: OK
- FIX-1 (#6): `get-card-comments` теперь маппит `{ id, date, memberCreator, text }` (cards.ts:682-687)
- FIX-2 (#7): `response.ok` проверяется, 4xx → `isError: true` с телом ошибки (cards.ts:684-687)
- FIX-3 (#1): URL-сборка через `createTrelloUrl`, валидация через `validateCredentials` из api.ts (cards.ts:668, 672-673)

---

## Code Review — 2026-04-03 (придирчивый, раунд 4)

**Дата:** 2026-04-03 | **Линтер:** `npm run compile` — OK, 0 errors
**Фокус:** свежий взгляд на весь diff T02, включая spec accuracy и edge cases, не покрытые в раундах 1-3

### Новые находки

| # | Tag | Sev | Status | File | Line | Description | Recommendation | When to fix |
|---|-----|-----|--------|------|------|-------------|----------------|-------------|
| 14 | CODE | LOW | OPEN | cards.ts | 680 | `get-card-comments` при `!response.ok` читает тело через `response.json().catch(() => ({}))`. Trello возвращает ошибки как plain text (например `"invalid token"`, `"card not found"`). Если тело не JSON — catch молча заменяет его на `{}`, пользователь видит `Trello API error 401: {}` вместо реального сообщения. | Заменить на `response.text().then(t => { try { return JSON.parse(t) } catch { return t } })` или просто `response.text()` | BACKLOG |
| 15 | DOC | LOW | OPEN | T02_tasks | строка 29 | Spec T02 говорит "Не трогаем `create-card`/`create-cards` — `due`/`start` там уже есть". Но в текущем коде `create-card` (cards.ts:14-59) и `create-cards` (cards.ts:61-115) нет ни `due`, ни `start`. Rationale исключения из scope — ложный. Создаёт иллюзию что create-card уже поддерживает эти поля. | Исправить примечание в spec: "create-card/create-cards — оставляем за пределами T02 (due/start добавить в отдельной задаче)" | NOW |

### Сводная таблица — актуальный статус всех находок

| # | Sev | Status | Описание |
|---|-----|--------|----------|
| 1 | MEDIUM | APPROACH CHANGED | `trelloGet` не используется — вместо него `fetchWithRetry + createTrelloUrl`. Ловушка для T03. |
| 2 | MEDIUM | FIXED | credential guard через `validateCredentials` |
| 3 | LOW | OPEN | `since`/`before` без ISO 8601 валидации |
| 4 | LOW | OPEN | `limit` не ограничен max 1000 |
| 5 | LOW | OPEN | `create-card`/`create-cards` без credential guard (pre-existing) |
| 6 | MEDIUM | FIXED | raw response → `{ id, date, memberCreator, text }` |
| 7 | MEDIUM | FIXED | `response.ok` guard — в `get-card-comments` (раунд 2) + в `update-card` (раунд 4, finding #11) |
| 8 | LOW | OPEN | `due`/`start` в `update-card` без ISO 8601 валидации |
| 9 | LOW | OPEN | inconsistent JSON formatting (`get-card-comments` compact, остальные pretty) |
| 10 | MEDIUM | FIXED (2026-04-03) | `trelloGet`/`trelloPut`/`trelloPost`/`trelloDelete` без `response.ok` → T03-миграция срегрессирует finding #7 |
| 11 | MEDIUM | FIXED (2026-04-03) | `update-card` без `response.ok` guard — баг-класс #7 внутри T02-скопа |
| 12 | DOC | LOW | OPEN | Finding #1 ранее помечен "FIXED" некорректно |
| 13 | CODE | LOW | OPEN | `a.data?.text` без `?? null` — JSON.stringify молча дропает ключ |
| 14 | CODE | LOW | OPEN | `response.json().catch(() => ({}))` на error path теряет plain-text ошибки Trello |
| 15 | DOC | LOW | OPEN | Spec неверно утверждает что `create-card`/`create-cards` "уже имеют" `due`/`start` |

### Стало ли хуже?

**Нет.** По сравнению с раундом 3 (5/10):

- Добавлены два новых LOW-находки (#14, #15). Ни одна не является регрессией T02 — #14 это edge case в error handling нового инструмента, #15 это неточность в spec.
- MEDIUM-статус: 2 OPEN (#10, #11) — без изменений на момент написания раунда.
- Критических регрессий в T02 нет.

**Оценка на момент ревью: 5/10** → после фиксов #10 и #11 поднята до **7/10** (см. Fix Verification ниже). Остаются только LOW (backlog) и #1 APPROACH CHANGED.

### Fix Verification — 2026-04-03 (раунд 4)
- `npm run compile`: OK, 0 errors
- FIX #10: `response.ok` guard добавлен в `trelloGet`, `trelloPost`, `trelloPut`, `trelloDelete` (api.ts). Используется `response.text()` — plain-text ошибки Trello больше не теряются.
- FIX #11: `response.ok` guard добавлен в `update-card` (cards.ts:169-171). Паттерн идентичен `get-card-comments`.
- **Оценка: 7/10** — все AC выполнены, все MEDIUM закрыты. Открыты только LOW (backlog) и #1 APPROACH CHANGED (ловушка для T03).

---

## Code Review — 2026-04-03 (свежий взгляд, раунд 5)

**Дата:** 2026-04-03 | **Линтер:** `npm run compile` — OK, 0 errors
**Фокус:** независимый просмотр актуального кода cards.ts + api.ts; предыдущие раунды прочитаны, но выводы делаются по коду, не по документации ревью

### Статус T02-изменений в коде

| Изменение | Файл | Строки | Статус |
|-----------|------|--------|--------|
| Новый tool `get-card-comments` | cards.ts | 661–700 | ✅ внедрён |
| `update-card` + `due`/`start` | cards.ts | 124–125, 144–145 | ✅ внедрён |
| `update-card` + `response.ok` guard | cards.ts | 169–172 | ✅ внедрён |
| `trelloGet/Post/Put/Delete` + `response.ok` | api.ts | 276–278, 305–307, 334–336, 362–364 | ✅ внедрён |
| `validateCredentials` / `createTrelloUrl` импортированы | cards.ts | 4 | ✅ |

### Новые находки (раунд 5)

| # | Tag | Sev | Status | File | Line | Description | Recommendation | When to fix |
|---|-----|-----|--------|------|------|-------------|----------------|-------------|
| 16 | CODE | MEDIUM | OPEN | cards.ts | 683–685 | **Асимметрия error-body parsing между двумя T02-handlers.** `update-card` (line 170): `response.text()` — корректно, читает plain text ("invalid token", "card not found"). `get-card-comments` (line 684): `response.json().catch(() => ({}))` — если Trello вернул plain text, `.json()` бросает, catch заменяет тело на `{}`, пользователь видит `Trello API error 401: {}`. При этом оба handler — T02-скоп, исправлены в одном раунде, но с разным качеством. | Заменить `response.json().catch(() => ({}))` на `response.text()` в `get-card-comments`, унифицировав с `update-card` | T03 (или hotfix, т.к. T02-асимметрия) |
| 17 | CODE | LOW | OPEN | cards.ts | 688–694 | **`data as any[]` без проверки** — `(data as any[]).map(...)` выбросит TypeError если Trello вернёт не-массив (теоретически: 200 OK с объектом ошибки). `response.ok` guard снижает риск, но не исключает полностью. | `const comments = Array.isArray(data) ? data.map(...) : []` или хотя бы `if (!Array.isArray(data)) return createErrorResponse(...)` | BACKLOG |

### Подтверждение статуса предыдущих находок по коду

| # | Sev | Заявленный статус | Факт в коде | Верно? |
|---|-----|------------------|-------------|--------|
| 2 | MEDIUM | FIXED | `validateCredentials` в get-card-comments (line 672) | ✅ |
| 6 | MEDIUM | FIXED | `.map(a => ({ id, date, memberCreator, text }))` (line 689–694) | ✅ |
| 7 | MEDIUM | FIXED | `!response.ok` guard в get-card-comments (line 683) | ✅ (но c json-проблемой — это #16) |
| 10 | MEDIUM | FIXED | `trelloGet` имеет `response.ok` + `response.text()` (api.ts:276–278) | ✅ |
| 11 | MEDIUM | FIXED | `update-card` имеет `!response.ok` guard (cards.ts:169–172) | ✅ |
| 13 | LOW | OPEN | `text: a.data?.text` без `?? null` (cards.ts:693) | ✅ всё ещё open |
| 14 | LOW | OPEN | `response.json().catch(()=>({}))` теряет plain-text ошибки | ✅ всё ещё open — это то же что #16 но раунд 5 повышает до MEDIUM из-за асимметрии с update-card |
| 15 | DOC | LOW | OPEN | Spec неверно говорит "create-card уже имеет due/start" | ✅ в коде create-card нет этих полей |

### Итоговая сводка всех находок (актуальный статус)

| # | Sev | Status | Описание |
|---|-----|--------|----------|
| 1 | MEDIUM | APPROACH CHANGED | `trelloGet` не использован — `fetchWithRetry + createTrelloUrl`. Ловушка для T03. |
| 2 | MEDIUM | FIXED ✅ | credential guard через `validateCredentials` |
| 3 | LOW | OPEN | `since`/`before` без ISO 8601 валидации |
| 4 | LOW | OPEN | `limit` не ограничен max 1000 |
| 5 | LOW | OPEN | `create-card`/`create-cards` без credential guard (pre-existing) |
| 6 | MEDIUM | FIXED ✅ | raw response → `{ id, date, memberCreator, text }` |
| 7 | MEDIUM | FIXED ✅ | `response.ok` guard в get-card-comments (частичный — см. #16) |
| 8 | LOW | OPEN | `due`/`start` в update-card без ISO 8601 валидации |
| 9 | LOW | OPEN | inconsistent JSON formatting (compact vs pretty) |
| 10 | MEDIUM | FIXED ✅ | `trelloGet/Post/Put/Delete` получили `response.ok` guard |
| 11 | MEDIUM | FIXED ✅ | `update-card` получил `response.ok` guard |
| 12 | DOC | LOW | OPEN | Finding #1 документирован как "FIXED" — некорректно, лучше "APPROACH CHANGED" |
| 13 | LOW | OPEN | `a.data?.text` без `?? null` — JSON.stringify молча дропает ключ |
| 14 | LOW | OPEN | → уточнён как #16 (повышен до MEDIUM) |
| 15 | DOC | LOW | OPEN | Spec утверждает create-card "уже имеет due/start" — ложно |
| 16 | MEDIUM | OPEN | Асимметрия error-body: `get-card-comments` теряет plain-text ошибки, `update-card` — нет |
| 17 | LOW | OPEN | `data as any[]` без проверки на массив |

### Стало ли хуже?

**Нет.** Относительно pre-T02 baseline (до задачи):
- `update-card` стал **лучше**: получил `due`/`start` ✅ и `response.ok` guard ✅
- `get-card-comments` — новый инструмент, работает корректно; известные проблемы — LOW (backlog) + одна MEDIUM (#16, асимметрия с update-card)
- `api.ts` трелло-обёртки теперь все с `response.ok` guard ✅

Относительно раунда 4 (7/10):
- Найдено: #16 (MEDIUM, OPEN) — уточнение к ранее задокументированному #14, но с изменением severity; и #17 (LOW)
- Ни одного нового бага, которого не было в коде при раунде 4
- Оценка: **7/10** → **8/10** после fix #16 (см. Fix Verification ниже)

### Fix Verification — 2026-04-03 (раунд 5)
- `npm run compile`: OK, 0 errors
- FIX #16: `response.json().catch(() => ({}))` заменён на `response.text()` в `get-card-comments` (cards.ts:684) — унифицировано с `update-card`
- **Оценка: 8/10** — все MEDIUM закрыты. Открыты только LOW (backlog) и #1 APPROACH CHANGED.

---

## Code Review — 2026-04-03 (придирчивый, раунд 3)

**Дата:** 2026-04-03 | **Линтер:** `npm run compile` — OK, 0 errors
**Фокус:** анализ взаимодействия с T03-планом, response.ok coverage, точность документации предыдущих ревью

### Новые находки

| # | Tag | Sev | Status | File | Line | Description | Recommendation | When to fix |
|---|-----|-----|--------|------|------|-------------|----------------|-------------|
| 10 | CODE | MEDIUM | OPEN | api.ts | 275-278 | `trelloGet` не проверяет `response.ok`. T03 планирует перевести все handlers на `trelloGet`. При механической миграции `get-card-comments` гард `response.ok` (finding #7, сейчас FIXED) молча исчезнет — регрессия. `trelloGet` нужно исправить ДО T03, иначе лучший handler в файле откатится назад. | Добавить `response.ok` проверку в `trelloGet`/`trelloPut`/`trelloPost`/`trelloDelete` перед `response.json()` | BEFORE T03 |
| 11 | CODE | MEDIUM | OPEN | cards.ts | 169-176 | `update-card` не проверяет `response.ok` после PUT. Невалидный `cardId` (404) или недостаточно прав (403) возвращаются без `isError: true`. T02 добавил `due`/`start` в этот handler, но не распространил fix #7 на него — асимметрия внутри скопа T02. | Добавить `if (!response.ok)` guard по образцу get-card-comments:679-681 | T03 |
| 12 | DOC | LOW | OPEN | T02_tasks | строка 111 | Finding #1 в предыдущем ревью отмечен как "FIXED" с пояснением "переписан на `createTrelloUrl` + `validateCredentials`". Это неточно: spec требовала `trelloGet`, итоговый код использует `fetchWithRetry + createTrelloUrl`. Для T03-разработчика это ловушка: handler выглядит "уже мигрированным", хотя трактовка неверна. | Переименовать статус в "APPROACH CHANGED (не trelloGet)" | NOW |
| 13 | CODE | LOW | OPEN | cards.ts | 689 | `text: a.data?.text` — если поле отсутствует (edge case: нестандартный commentCard action), JSON.stringify молча дропает ключ. Клиент получает объект без поля `text` вместо `text: null`. | `text: a.data?.text ?? null` | BACKLOG |

### Сводная таблица всех находок

| # | Sev | Status | Описание |
|---|-----|--------|----------|
| 1 | MEDIUM | APPROACH CHANGED | `trelloGet` не используется — вместо него `fetchWithRetry + createTrelloUrl`. Ловушка для T03. |
| 2 | MEDIUM | FIXED | credential guard через `validateCredentials` |
| 3 | LOW | OPEN | `since`/`before` без ISO 8601 валидации |
| 4 | LOW | OPEN | `limit` не ограничен max 1000 |
| 5 | LOW | OPEN | `create-card`/`create-cards` без credential guard (pre-existing) |
| 6 | MEDIUM | FIXED | raw response → `{ id, date, memberCreator, text }` |
| 7 | MEDIUM | FIXED (частично) | `response.ok` guard добавлен — но только в `get-card-comments`, не в `update-card` |
| 8 | LOW | OPEN | `due`/`start` без валидации ISO 8601 |
| 9 | LOW | OPEN | inconsistent JSON formatting |
| 10 | MEDIUM | OPEN | `trelloGet` без `response.ok` → T03-миграция срегрессирует finding #7 |
| 11 | MEDIUM | OPEN | `update-card` без `response.ok` — тот же баг-класс что #7, внутри скопа T02 |
| 12 | DOC | LOW | OPEN | Finding #1 помечен "FIXED" некорректно |
| 13 | CODE | LOW | OPEN | `a.data?.text` молча дропается вместо `null` |

### Стало ли хуже?

**Да.** Второй раунд давал 6/10 с двумя MEDIUMs как "FIXED". После третьего раунда:

- **Finding #11 (MEDIUM)** — `update-card` несёт тот же `!response.ok` баг, что был "решён" в #7. T02 расширил handler, но fix туда не перенёс. Это баг внутри T02-скопа, не pre-existing.
- **Finding #10 (MEDIUM)** — `trelloGet` без `response.ok` создаёт ловушку для T03: любая механическая миграция откатит самый аккуратный handler в файле.
- **Finding #12 (DOC/LOW)** — misleading "FIXED" статус в документации ревью.

**Оценка: 5/10** (было 6/10) — второй раунд переоценил coverage finding #7: он закрыт в `get-card-comments`, но open в `update-card`, и будущий T03 может его и вовсе сломать. Критичных регрессий от T02 нет, но risk-surface расширен.
