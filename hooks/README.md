# `.claude/hooks/`

Cross-platform git hooks, реализующие governance из `.claude/shards/`.

## pre-commit-triple-check

Pre-commit hook, который запускается при `git commit` и сканирует staged `.md` файлы на наличие factual claims без inline-источников.

| File | Platform |
|------|----------|
| `pre-commit-triple-check.sh` | bash (Linux / macOS / Git Bash on Windows) |
| `pre-commit-triple-check.ps1` | PowerShell (Windows native) |

> ⚠️ **Кодировка `pre-commit-triple-check.ps1`:** файл сохранён в **UTF-8 с BOM**. Windows PowerShell 5.1 читает UTF-8 без BOM как ANSI, что ломает кириллицу в regex и парсер. При редактировании сохраняйте BOM (`Out-File -Encoding utf8` в PS 5.1 добавляет BOM; в редакторе — "UTF-8 with BOM"). Bash-версия — обычный UTF-8 без BOM.

> Оба хука принимают `--advisory-single <file>` — проверка одного файла в advisory-режиме (используется Claude Code PostToolUse hook из `hooks.json`).

### Что проверяет

Для каждого staged `.md` файла:
1. Игнорирует YAML frontmatter, code blocks, `<!-- wiki:see-also-* -->` блоки, headings
2. Находит строки с числами / процентами / денежными суммами / x-ratios → это **claim'ы**
3. Для каждого claim проверяет наличие inline-ссылки на `adrs/`, `methodologies/`, `researches/`, или внешнего `http(s)://` URL
4. Считает MISSING (claim без источника) → выдаёт предупреждение

### Игнорируется

- `*/wiki-backlinks.md` (auto-generated)
- `wiki/concepts/*.md` (auto-generated)
- `.claude/**` (внутренний harness)
- YAML frontmatter, code blocks, wiki:see-also marker-блоки

### Режимы

| Env | Значение | Поведение |
|-----|----------|-----------|
| `TRIPLE_CHECK_MODE` | `advisory` (default) | Печатает warning, не блокирует commit |
| `TRIPLE_CHECK_MODE` | `strict` | Блокирует commit при MISSING > threshold |
| `TRIPLE_CHECK_MISSING_THRESHOLD` | `3` (default) | Порог для strict mode |

### Установка

**Вариант 1 (рекомендуется):** настроить git на использование hooks-папки из репо:

```bash
git config core.hooksPath .claude/hooks
```

После этого git автоматически найдёт `pre-commit-triple-check.sh` (если запущен под bash) или `.ps1` (если под PowerShell). Это работает только если файл назван **`pre-commit`** — поэтому переименуй / симлинкни:

```bash
# Linux / macOS / Git Bash:
ln -s pre-commit-triple-check.sh .claude/hooks/pre-commit
chmod +x .claude/hooks/pre-commit
```

```powershell
# PowerShell:
Copy-Item .claude\hooks\pre-commit-triple-check.ps1 .claude\hooks\pre-commit
```

**Вариант 2:** одноразовый запуск без установки:

```bash
.claude/hooks/pre-commit-triple-check.sh
```

```powershell
pwsh .claude\hooks\pre-commit-triple-check.ps1
```

### Pre-commit отказ

Если в commit'е содержательно НЕТ числовых claims без источника (всё подкреплено) — hook молчит и пропускает commit.

Если есть MISSING:
- **advisory mode**: печатает список проблемных мест, commit проходит
- **strict mode** + MISSING > threshold: commit блокируется

### Bypass (только для emergency!)

```bash
# Pre-commit hook можно обойти --no-verify, но это нарушает governance.
# Используй ТОЛЬКО когда понимаешь причину пропуска.
git commit --no-verify -m "..."
```

### Связано

- `.claude/commands/triple-check.md` — полная спецификация валидатора
- `.claude/shards/triple-pillar.shard.md` — governance shard, описывающий 3-pillar discipline
- `.claude/rules/inline-source-verification.md` — правила inline-ссылок
- `wiki/graph.json` `integrity` секция — машинный отчёт о coverage
