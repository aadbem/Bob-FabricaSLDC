# Melhorias Priorizadas — Galaxium Travels

> Documento gerado em: 2025-07-25
> Base de análise: [`GT_documentacao.md`](../galaxium-travels/GT_documentacao.md) + inspeção da arquitetura técnica

---

## Critérios de priorização

| Critério | Peso |
|----------|------|
| Valor de negócio (risco, performance, UX) | 60% |
| Esforço de implementação | 40% |

Escala de esforço: **1 sprint = 2 semanas**

---

## Visão geral das 5 melhorias

| # | Melhoria | Valor | Esforço | Prioridade |
|---|----------|-------|---------|------------|
| 1 | Autenticação com senha + sessão segura | 🔴 Crítico | 1,5 sprints | **Agora** |
| 2 | Banco de dados persistente + migrações | 🔴 Crítico | 1 sprint | **Agora** |
| 3 | Testes de frontend (Vitest + Testing Library) | 🟡 Alto | 1 sprint | **Próximo** |
| 4 | Pesquisa e filtros no backend (API server-side) | 🟡 Alto | 1 sprint | **Próximo** |
| 5 | Observabilidade: logs estruturados + métricas | 🟢 Médio | 1 sprint | **Depois** |

---

## Melhoria 1 — Autenticação com senha e sessão segura

### Problema que resolve

O sistema identifica o usuário apenas por **nome + e-mail em texto simples**, armazenados no `localStorage` sem qualquer token ou sessão. Qualquer pessoa que conheça o e-mail de outro usuário pode reservar ou cancelar voos em seu nome. Isso expõe risco real de fraude e viola o princípio de autenticidade.

> Trecho relevante: `UserIdentification.tsx` faz `POST /register` ou `GET /user` com `name + email`; o retorno é salvo diretamente no `localStorage` como `galaxium_user`.

### Solução proposta

1. Adicionar campo `password_hash` (bcrypt) no modelo `User` do backend.
2. Criar endpoint `POST /login` que valida credenciais e retorna um **JWT** com expiração de 8 horas (TLS 1.3 obrigatório).
3. Adicionar middleware de autenticação no backend: rotas `/book`, `/cancel`, `/bookings/{user_id}` exigem `Authorization: Bearer <token>`.
4. No frontend, substituir armazenamento no `localStorage` por **httpOnly cookie** via endpoint de sessão — ou, em alternativa segura, armazenar apenas o token JWT em `sessionStorage` (sem dados sensíveis no `localStorage`).
5. Adicionar `POST /logout` para invalidação do token no servidor (blocklist em memória ou Redis).

### Benefício mensurável esperado

- Eliminação do vetor de impersonação de usuário (0 reservas fraudulentas possíveis sem credencial válida).
- Conformidade com requisitos mínimos de autenticação (OWASP A07:2021).
- Redução de suporte por "reserva que não fiz" → estimativa de -80% dos casos.

### Esforço estimado

**1,5 sprints (3 semanas)**
- Sprint 1: backend — modelo, hash de senha, JWT, middleware
- Meia sprint 2: frontend — fluxo de login, troca de localStorage por sessionStorage/cookie

### Dependências

- Sem dependências com outras melhorias da lista.
- **Pré-requisito lógico** para a Melhoria 5 (logs de autenticação significam pouco sem identidade rastreável).

---

## Melhoria 2 — Banco de dados persistente com migrações (SQLite → PostgreSQL + Alembic)

### Problema que resolve

O `seed.py` **apaga e recria todos os dados a cada reinicialização** do servidor. Isso torna o sistema inutilizável em produção: reservas feitas pelo usuário desaparecem ao reiniciar o backend. Também bloqueia qualquer evolução de schema sem perda total de dados.

> Trecho relevante: `seed.py` — *"Executado automaticamente no startup via lifespan. Apaga e reconstrói todos os dados a cada inicialização."*

### Solução proposta

