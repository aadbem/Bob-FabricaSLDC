# AGENTS.md — Modo Ask

Este arquivo fornece contexto e atalhos específicos para busca de informações e respostas de dúvidas (Ask).

## Contexto do Projeto (Não-Óbvio)
- **Dois Servidores Separados**: O backend roda em `:8080` (FastAPI) e o frontend em `:5173` (Vite). A comunicação é direta via `VITE_API_URL` (padrão `http://localhost:8080`).
- **Protocolo MCP**: O endpoint `/mcp` monta uma aplicação HTTP FastMCP independente no mesmo processo FastAPI. Não é um endpoint REST padrão.
- **Tipagem Canônica**: `src/types/index.ts` é o arquivo único que espelha os schemas de dados Pydantic do backend para o frontend.
- **Database Volátil**: O banco SQLite `booking.db` é reconstruído e semeado com dados padrão a cada startup do backend. Sumiço de dados anteriores é comportamento esperado por design.
- **Estilos Customizados**: Cores e animações customizadas como `cosmic-purple` ou `twinkle` estão registradas no arquivo de configuração do Tailwind, não vêm do framework básico.
