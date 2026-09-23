# AGENTS.md — Modo Agent

Este arquivo fornece regras e instruções específicas para desenvolvimento e codificação (Agent).

## Regras Críticas de Codificação (Não-Óbvias)
- **Imports Backend Planos**: Use sempre `from models import ...` ou `from services import ...`. Nunca prefixe com o diretório pai.
- **Retornos de Serviço**: Funções em `services/` retornam `Model | ErrorResponse`. Em ferramentas MCP, lance exceções Python se o retorno for `ErrorResponse` (`raise Exception(result.details)`). Em REST endpoints, retorne o union direto (HTTP 200).
- **Validação de Schemas**: Sempre use `model_validate()` do Pydantic v2 para conversão ORM, nunca `.from_orm()`.
- **Decremento/Incremento de Vagas**: Modifique sempre `FlightSeatClass.seats_available` em vez de `Flight.seats_available` (este é legado).
- **Frontend Type Guard**: Para tratamento de erro, use a função utilitária `isErrorResponse(response)` em `src/services/api.ts` em vez de `instanceof`.
- **JSDoc**: Forneça JSDoc conciso para toda nova função pública criada no frontend.
