<div align="center">

# autodev

**Автономная мультиагентная система разработки на базе opencode + DeepSeek**


</div>

Команда из 8 агентов (planner, contractor, coder, designer, tester, researcher, arbiter, acceptor) взаимодействует через opencode + DeepSeek V4, чтобы исследовать рыночный спрос, декомпозировать функциональность на задачи, реализовывать код и проверять его соответствие критериям приёмки. Оркестратор на FastAPI тикает двух первичных агентов (planner, contractor) и предоставляет слой MCP-инструментов, через который они вызывают подагентов; каждый агент запускается в эфемерном docker-контейнере. Состояние хранится в SQLite и отображается в React UI с визуальным редактором графа потоков данных для конфигурации агентов.

## ■ Возможности

- ❖ **8 агентов** — planner (исследование рынка), contractor (декомпозиция), coder, designer (ревью UI), tester (чёрный ящик), researcher, arbiter (проверка цели), acceptor (ревью)
- ❖ **Оркестратор** — конечный автомат тикает только первичных агентов: `idle -> planning -> contracting -> idle`; подагенты вызываются через MCP
- ❖ **Слой MCP-инструментов** — сервер `autodev-agents` предоставляет `call` / `call_async` / `read_db` / `write_db` / `git_commit_push`, доступ определяется пермишенами агента, выведенными из графа
- ❖ **Изолированные запуски** — каждый запуск агента поднимает новый docker-контейнер с примонтированным проектом и opencode-serve внутри
- ❖ **Редактор потоков на React** — канвас @xyflow/react с узлами агентов и мета-узлами db/ticker/text/git, создание рёбер перетаскиванием, undo/redo
- ❖ **Инспектор агента** — редактирование роли, модели (Pro/Flash), инструментов, температуры, режима мышления, системного промпта, prefill-промпта, формата входа/выхода
- ❖ **Живой UI** — WebSocket pub/sub проталкивает каждую CRUD-мутацию для мгновенной инвалидации TanStack Query
- ❖ **Журнал запусков** — рендерер транскриптов с вызовами инструментов, рассуждениями, учётом токенов и стоимости за запуск
- ❖ **Дашборд статистики** — линейные графики cost/tokens, столбчатый график запусков, окно 14 дней с заполнением нулями
- ❖ **Авто-коммит** — contractor коммитит и пушит через MCP-инструмент `git_commit_push` после того, как acceptor принял элемент бэклога, используя бот-идентификатор git для каждого проекта

## ■ Стек

<div align="center">

| Компонент | Технология |
|-----------|------------|
| Backend | FastAPI + SQLAlchemy 2.0 async + aiosqlite |
| Frontend | React 18 + Vite 5 + TypeScript + TanStack Query + @xyflow/react |
| Среда выполнения агентов | opencode (sandboxed, image `opencode-runner:1.14.25`) |
| Коммуникация агентов | MCP server `autodev-agents` (FastMCP, streamable HTTP) |
| LLM | DeepSeek V4 (per-agent Pro/Flash, selectable in the inspector) |
| Веб-поиск | Serper.dev |
| База данных | SQLite (projects, backlog, tasks, runs, agent sessions) |
| Миграции | Alembic |
| Дизайн | Catppuccin Mocha (CSS variables) |

</div>

## ■ Как это работает

```
1. Браузер подключается к FastAPI :8765 через REST/WebSocket; оркестратор тикает planner и contractor для каждого проекта в цикле конечного автомата (idle → planning → contracting → idle).
2. Агенты planner и contractor запускаются внутри эфемерных docker-контейнеров (opencode-serve) и общаются с хостом через MCP-сервер autodev-agents по адресу /mcp/mcp.
3. Первичные агенты диспатчат подагентов (coder, designer, tester, researcher, arbiter, acceptor) через MCP-инструменты call / call_async, с пермишенами, выведенными из графа потоков.
4. Каждый агент вызывает DeepSeek V4; результаты, транскрипты и стоимость в токенах сохраняются в SQLite и мгновенно отправляются в React UI через WebSocket pub/sub.
5. После того как acceptor одобряет элемент бэклога, contractor вызывает git_commit_push для коммита и пуша изменений с использованием бот-идентификатора git для каждого проекта.
```

## ■ Скриншоты

<div align="center">

![Screenshot](screenshots/main.png)

*Основной интерфейс с редактором графа потоков данных агентов и дашбордом проектов*

</div>

## ■ Использование

```bash
# Backend
make install
make serve              # uvicorn :8765

# Frontend
make web-install        # pnpm install
make web-dev            # vite :5174

# Orchestrator
make orchestrate        # непрерывный цикл тиков
make run-once           # один тик

# Sandbox runtime (docker)
make sandbox-build
make sandbox-up

# Migrations / quality
make migrate-up         # alembic upgrade head
make test               # pytest
make lint               # ruff
```

## ■ Лицензия

MIT © [pluttan](https://github.com/pluttan)
