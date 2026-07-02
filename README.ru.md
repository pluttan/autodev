![Header](header.png)

<div align="center">

# autodev

**Автономная мультиагентная система разработки на базе opencode + DeepSeek**

[![License](https://img.shields.io/badge/license-MIT-2C2C2C?style=for-the-badge&labelColor=1E1E1E)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.12-2C2C2C?style=for-the-badge&logo=python&labelColor=1E1E1E)]()
[![FastAPI](https://img.shields.io/badge/FastAPI-backend-2C2C2C?style=for-the-badge&logo=fastapi&labelColor=1E1E1E)]()
[![React](https://img.shields.io/badge/React-18-2C2C2C?style=for-the-badge&logo=react&labelColor=1E1E1E)]()

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

## ■ Архитектура

```
Browser  --REST/WS-->  FastAPI :8765  --/mcp/mcp-->  agents (opencode)
                         |                              |
                         v                              v
                      SQLite                     docker sandbox
                         |                       (opencode-serve)
                         v                              |
              Orchestrator (state machine)              v
                         |                        DeepSeek V4 API
                         v
                 Managed projects
```

Оркестратор тикает planner/contractor для каждого проекта; они передают работу подагентам через MCP-инструменты `call` / `call_async`. Каждый проект получает один docker-контейнер (переиспользуется между запусками); агенты обращаются к серверу autodev MCP обратно на хосте по адресу `/mcp/mcp`.

## ■ Скриншоты

<div align="center">

![Screenshot](screenshots/main.png)

*Основной интерфейс с редактором графа потоков данных агентов и дашбордом проектов*

</div>

## ■ Запуск

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
