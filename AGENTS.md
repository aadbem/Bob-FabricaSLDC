# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## Diretrizes e Estilo (Regras do Projeto)
- **Idioma**: Toda documentação e interações do projeto devem ser em **Português do Brasil**.
- **JSDoc**: Sempre inclua strings JSDoc concisas para todas as funções públicas (no frontend).
- **Tratamento de Erros**: O backend retorna HTTP 200 com `ErrorResponse` em caso de erros de negócio. Use o type guard `isErrorResponse(response)` no frontend; não use `instanceof`.
- **Tratamento de Vagas**: Alterações de vagas de assento incrementam/decrementam `FlightSeatClass.seats_available`, e **não** `Flight.seats_available` (campo legado).
- **Resumos**: No final de cada tarefa/interação, salve um resumo em `internal-monologue/` nomeado como `YYYY-MM-DD_descricao-concisa.md`.

## Comandos Críticos (Executados a partir da raiz)
- **Rodar teste específico (Python)** (use o executável do venv se o global não estiver instalado):
  ```bash
  ./galaxium-travels/booking_system_backend/.venv/bin/pytest galaxium-travels/booking_system_backend/tests/test_services.py::TestBookingService::test_book_flight_success
  ```
- **Rodar todos os testes (Python)**:
  ```bash
  ./galaxium-travels/booking_system_backend/.venv/bin/pytest galaxium-travels/booking_system_backend
  ```
- **Lint do Frontend**:
  ```bash
  npm --prefix galaxium-travels/booking_system_frontend run lint
  ```
- **Build do Frontend**:
  ```bash
  npm --prefix galaxium-travels/booking_system_frontend run build
  ```

## Gotchas Técnicos
- **Imports Backend**: Os imports no backend são planos (`from models import ...`, não usar prefixes de pacote). O `conftest.py` manipula `sys.path` automaticamente nos testes.
- **Banco de Dados**: O arquivo `booking.db` é temporário e recriado via `seed()` no startup. Toda alteração de schema requer reinicializar o app (sem migrações de banco via Alembic).
- **FastMCP**: O FastMCP é montado via `app.mount("/mcp", mcp_app)` e deve ser inicializado antes do `FastAPI()`.
