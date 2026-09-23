# Bob-Fabrica

Repositório de laboratório para exploração prática do **IBM Bob** — IA de desenvolvimento da IBM. Contém projetos de exemplo integrados ao Bob via MCP, demonstrando fluxos de agente, dual-protocol (REST + MCP) e boas práticas de desenvolvimento assistido por IA.

---

## Projetos

### 🚀 Galaxium Travels

Sistema full-stack de reservas de viagens interplanetárias, desenvolvido como projeto-demonstração para o Bob.

**Stack:**
- **Backend** — Python · FastAPI · SQLAlchemy · Pydantic v2 · FastMCP · SQLite
- **Frontend** — React 19 · TypeScript · Vite · Tailwind CSS · Framer Motion

**Diferenciais:**
- Dual-protocol: mesma lógica de negócio exposta via **REST API** e **MCP tools**
- Servidor MCP integrado ao FastAPI em `/mcp` — pronto para conexão com agentes de IA
- Banco recriado automaticamente com dados de demo a cada inicialização

👉 Veja [`galaxium-travels/README.md`](galaxium-travels/README.md) para documentação completa.

---

## Estrutura do Repositório

```
Bob-Fabrica/
├── galaxium-travels/
│   ├── booking_system_backend/   # FastAPI + FastMCP (Python)
│   ├── booking_system_frontend/  # React + TypeScript
│   └── start.sh                  # Inicialização com um comando
├── AGENTS.md                     # Regras e instruções para agentes de IA
└── .bob/                         # Configurações do IBM Bob (MCP, skills, etc.)
```

---

## Quick Start — Galaxium Travels

### Pré-requisitos

| Ferramenta | Versão mínima |
|---|---|
| Python | 3.11+ |
| Node.js | 18+ |

### Um comando

```bash
cd galaxium-travels
./start.sh
```

O script cria o virtualenv, instala todas as dependências e sobe os dois servidores:

| URL | Serviço |
|---|---|
| `http://localhost:5173` | Frontend React |
| `http://localhost:8080` | Backend REST |
| `http://localhost:8080/docs` | Swagger UI |
| `http://localhost:8080/mcp` | MCP endpoint |

### Inicialização manual

**Backend:**
```bash
cd galaxium-travels/booking_system_backend
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python server.py
```

**Frontend** (em outro terminal):
```bash
cd galaxium-travels/booking_system_frontend
npm install
npm run dev
```

---

## Integração com IBM Bob (MCP)

O backend expõe um servidor MCP em `http://localhost:8080/mcp`. Para conectar o Bob ao projeto, adicione a entrada em `.bob/mcp.json`:

```json
{
  "mcpServers": {
    "galaxium-booking": {
      "type": "http",
      "url": "http://localhost:8080/mcp"
    }
  }
}
```

### MCP Tools disponíveis

| Ferramenta | Parâmetros | Descrição |
|---|---|---|
| `list_flights` | — | Lista voos com classes de assento |
| `book_flight` | `user_id`, `name`, `flight_id`, `seat_class` | Reserva um assento |
| `get_bookings` | `user_id` | Lista reservas do usuário |
| `cancel_booking` | `booking_id` | Cancela uma reserva |
| `register_user` | `name`, `email` | Registra um novo usuário |
| `get_user_id` | `name`, `email` | Busca usuário por nome e e-mail |

---

## Testes

```bash
# Todos os testes (Python)
./galaxium-travels/booking_system_backend/.venv/bin/pytest galaxium-travels/booking_system_backend

# Lint do frontend
npm --prefix galaxium-travels/booking_system_frontend run lint

# Build do frontend
npm --prefix galaxium-travels/booking_system_frontend run build
```

---

## Documentação

| Documento | Conteúdo |
|---|---|
| [`galaxium-travels/README.md`](galaxium-travels/README.md) | Visão geral, quick start, funcionalidades, troubleshooting |
| [`galaxium-travels/booking_system_backend/README.md`](galaxium-travels/booking_system_backend/README.md) | Endpoints REST, MCP tools, modelo de dados, testes |
| [`galaxium-travels/booking_system_frontend/README.md`](galaxium-travels/booking_system_frontend/README.md) | Componentes, design system, deploy |
| [`galaxium-travels/GT_documentacao.md`](galaxium-travels/GT_documentacao.md) | Documentação técnica detalhada (arquitetura, fluxos, schemas) |
| [`AGENTS.md`](AGENTS.md) | Instruções e gotchas para agentes de IA |

---

*Feito com IBM Bob* ✦
