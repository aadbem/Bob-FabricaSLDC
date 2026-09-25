# Melhorias Funcionais — Galaxium Travels

> Documento gerado em: 2025-07-25  
> Foco: **valor de negócio e experiência do usuário** — complementa [`melhorias-priorizadas.md`](./melhorias-priorizadas.md) (técnico/segurança)  
> Base de análise: inspeção completa do código — `models.py`, `services/`, `server.py`, `Flights.tsx`, `MyBookings.tsx`, `types/index.ts`, `seed.py`

---

## Resumo executivo

O Galaxium Travels é um sistema de reservas interplanetárias com fluxo funcional básico completo (busca → reserva → cancelamento). As lacunas de **negócio** mais críticas são: ausência de pagamento, impossibilidade de alterar reservas, comunicação zero com o passageiro, busca limitada a campos isolados e nenhuma personalização de oferta. As 5 melhorias abaixo foram priorizadas pelo impacto direto na experiência do usuário e no potencial de receita, balanceadas pelo esforço de entrega.

---

## Critérios de priorização

| Critério | Peso |
|----------|------|
| Valor de negócio (UX, receita, retenção) | 60% |
| Esforço de implementação (1 sprint = 2 semanas) | 40% |

---

## Visão geral das 5 melhorias

| # | Melhoria | Valor | Esforço | Fase |
|---|----------|-------|---------|------|
| 1 | Fluxo de pagamento simulado com resumo de compra | 🔴 Crítico | 1,5 sprints | **Agora** |
| 2 | Alteração e reemissão de reserva (rebooking) | 🔴 Crítico | 1 sprint | **Agora** |
| 3 | Busca inteligente e recomendações de voo | 🟡 Alto | 1,5 sprints | **Próximo** |
| 4 | Notificações e confirmação pós-reserva | 🟡 Alto | 1 sprint | **Próximo** |
| 5 | Painel do passageiro com histórico e fidelidade | 🟢 Médio | 2 sprints | **Depois** |

---

## Melhoria 1 — Fluxo de pagamento simulado com resumo de compra

### Problema que resolve

O usuário seleciona um voo, escolhe a classe e confirma — a reserva é criada instantaneamente sem nenhum **resumo de custo, confirmação financeira ou recibo**. Não há campo de preço final na reserva (`Booking` não armazena o valor pago). O usuário não sabe quanto pagou e não há oportunidade de captura de receita real ou simulada.

> Evidência no código: `BookingRequest` em [`types/index.ts`](../galaxium-travels/booking_system_frontend/src/types/index.ts) contém apenas `user_id`, `name`, `flight_id` e `seat_class` — sem `price_paid`. O modelo [`Booking`](../galaxium-travels/booking_system_backend/models.py) não persiste o valor cobrado.

### Solução proposta

1. **Backend** — adicionar campo `price_paid` (Integer) ao modelo `Booking` e populá-lo no momento da reserva via `round(flight.price × price_multiplier)`.
2. **Backend** — criar endpoint `POST /checkout` que recebe os dados da reserva, devolve um "resumo de pedido" (voo, classe, passageiro, valor total) e só após confirmação do frontend cria a reserva.
3. **Frontend** — adicionar etapa de "Resumo & Pagamento" no `BookingModal`: tela intermediária com valor total destacado, seletor de forma de pagamento simulada (cartão galáctico / créditos de viagem) e botão "Confirmar e Pagar".
4. **Frontend** — exibir recibo inline após confirmação com número da reserva, valor e itinerário.
5. **Frontend** — mostrar o campo `price_paid` nos cards de [`MyBookings.tsx`](../galaxium-travels/booking_system_frontend/src/pages/MyBookings.tsx).

### Benefício mensurável esperado

- **Transparência de preço**: 100% das reservas passam a ter valor registrado → base para relatórios de receita.
- **Redução de abandono no checkout**: etapa de confirmação com resumo claro tende a reduzir dúvidas de preço em ~30% (benchmark e-commerce).
- **Receita rastreável**: viabiliza dashboard de faturamento futuro (soma de `price_paid` por período/destino).

### Esforço estimado

**1,5 sprints (3 semanas)**
- Meia sprint 1: modelo + campo `price_paid` no backend + endpoint `/checkout`
- Sprint 1,5: etapa de resumo no `BookingModal` + recibo + exibição em `MyBookings`

### Dependências

