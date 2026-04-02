# Tasks

Эта папка хранит активные задачи и их архив.

## Структура

| Путь | Назначение |
| --- | --- |
| `./` | Активные задачи |
| `Done/` | Завершенные задачи |
| `Failed/` | Проваленные или отмененные задачи |

## Рекомендуемый формат имени

```text
TNN_short_snake_case_name.md
```

Примеры:

- `T01_sync_root_docs_with_code.md`
- `T02_restore_get_action_card_suite.md`

## Жизненный цикл задачи

1. Создать задачу в корне `3. tasks/`
2. Выполнить код и документацию
3. Обновить [../HANDOFF.md](../HANDOFF.md) и [../CHANGELOG.md](../CHANGELOG.md)
4. Переместить файл в `Done/` или `Failed/`

Для подробностей см. [../4. guides/task_decomposition_guide.md](../4.%20guides/task_decomposition_guide.md).
