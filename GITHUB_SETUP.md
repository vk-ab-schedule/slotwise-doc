# Настройка CI и обязательного ревью

Файлы workflow и CODEOWNERS хранятся в репозиториях. Обязательность проверок и одобрений включается отдельно в GitHub администратором. Наличие CODEOWNERS само по себе не блокирует merge.

## Подготовка

1. В организации откройте [Teams → Dream team](https://github.com/orgs/vk-ab-schedule/teams/dream-team).
2. Проверьте, что команда видимая (Visible), а не Secret.
3. На вкладке Repositories проверьте доступ команды к `slotwise-back`, `slotwise-front`, `slotwise-doc` и `slotwise-infra`: нужен как минимум Write именно у команды. При необходимости администратор должен назначить этот доступ.
4. Доставьте `.github/CODEOWNERS` в `main` и каждую действующую `sprint/N`, куда открываются PR. GitHub использует CODEOWNERS из целевой ветки PR. В пустом репозитории сначала нужен начальный коммит.
5. Доставьте workflow в backend и doc, откройте PR и дождитесь первого запуска проверок. Их названия после этого можно выбрать в настройках защиты.

Во всех четырёх репозиториях CODEOWNERS назначает `@vk-ab-schedule/dream-team` владельцем всех файлов, включая сам CODEOWNERS.

## Правило защиты веток

Повторите для каждого из четырёх репозиториев:

1. Откройте **Settings → Rules → Rulesets → New ruleset → New branch ruleset**.
2. Имя: `team-review`. **Enforcement status: Active**.
3. **Bypass list** оставьте пустым: Dream team не нужно добавлять в исключения.
4. В **Target branches → Add target → Include by pattern** добавьте `main` и `sprint/*`. Второй шаблон охватывает ветки спринтов, например `sprint/2`; в doc достаточно `main`.
5. Включите **Require a pull request before merging**:
   - **Required approvals: 1**;
   - **Require review from Code Owners**;
   - **Dismiss stale pull request approvals when new commits are pushed**;
   - **Require conversation resolution before merging**, если пункт находится здесь или в общем списке правил.
6. Включите **Block force pushes** и **Restrict deletions** для защищённых веток.
7. В backend и doc включите **Require status checks to pass**, затем **Add checks**:

   | Репозиторий | Обязательная проверка | Источник |
   | --- | --- | --- |
   | `slotwise-back` | `Backend tests` | GitHub Actions |
   | `slotwise-doc` | `Markdown lint` | GitHub Actions |

   В front и infra обязательных CI-проверок пока нет: оставьте этот пункт выключенным.

8. Нажмите **Create** и проверьте, что правило имеет статус Active.

Не включайте **Restrict updates**: это отдельный запрет обновлений веток. Не включайте **Require linear history**: проект использует merge-коммиты при выпуске спринта в main. Ветки задач не защищаем этим правилом, чтобы автор мог отправлять в них коммиты.

Одно одобрение участника Dream team удовлетворяет одновременно требованиям «1 approval» и «Code Owner review». Автор PR не может одобрить собственный PR. При новых изменениях кода ранее выданный approval сбрасывается, и требуется повторное ревью.

Если Rulesets недоступны, проверьте права администратора и тариф организации: для приватных репозиториев организации требуются возможности GitHub Team или Enterprise. Не считайте требование включённым, пока GitHub не показывает активное правило.

## Проверка настройки

Откройте обычный PR с небольшим изменением в защищённую ветку:

- GitHub запрашивает ревью Dream team (для draft PR запрос появится после Ready for review).
- До одобрения участником команды merge заблокирован.
- В backend/doc merge также заблокирован при упавшем обязательном CI.
- После одного одобрения и успешного CI merge разрешён, если нет конфликтов и незакрытых обсуждений.

## Текущие проверки CI

- Backend: каждый PR, push в `main` и `sprint/*`, ручной запуск. JDK 25, Gradle Wrapper, `./gradlew check`, PostgreSQL через Testcontainers. Отчёты доступны в артефакте `backend-test-reports` 7 дней.
- Документация: каждый PR, push в `main`, ручной запуск. Все Markdown-файлы проверяет `markdownlint-cli@0.49.1` с конфигурацией `.markdownlint.yaml`. Проверки длины строк и выравнивания столбцов таблиц отключены; остальные правила действуют.
- Деплой и публикация образов пока не настроены.

Локальная проверка документации (Node.js 24):

```bash
npx --yes markdownlint-cli@0.49.1 '**/*.md'
```

## Документация GitHub

- [CODEOWNERS: доступ команды и целевая ветка](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)
- [Правила защиты веток](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets)