1. Separar o `seed()` da inicialização: executá-lo apenas quando a flag `SEED_ON_STARTUP=true` estiver definida (útil para demos; desativado em produção).
2. Introduzir **Alembic** para versionamento de schema: `alembic init`, `alembic revision --autogenerate`, `alembic upgrade head` no startup (apenas migrações pendentes).
3. Substituir SQLite por **PostgreSQL** como banco de produção (manter SQLite para testes unitários em memória — já funciona via `conftest.py`).
4. Mover a string de conexão para variável de ambiente `DATABASE_URL` (já suportada pelo `python-dotenv`).

### Benefício mensurável esperado

- Dados de usuário e reservas sobrevivem a reinicializações → habilitação de uso real.
- Ciclo de deploy sem perda de dados: de 0% de resiliência para 100%.
- Capacidade de evoluir o schema sem reescrever migration manual.

### Esforço estimado

**1 sprint (2 semanas)**
- Alembic + separação do seed: 3 dias
- Migração para PostgreSQL + docker-compose: 3 dias
- Ajuste de variáveis de ambiente + documentação: 2 dias

### Dependências

- Sem dependências com outras melhorias.
- **Facilita** a Melhoria 1 (tabela de sessões/blocklist de tokens) e a Melhoria 5 (logs com contexto de banco).

---

## Melhoria 3 — Cobertura de testes no frontend (Vitest + Testing Library)

### Problema que resolve

O backend tem cobertura de testes unitários e de integração (`test_services.py`, `test_rest.py`). O frontend **não possui nenhum teste automatizado**. Componentes críticos como `BookingModal`, `UserIdentification` e `Flights` podem regredir silenciosamente a cada mudança.

> Trecho relevante: *"Diretório: `booking_system_backend/tests/`"* — nenhuma menção a testes no frontend.

### Solução proposta

1. Instalar **Vitest** (compatível com Vite) + **@testing-library/react** + **@testing-library/user-event** + **msw** (mock de API).
2. Escrever testes para os 3 fluxos críticos:
   - `UserIdentification`: exibe formulário, chama `registerUser` ou `getUserByCredentials`, salva usuário no contexto.
   - `BookingModal`: exibe detalhes do voo, seleciona classe, confirma reserva, exibe erro de negócio.
   - `Flights`: renderiza lista, aplica filtros, abre modal de reserva.
3. Adicionar script `npm run test` ao `package.json` e integrar ao CI (GitHub Actions ou equivalente).
4. Meta de cobertura mínima: **70% de linhas** nos componentes de domínio.

### Benefício mensurável esperado

- Redução de regressões visuais/funcionais detectadas em produção: estimativa -60%.
- Feedback de quebra em < 2 minutos (pipeline) vs. descoberta manual.
- Base segura para implementar a Melhoria 1 (novo fluxo de login testado).

### Esforço estimado

**1 sprint (2 semanas)**
- Setup de Vitest + msw: 2 dias
- Testes de `UserIdentification` + `BookingModal`: 4 dias
- Testes de `Flights` + configuração de CI: 4 dias

### Dependências

- Independente das outras melhorias.
- **Altamente recomendada antes** da Melhoria 1 para evitar regredir o fluxo de login.

---

## Melhoria 4 — Filtros e paginação no backend (API server-side)

### Problema que resolve

O endpoint `GET /flights` retorna **todos os voos sem filtro ou paginação**. Os filtros (origem, destino, faixa de preço) são aplicados **exclusivamente no frontend** em memória. Conforme o volume de voos cresce, isso aumenta o payload de rede, o tempo de carregamento e o uso de memória no cliente.

> Trecho relevante: `Flights.tsx` — *"Filtros reativos: origem, destino, preço mínimo/máximo"* (front-only). `GET /flights` retorna todos os voos.

### Solução proposta

1. Adicionar parâmetros de query ao `GET /flights`:
   - `?origin=`, `?destination=`, `?min_price=`, `?max_price=`, `?page=`, `?page_size=` (padrão: 20)
2. Implementar a filtragem na camada de serviço (`services/flight.py`) via cláusulas SQLAlchemy (`.filter()`, `.offset()`, `.limit()`).
3. Retornar envelope de paginação: `{ data: [...], total: N, page: P, page_size: 20 }`.
4. Adaptar o frontend para passar os filtros como query params e consumir o envelope paginado.
5. Manter os filtros locais como fallback para UX responsiva (debounce no input → chamada à API).

