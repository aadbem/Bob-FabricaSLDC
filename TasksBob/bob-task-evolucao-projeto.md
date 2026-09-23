# Com base na análise anterior, proponha as 5 melhorias mais impactantespara este sistema, priorizadas por
:
-
Valor de negócio (redução de risco, ganho de performance, experiência do usuário)
-
Esforço de implementação (estimativa em sprints de 2 semanas)
Para cada melhoria, inclua
:
-
Problema que resolve
-
Solução proposta
-
Benefício mensurável esperado
-
Esforço estimado
-
Dependências com outras melhorias. Salve um arquivo markdown na pasta evolucao-projeto com as sugestoes

---

**Status:** active  **Date:** 2026-09-23

---

### 👤 User

Com base na análise anterior, proponha as 5 melhorias mais impactantespara este sistema, priorizadas por
:
-
Valor de negócio (redução de risco, ganho de performance, experiência do usuário)
-
Esforço de implementação (estimativa em sprints de 2 semanas)
Para cada melhoria, inclua
:
-
Problema que resolve
-
Solução proposta
-
Benefício mensurável esperado
-
Esforço estimado
-
Dependências com outras melhorias. Salve um arquivo markdown na pasta evolucao-projeto com as sugestoes

---

### 🤖 Assistant

> 🔧 called: list_files, glob



> 🔧 **list_files** — .

> 🔧 **glob** — internal-monologue/*.md

### 🤖 Assistant

> 🔧 called: list_files, list_files



> 🔧 **list_files** — internal-monologue

> 🔧 **list_files** — galaxium-travels

### 🤖 Assistant

> 🔧 called: read_file, read_file



> 🔧 **read_file** — internal-monologue/2025-07-24_criacao-readme-raiz-repositorio.md

> 🔧 **read_file** — galaxium-travels/GT_documentacao.md

### 🤖 Assistant

> 🔧 called: read_file



> 🔧 **read_file** — galaxium-travels/GT_documentacao.md

### 🤖 Assistant

> 🔧 called: read_file



> 🔧 **read_file** — galaxium-travels/GT_documentacao.md

### 🤖 Assistant

> 🔧 called: read_file



> 🔧 **read_file** — galaxium-travels/GT_documentacao.md

### 🤖 Assistant

> 🔧 called: list_files

Tenho o contexto completo do sistema. Agora vou gerar o documento de melhorias.

> 🔧 **list_files** — .

### 🤖 Assistant

> 🔧 called: write_file



> 🔧 **write_file** — evolucao-projeto/melhorias-priorizadas.md

### 🤖 Assistant

> 🔧 called: write_file



> 🔧 **write_file** — internal-monologue/2025-07-25_melhorias-priorizadas-galaxium-travels.md

### 🤖 Assistant

Arquivo salvo em [`evolucao-projeto/melhorias-priorizadas.md`](evolucao-projeto/melhorias-priorizadas.md).

---

## As 5 melhorias mais impactantes — Galaxium Travels

Resumo executivo baseado na análise completa da arquitetura:

---

### 🔴 #1 — Autenticação com senha + sessão segura
**Problema:** O sistema identifica usuários apenas por nome + e-mail em texto plano. Qualquer pessoa com o e-mail de outro usuário pode reservar e cancelar em seu nome.
**Solução:** Senha com bcrypt + JWT com expiração + middleware de autenticação nas rotas `/book`, `/cancel`, `/bookings`.
**Benefício:** Eliminação do vetor de impersonação; conformidade com OWASP A07:2021.
**Esforço:** 1,5 sprints

---

### 🔴 #2 — Banco de dados persistente + migrações (Alembic)
**Problema:** `seed.py` apaga e recria **todos os dados a cada restart**. Reservas não sobrevivem a um simples `Ctrl+C`.
**Solução:** Separar seed da inicialização via flag `SEED_ON_STARTUP=true`, introduzir Alembic para versionamento de schema, migrar para PostgreSQL em produção.
**Benefício:** Dados persistem 100% entre deploys; ciclo de evolução de schema sem perda de dados.
**Esforço:** 1 sprint

---

### 🟡 #3 — Testes de frontend (Vitest + Testing Library)
**Problema:** Backend tem cobertura completa; frontend **zero testes**. `BookingModal`, `UserIdentification` e `Flights` podem regredir silenciosamente.
**Solução:** Vitest + @testing-library/react + msw para mock de API; meta de 70% de cobertura nos componentes de domínio.
**Benefício:** -60% de regressões detectadas em produção; base segura para implementar autenticação.
**Esforço:** 1 sprint

---

### 🟡 #4 — Filtros e paginação no backend (server-side)
**Problema:** `GET /flights` retorna todos os voos e os filtros rodam só no frontend. Não escala.
**Solução:** Query params `?origin=`, `?destination=`, `?min_price=`, `?max_price=`, `?page=` no endpoint; filtragem via SQLAlchemy; envelope de paginação.
**Benefício:** -85% de payload de rede; tempo de carregamento de ~800ms para ~120ms com volume crescente.
**Esforço:** 1 sprint

---

### 🟢 #5 — Observabilidade: logs estruturados + métricas
**Problema:** Falhas (`NO_SEATS_AVAILABLE`, `NAME_MISMATCH`) não são registradas. Impossível diagnosticar incidentes ou medir uso real.
**Solução:** structlog com saída JSON; campos `event`, `user_id`, `error_code`, `duration_ms`; prometheus-client com `/metrics`.
**Benefício:** MTTR de horas para minutos; visibilidade de KPIs de negócio (reservas por classe, taxa de cancelamento).
**Esforço:** 1 sprint

---

### Roadmap recomendado

| Fase | Melhorias |
|------|-----------|
| **Agora** (Sem. 1–4) | #2 Persistência + #3 Testes Frontend |
| **Próximo** (Sem. 5–8) | #1 Autenticação + #4 Filtros Backend |
| **Depois** (Sem. 9–10) | #5 Observabilidade |

**Trade-off explícito:** Autenticação (#1) é a mais crítica em segurança, mas foi posicionada em "Próximo" porque depende de testes estáveis (#3) para não regredir o fluxo de login existente.

**Próxima decisão para aprovar:** confirmar se a prioridade de #1 (Autenticação) deve ser antecipada para "Agora" aceitando o risco de regressão, ou manter o sequenciamento conservador proposto.