- **Independente** das outras melhorias.
- **Facilita** a Melhoria 5 (painel de histórico com gastos totais).
- **Recomendada antes** da Melhoria 4 (notificação de confirmação precisa incluir o valor pago).

---

## Melhoria 2 — Alteração e reemissão de reserva (rebooking)

### Problema que resolve

O único ciclo de vida de uma reserva é `booked → cancelled`. Não existe opção de **trocar de voo, alterar classe ou remarcar** uma viagem. O usuário que precisa mudar de itinerário é forçado a cancelar e refazer a reserva manualmente — dois fluxos separados sem garantia de disponibilidade. Em sistemas de transporte, a impossibilidade de rebooking é uma das principais causas de abandono de plataforma.

> Evidência no código: [`cancel_booking`](../galaxium-travels/booking_system_backend/services/booking.py) apenas muda `status` para `"cancelled"` e restaura o assento. Não há endpoint `PATCH /bookings/{id}` ou similar.

### Solução proposta

1. **Backend** — criar endpoint `PATCH /bookings/{booking_id}` que aceita `new_flight_id` e/ou `new_seat_class` opcionais:
   - Valida disponibilidade no novo voo/classe antes de alterar.
   - Restaura assento no voo original, decrementa no novo.
   - Atualiza `Booking.flight_id`, `Booking.seat_class` e `Booking.price_paid` (se implementada a Melhoria 1).
   - Adiciona campo `last_modified_time` ao modelo `Booking`.
2. **Backend** — adicionar validação de regra de negócio: rebooking só permitido com pelo menos **24h de antecedência** da partida original.
3. **Frontend** — adicionar botão "Alterar reserva" em [`BookingCard`](../galaxium-travels/booking_system_frontend/src/components/bookings/BookingCard.tsx) para reservas `booked`.
4. **Frontend** — modal de rebooking com seletor de novo voo (lista filtrada por mesma origem/destino) e nova classe, com comparativo de preço.

### Benefício mensurável esperado

- **Retenção de receita**: rebooking retém 100% da receita vs. cancelamento que perde 100%. Estimativa: conversão de 40% dos cancelamentos em reemissões.
- **Redução de churn**: usuários com opção de alterar têm menor probabilidade de migrar para concorrente.
- **Volume de reservas ativas**: aumento estimado de 15–20% no total de reservas `booked` em um dado momento.

### Esforço estimado

**1 sprint (2 semanas)**
- Backend — endpoint `PATCH`, validações de disponibilidade + regra das 24h: 5 dias
- Frontend — botão + modal de rebooking + comparativo de preço: 5 dias

### Dependências

- **Beneficia-se da Melhoria 1** (campo `price_paid` permite mostrar diferença de custo no rebooking).
- **Independente** das demais melhorias — pode ser entregue isoladamente.

---

## Melhoria 3 — Busca inteligente e recomendações de voo

### Problema que resolve

A busca atual em [`Flights.tsx`](../galaxium-travels/booking_system_frontend/src/pages/Flights.tsx) oferece apenas filtros de **origem exata, destino exato e faixa de preço** — todos processados no frontend com todos os voos carregados em memória. Não há campo de texto livre, sugestão de destinos, ordenação por critério (mais barato, mais rápido, partida mais próxima), nem recomendação baseada em comportamento.

> Evidência no código: `EMPTY_FILTERS` em [`Flights.tsx`](../galaxium-travels/booking_system_frontend/src/pages/Flights.tsx) tem `searchTerm` definido em [`FlightFilters`](../galaxium-travels/booking_system_frontend/src/types/index.ts) mas **nunca utilizado** na lógica de filtragem. O campo foi declarado mas não implementado.

### Solução proposta

1. **Backend** — adicionar parâmetro `?search=` ao `GET /flights`: busca por substring em `origin` e `destination` (case-insensitive, SQLAlchemy `.ilike()`).
2. **Backend** — adicionar parâmetro `?sort_by=price|departure|duration` com `?sort_order=asc|desc`.
3. **Backend** — criar endpoint `GET /flights/recommendations?user_id=` que retorna os 3 voos mais reservados pelo usuário (por origem/destino) + 2 destinos populares da semana.
4. **Frontend** — implementar o campo `searchTerm` já declarado no tipo `FlightFilters`: input de texto livre com debounce de 300ms.
5. **Frontend** — adicionar controles de ordenação acima da grade de voos.
6. **Frontend** — exibir seção "Recomendados para você" na Home para usuários logados.