### Benefício mensurável esperado

- Redução de payload de rede: de O(n_voos) para O(page_size) — estimativa -85% com 500+ voos.
- Tempo de carregamento inicial da página `/flights`: de ~800ms para ~120ms (estimativa com 200 voos).
- Preparação para escala sem refatoração futura.

### Esforço estimado

**1 sprint (2 semanas)**
- Backend — query params + SQLAlchemy filters + paginação: 4 dias
- Frontend — debounce + adaptação ao envelope paginado + componente de paginação: 4 dias
- Testes de integração REST para os novos parâmetros: 2 dias

### Dependências

- **Independente** das outras melhorias.
- Beneficia-se da Melhoria 3 (testes de frontend facilitam validar a adaptação de `Flights.tsx`).

---

## Melhoria 5 — Observabilidade: logs estruturados + métricas de negócio

### Problema que resolve

O backend não possui **logging estruturado** nem **métricas** de operação. Falhas silenciosas (ex.: `NO_SEATS_AVAILABLE`, `NAME_MISMATCH`) não são rastreáveis em produção. Não há visibilidade sobre volume de reservas, taxa de cancelamento ou erros frequentes.

> Trecho relevante: serviços retornam `ErrorResponse` sem registrar eventos — sem `logger.info`, `logger.error` ou equivalente visível na documentação.

### Solução proposta

1. Introduzir **structlog** (ou `logging` padrão com formatador JSON) no backend.
2. Adicionar log em cada operação de serviço com campos: `event`, `user_id`, `flight_id`, `seat_class`, `error_code`, `duration_ms`.
3. Nunca logar dados sensíveis (e-mail, nome completo) — apenas IDs e códigos de erro.
4. Expor endpoint `GET /metrics` com contadores básicos via **prometheus-client**:
   - `bookings_total` (por `seat_class` e `status`)
   - `cancellations_total`
   - `errors_total` (por `error_code`)
5. Documentar dashboard mínimo em Grafana (ou exibir em Swagger para demos).

### Benefício mensurável esperado

- MTTR (tempo médio de resolução de incidentes): de horas para minutos com logs pesquisáveis.
- Visibilidade de taxa de erro por `error_code` → identifica gargalos de UX (ex.: classe `galaxium` esgotando mais rápido).
- Base para SLA: é impossível medir o que não se observa.

### Esforço estimado

**1 sprint (2 semanas)**
- structlog + formatação JSON: 2 dias
- Instrumentação de todos os serviços: 3 dias
- prometheus-client + endpoint `/metrics`: 3 dias
- Documentação de dashboard mínimo: 2 dias

### Dependências

- **Beneficia-se da Melhoria 1** (autenticação): logs ficam mais ricos com `user_id` rastreável via JWT.
- **Beneficia-se da Melhoria 2** (persistência): logs de banco fazem sentido quando dados sobrevivem ao restart.
- Pode ser implementada de forma independente com valor parcial.

---

## Mapa de dependências

```
Melhoria 1 (Autenticação)
    ↑ facilita logs ricos
Melhoria 5 (Observabilidade) ←── Melhoria 2 (Persistência)
                                       ↑
                                  sem dependência direta

Melhoria 3 (Testes Frontend) ←── recomendada antes de Melhoria 1
Melhoria 4 (Filtros Backend) ←── independente; beneficia-se de Melhoria 3
```

---

## Roadmap sugerido

| Fase | Melhorias | Período |
|------|-----------|---------|
| **Agora** (Sprint 1–2) | Melhoria 2 (Persistência) + Melhoria 3 (Testes) | Semanas 1–4 |
| **Próximo** (Sprint 3–4) | Melhoria 1 (Autenticação) + Melhoria 4 (Filtros) | Semanas 5–8 |
| **Depois** (Sprint 5) | Melhoria 5 (Observabilidade) | Semanas 9–10 |

> **Trade-off explícito:** Autenticação (Melhoria 1) tem alto valor de segurança, mas requer base estável de testes (Melhoria 3) para não regredir o fluxo de login. Por isso é alocada em "Próximo", não em "Agora".

---

*Documento gerado por análise de produto — Galaxium Travels v1.0*
