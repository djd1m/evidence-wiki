# llm-wiki-harness

Claude Code plugin: **LLM-Wiki генератор с 3-pillar evidence discipline**.

**Версия:** 0.1.0 · **Лицензия:** MIT

## Что делает

1. **LLM-Wiki** — строит слой атомарных self-contained концепт-страниц поверх вашей документации (концепция Karpathy: каждая страница читается без контекста, явный граф связей, JSON-индекс для LLM resolver).
2. **3-pillar discipline** — гарантирует, что каждый факт опирается на 3 типа источников (по умолчанию: ADR / methodology / research — конфигурируемо).
3. **Автоматизация** — команды `/wiki-generate` и `/triple-check`, governance shard, git pre-commit хуки.

## Состав плагина

```
llm-wiki-harness/
├── .claude-plugin/plugin.json        — манифест + config
├── skills/concept-wiki-generator/    — 8-шаговый pipeline генерации
├── commands/
│   ├── wiki-generate.md              — /wiki-generate
│   └── triple-check.md               — /triple-check (валидатор)
├── shards/triple-pillar.shard.md     — governance gate (instantiation example)
├── governance/
│   ├── triple-pillar.governance.md   — домен-агностичная спецификация протокола
│   └── README.md
├── hooks/
│   ├── hooks.json                    — Claude Code PostToolUse advisory
│   ├── pre-commit-triple-check.sh    — git hook (bash)
│   ├── pre-commit-triple-check.ps1   — git hook (PowerShell)
│   └── README.md
└── templates/
    ├── concept.template.md           — шаблон концепт-страницы
    └── triple-pillar.config.yaml     — шаблон конфигурации
```

## Установка

### Как Claude Code plugin

```bash
# Из marketplace (когда опубликован):
/plugin install llm-wiki-harness

# Локально (из корня этого репозитория плагина):
cp -r . ~/.claude/plugins/llm-wiki-harness/
```

### Настройка под проект

1. Скопируйте `templates/triple-pillar.config.yaml` в корень проекта
2. Адаптируйте `pillar_types` и `pillar_paths` под вашу структуру (не у всех ADR/methodology/research — может быть decision/model/experiment)
3. Запустите `/wiki-generate` для первичной генерации

### Git pre-commit hook (опционально)

Установка hook'а в целевой проект — симлинком или копией в `.git/hooks/`. Точные команды (bash / PowerShell / `core.hooksPath`) — см. `hooks/README.md` и секцию `git_hooks` в `hooks/hooks.json`.

## Использование

| Команда | Назначение |
|---------|-----------|
| `/wiki-generate` | Полная генерация/регенерация wiki-слоя |
| `/wiki-generate <slug>` | Добавить/обновить один концепт |
| `/triple-check <path>` | Валидировать 3-pillar discipline в файле/директории |

## Конфигурируемость

Плагин **домен-агностичен**. `pillar_types` настраиваются: вместо ADR/methodology/research можно использовать decision/model/experiment, или standard/regulation/case-study — любые 2+ типа источников. См. `governance/triple-pillar.governance.md` §Configuration.

## Провенанс

Извлечён из проекта [GenAI-Bundle-GTM](https://github.com/djd1m/GenAI-Bundle-GTM), где провалидирован на 20 концептах, 21 ADR, 13 методиках, 13 исследованиях (все 20 концептов — full triple-pillar). Полная история — в `HARNESS_MANIFEST.md` корневого репозитория.

Core governance-протокол — часть `@dzhechkov/keysarium-core` (см. `governance/`).

## Roadmap

- [ ] `/wiki-link-check` — отдельная команда проверки всех wiki-ссылок
- [ ] HNSW vector index для semantic concept lookup
- [ ] MCP resolver `/concept <slug>` → context bundle
- [ ] Promotion остальных модулей keysarium-core (memory, orchestration, verification)