### Benefício mensurável esperado

- **Tempo até reserva**: redução estimada de 40% no tempo de descoberta de voo (menos cliques para encontrar o destino desejado).
- **Taxa de conversão**: busca relevante aumenta conversão de visita → reserva em ~25% (benchmark viagens online).
- **Engajamento na Home**: seção de recomendações aumenta cliques em voos em ~35% vs. lista estática.

### Esforço estimado

**1,5 sprints (3 semanas)**
- Sprint 1: backend — busca por texto, ordenação, endpoint de recomendações simples
- Meia sprint 2: frontend — campo de busca, ordenação, seção "Recomendados" na Home

### Dependências

- **Independente** da Melhoria 1 (pagamento) e Melhoria 2 (rebooking).
- **Beneficia-se** de dados reais de reservas (quanto mais bookings, melhores as recomendações) → relacionada indiretamente com a Melhoria 5.
- Complementa (sem depender de) a Melhoria 4 (notificações).

---

## Melhoria 4 — Notificações e confirmação pós-reserva

### Problema que resolve

Após confirmar uma reserva, o usuário recebe apenas um toast ("Booking confirmed") que desaparece em 3 segundos. **Não há confirmação por e-mail, nenhum lembrete de viagem, nenhum aviso de cancelamento.** O usuário não tem como recuperar o número da reserva sem acessar o sistema. Em qualquer plataforma de viagens, a ausência de comunicação pós-reserva é associada à percepção de baixa confiabilidade.

> Evidência no código: `handleBookingSuccess` em [`Flights.tsx`](../galaxium-travels/booking_system_frontend/src/pages/Flights.tsx) apenas recarrega a lista de voos. Nenhum serviço de e-mail ou notificação está presente no backend.

### Solução proposta

1. **Backend** — criar módulo `services/notification.py` com função `send_booking_confirmation(booking, user, flight)` que:
   - Formata um e-mail HTML com detalhes do voo (origem, destino, horário, classe, `booking_id`).
   - Envia via **SMTP simulado** (MailHog para dev) ou serviço real configurável por `EMAIL_PROVIDER` env var.
2. **Backend** — disparar `send_booking_confirmation` após criação bem-sucedida em `services/booking.py`.
3. **Backend** — criar job agendado (APScheduler) para lembrete 48h antes da partida: `GET /flights` com `departure_time` próximo + `Booking.status == "booked"` → envia e-mail de lembrete.
4. **Frontend** — após reserva confirmada, exibir modal de "Confirmação enviada" com o `booking_id` e instrução de verificar e-mail.
5. **Frontend** — adicionar página `/booking/{id}` acessível por link direto (deep link compartilhável).

### Benefício mensurável esperado

- **Confiabilidade percebida**: confirmação por e-mail é citada em 78% das avaliações positivas de plataformas de viagem (pesquisa de setor).
- **Redução de suporte**: "não recebi meu bilhete" → estimativa de -60% dos contatos de suporte.
- **Lembretes reduzem no-show**: notificação 48h antes reduz não-comparecimento em ~20% (dado da indústria aérea).

### Esforço estimado

**1 sprint (2 semanas)**
- Backend — módulo de notificação + SMTP simulado + trigger pós-booking: 5 dias
- Backend — job de lembrete (APScheduler): 2 dias
- Frontend — modal de confirmação + página de detalhe `/booking/{id}`: 3 dias

### Dependências

- **Beneficia-se da Melhoria 1** (campo `price_paid` enriquece o e-mail de confirmação com o valor cobrado).
- **Independente** das Melhorias 2, 3 e 5.
- **Recomendada em paralelo** com a Melhoria 2: rebooking também deve disparar notificação de alteração.

---

## Melhoria 5 — Painel do passageiro com histórico e programa de fidelidade

### Problema que resolve

A página [`MyBookings.tsx`](../galaxium-travels/booking_system_frontend/src/pages/MyBookings.tsx) exibe uma lista de reservas ativas e passadas, mas sem **estatísticas, valor total gasto, destinos favoritos ou qualquer forma de recompensa por fidelidade**. O usuário recorrente não tem incentivo visual para continuar usando a plataforma. Não há diferenciação entre um passageiro de primeira viagem e um que já fez 10 viagens interplanetárias.

