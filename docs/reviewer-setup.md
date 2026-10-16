# Агент-ревʼюер: налаштування й робота

Ревʼюер працює як GitHub Action на базі
[anthropics/claude-code-action](https://github.com/anthropics/claude-code-action).
На кожному PR, виставленому в Ready for review, він читає діф, запускає
`go build/vet/test`, звіряє Definition of Done і лишає ревʼю українською.
Код він не змінює.

## Файли

| Файл | Призначення |
|---|---|
| `.github/workflows/claude-review.yml` | Автоматичне ревʼю PR |
| `.github/workflows/claude-mention.yml` | Відповіді на `@claude` у коментарях (лише читання) |
| `.github/review/REVIEWER.md` | Правила ревʼю: що перевіряти, стиль, формат, межі |
| `.github/pull_request_template.md` | Шаблон PR із Definition of Done |

Щоб змінити поведінку ревʼюера, правиш `REVIEWER.md`, а не workflow.

## Одноразове налаштування

1. Встанови Claude GitHub App для репозиторію (https://github.com/apps/claude)
   або запусти в Claude Code команду `/install-github-app`, яка проведе
   через налаштування.
2. Додай секрет `ANTHROPIC_API_KEY`: Settings → Secrets and variables →
   Actions → New repository secret. Ключ береться в Anthropic Console, а
   оплата йде за токени окремо від підписки claude.ai.
3. У Anthropic Console виставь ліміт витрат (spend limit) для цього ключа.
4. Скопіюй файли з цього пакета в репозиторій, зроби перший PR.
5. Branch protection (опційно): вимагай зелений CI (`go vet`, lint, test)
   перед мерджем. Ревʼю агента радше дорадче, його статус обовʼязковим
   краще не робити.

## Робочий процес

1. Береш задачу в Linear, створюєш гілку через Copy git branch name.
2. Відкриваєш PR як Draft. Ревʼюер у цей час мовчить і токени не витрачаються.
3. Коли готово, натискаєш Ready for review. Ревʼюер лишає один підсумковий
   коментар (він оновлюється, а не дублюється) та inline-зауваження.
4. Відповідаєш на питання й уточнюєш через `@claude ...` у коментарі.
5. Виправляєш і робиш push. Ревʼю запуститься знову.
6. Мерджиш, коли вердикт ✅ і CI зелений.

## Безпека

- Workflow запускається лише для PR із цього ж репозиторію (не з форків) і
  лише від власника. Це захищає секрет від чужого коду й prompt injection.
- Не переходь на тригер `pull_request_target`: він віддає секрети коду з
  PR.
- Ревʼюер запускає `go test` над кодом PR. Оскільки PR відкриваєш ти сам,
  це прийнятно. Якщо колись додаси сторонніх контриб'юторів, прибери
  `Bash(go ...)` з `--allowedTools`.
- Режим `@claude` має заборону на Edit/Write: він може лише пояснювати.

## Перша перевірка після налаштування

Після першого PR звір фактичну поведінку, бо action у бета-статусі:

- чи зʼявився один sticky-коментар, а не кілька;
- чи працюють inline-коментарі (інструмент `create_inline_comment`);
- чи ревʼюер реально запустив `go test` (якщо `go.mod` уже є);
- скільки токенів витратилося (Anthropic Console → Usage).

Якщо щось не так, спершу дивись лог кроку `Claude Code Action` у вкладці
Actions, потім
[FAQ action](https://github.com/anthropics/claude-code-action/blob/main/docs/faq.md).

## Що можна додати пізніше

- Підтягування критеріїв задачі напряму з Linear через MCP (зараз
  ревʼюер бере DoD з опису PR).
- Окремий workflow для глибшого ревʼю фази 2 (конкурентність, race
  detector, goroutine leaks) за фільтром `paths:`.
- Автоматична зміна статусу задачі в Linear після мерджу через
  інтеграцію Linear і GitHub.
