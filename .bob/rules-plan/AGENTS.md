# AGENTS.md — Modo Plan

Este arquivo fornece diretrizes e restrições arquiteturais para planejamento de arquitetura (Plan).

## Restrições de Arquitetura (Não-Óbvias)
- **Co-locação de Processos**: FastAPI e FastMCP rodam sob o mesmo processo Python. Separações em microsserviços devem prever alteração no ciclo de vida de inicialização.
- **Acoplamento Manual de Tipos**: O mapeamento de tipos Pydantic ↔ TypeScript é feito à mão em `src/types/index.ts`. Alterações de schema exigem manutenção em ambos os ecossistemas simultaneamente.
- **Ausência de Migrações de Dados**: Não há suporte ao Alembic. Mudanças estruturais de banco exigem recriação limpa usando `Base.metadata.create_all()` no ciclo de vida de startup.
- **Validação de Usuários Suspensa**: O sistema aceita qualquer `user_id` enviado nas requisições REST sem validação de sessão ativa de login. Autenticação completa requer middleware customizado no FastAPI.
- **Monkeypatch nos Testes**: O `conftest.py` substitui `SessionLocal` globalmente via monkeypatching. Refatorações estruturais em `db.py` podem quebrar essa infraestrutura de testes em memória.