> Evidência no código: `MyBookings.tsx` divide bookings em `activeBookings` e `pastBookings` — sem agregações ou gamificação. O modelo `User` contém apenas `user_id`, `name` e `email` — sem pontos, tier ou preferências.

### Solução proposta

1. **Backend** — criar endpoint `GET /users/{user_id}/stats` que retorna:
   - `total_bookings`, `total_spent` (soma de `price_paid`), `destinations_visited` (lista única), `most_frequent_route`, `current_tier` (`explorer / pioneer / galaxium_elite`).
2. **Backend** — adicionar campo `loyalty_points` ao modelo `User` (calculado como `price_paid / 10000` por reserva concluída). Atualizar ao marcar reserva como `completed`.
3. **Frontend** — criar componente `PassengerStats` com cards de métricas (viagens, créditos gastos, destinos visitados).
4. **Frontend** — exibir badge de tier ("Galaxium Elite 🚀") no `Header` para usuários autenticados.
5. **Frontend** — adicionar aba "Recompensas" em `MyBookings` com saldo de pontos e próximo milestone de tier.

### Benefício mensurável esperado

- **Retenção de longo prazo**: programas de fidelidade aumentam frequência de compra em 20–30% (benchmark aviação comercial).
- **Receita por usuário (ARPU)**: usuários de tier alto tendem a reservar classes superiores — aumento estimado de 15% no ticket médio.
- **Engajamento**: painel com estatísticas aumenta tempo de sessão médio em ~40% (benchmark plataformas com gamificação).

### Esforço estimado

**2 sprints (4 semanas)**
- Sprint 1: endpoint de stats + campo `loyalty_points` + lógica de tier no backend
- Sprint 2: componente `PassengerStats` + badge de tier no header + aba de recompensas

### Dependências

- **Requer a Melhoria 1** (campo `price_paid`) para calcular `total_spent` e `loyalty_points` com precisão.
- **Beneficia-se da Melhoria 4** (notificações de milestone de fidelidade enriquecem o programa).
- É a melhoria de maior esforço — indicada para "Depois" após base funcional consolidada.

---

## Mapa de dependências

```
Melhoria 1 (Pagamento simulado)
    ├──► enriquece ──► Melhoria 2 (Rebooking — diferença de preço)
    ├──► enriquece ──► Melhoria 4 (Notificações — valor no e-mail)
    └──► requerida ──► Melhoria 5 (Fidelidade — total_spent)

Melhoria 3 (Busca inteligente) ──► independente de todas

Melhoria 4 (Notificações) ──► recomendada em paralelo com Melhoria 2
```

---

## Roadmap sugerido

| Fase | Melhorias | Período |
|------|-----------|---------|
| **Agora** (Sprint 1–3) | Melhoria 1 (Pagamento) + Melhoria 2 (Rebooking) | Semanas 1–6 |
| **Próximo** (Sprint 4–5) | Melhoria 3 (Busca) + Melhoria 4 (Notificações) | Semanas 7–10 |
| **Depois** (Sprint 6–7) | Melhoria 5 (Painel + Fidelidade) | Semanas 11–14 |

> **Trade-off explícito:** A Melhoria 5 (Fidelidade) tem o maior potencial de retenção de longo prazo, mas **requer** a Melhoria 1 para ter dados financeiros confiáveis. Tentar entregas na ordem inversa geraria retrabalho de schema. Por isso é alocada em "Depois".

---

## Comparativo com melhorias técnicas

Este documento trata exclusivamente de **funcionalidades de negócio**. As melhorias de infraestrutura e segurança documentadas em [`melhorias-priorizadas.md`](./melhorias-priorizadas.md) (autenticação, persistência de BD, testes de frontend, filtros server-side, observabilidade) são **pré-requisitos transversais** e devem ser executadas em paralelo pela trilha técnica.

| Fase | Trilha Funcional (este doc) | Trilha Técnica ([melhorias-priorizadas.md](./melhorias-priorizadas.md)) |
|------|----------------------------|------------------------------------------------------------------------|
| **Agora** | Pagamento + Rebooking | Persistência de BD + Testes de Frontend |
| **Próximo** | Busca + Notificações | Autenticação + Filtros Server-side |
| **Depois** | Painel + Fidelidade | Observabilidade |

---

*Documento gerado por análise de produto — Galaxium Travels v1.0*
