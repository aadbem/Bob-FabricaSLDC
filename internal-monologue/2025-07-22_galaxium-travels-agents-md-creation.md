# Resumo: Criação de AGENTS.md para galaxium-travels

## O que foi feito

Analisado o projeto `galaxium-travels` (monorepo Python FastAPI + React 19/TypeScript) e criados arquivos de orientação para agentes de IA:

### Descobertas principais
- `AGENTS.md` raiz já existia e estava bem estruturado com informações não-óbvias corretas
- Não havia arquivos `.bob/rules-*/AGENTS.md` nem regras de outros assistentes (CLAUDE.md, .cursorrules, etc.)

### Arquivos criados/modificados
- **`galaxium-travels/.bob/rules-agent/AGENTS.md`** — regras de codificação não-óbvias (imports planos, unions em vez de exceções, `model_validate`, `FlightSeatClass` como fonte de verdade, ordem de criação MCP/FastAPI)
- **`galaxium-travels/.bob/rules-ask/AGENTS.md`** — contexto não-óbvio para perguntas (dois servidores separados, tipos canônicos em `src/types/index.ts`, endpoint `/mcp` como protocolo separado, cores Tailwind customizadas, `booking.db` temporário por design)
- **`galaxium-travels/.bob/rules-plan/AGENTS.md`** — restrições arquiteturais (MCP+FastAPI co-locados, acoplamento manual de tipos, SQLite sem migrações, seeding como único mecanismo de dados, ausência de autenticação, dependência de monkeypatch em testes)

### Stack identificado
- Backend: Python 3.11 + FastAPI + FastMCP + SQLAlchemy (SQLite) + Pydantic v2
- Frontend: React 19 + TypeScript 5.9 + Vite 7 + Tailwind CSS 3 + axios + framer-motion
- Testes: pytest + pytest-asyncio + httpx (in-memory SQLite)
