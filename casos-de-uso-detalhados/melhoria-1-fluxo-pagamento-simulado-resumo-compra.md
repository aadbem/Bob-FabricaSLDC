# Casos de Uso Detalhados — Melhoria 1: Fluxo de Pagamento Simulado com Resumo de Compra

> **Plataforma:** Galaxium Travels — sistema de reservas interplanetárias  
> **Melhoria:** #1 — Fluxo de pagamento simulado com resumo de compra  
> **Documento gerado em:** 2025-07-25  
> **Versão do sistema analisada:** v1.0 (`Booking` sem campo `price_paid`, `POST /book` sem etapa de checkout)

---

## Contexto do sistema atual

O fluxo de reserva atual é direto e sem etapa financeira:

```
Usuário seleciona voo → escolhe classe → clica "Book Now"
        → POST /book → booking criado instantaneamente
        → toast "Booking confirmed" (desaparece em ~3s)
```

**Lacunas identificadas no código:**
- [`BookingRequest`](../galaxium-travels/booking_system_frontend/src/types/index.ts:40) não contém `price_paid`
- Modelo [`Booking`](../galaxium-travels/booking_system_backend/models.py:30) não persiste valor cobrado
- Endpoint [`POST /book`](../galaxium-travels/booking_system_backend/server.py:145) cria reserva sem pré-visualização de custo
- [`MyBookings.tsx`](../galaxium-travels/booking_system_frontend/src/pages/MyBookings.tsx) não exibe valor pago

---

## Regras de negócio globais (aplicáveis a todos os UCs)

| ID | Regra |
|----|-------|
| RN-01 | O `price_paid` é calculado como `round(flight.price × seat_class.price_multiplier)` e registrado no momento da confirmação. |
| RN-02 | A reserva só é criada após confirmação explícita do usuário na tela de resumo ("Confirmar e Pagar"). |
| RN-03 | O valor exibido no resumo é o mesmo registrado em `price_paid` — sem variação entre as telas. |
| RN-04 | Classes de assento: `economy` (multiplicador 1.0×), `executive` (1.5×), `galaxium` (2.5×). |
| RN-05 | Um assento só é decrementado em `FlightSeatClass.seats_available` após a confirmação de pagamento, nunca durante a visualização do resumo. |
| RN-06 | Formas de pagamento simuladas aceitas: **Cartão Galáctico** e **Créditos de Viagem**. Nenhuma integração financeira real é processada. |
| RN-07 | O recibo inline deve conter: `booking_id`, valor pago, origem, destino, horário de partida, classe e nome do passageiro. |
| RN-08 | O endpoint `POST /checkout` retorna o resumo sem criar reserva; apenas `POST /book` (ou equivalente de confirmação) persiste a reserva. |

---

## UC-01 — Visualizar Resumo de Compra Antes de Confirmar

**ID:** UC-01  
**Nome:** Visualizar Resumo de Compra Antes de Confirmar  
**Versão:** 1.0

### Atores
- **Primário:** Passageiro (usuário autenticado)
- **Secundário:** Sistema de Precificação (backend — calcula `price_paid` via `price × price_multiplier`)

### Pré-condições
1. Passageiro está autenticado (possui `user_id` em sessão).
2. O voo selecionado existe em `Flight` e possui ao menos 1 assento disponível na classe escolhida (`FlightSeatClass.seats_available ≥ 1`).
3. O passageiro está na tela de seleção de voo com `BookingModal` aberto.

### Fluxo principal
| # | Ator | Ação |
|---|------|------|
| 1 | Passageiro | Seleciona um voo na lista e clica em "Book Now". |
| 2 | Sistema | Abre `BookingModal` na etapa de seleção de classe. |
| 3 | Passageiro | Escolhe a classe de assento (`economy`, `executive` ou `galaxium`). |
| 4 | Passageiro | Clica em "Ver Resumo". |
| 5 | Sistema | Envia `POST /checkout` com `{flight_id, seat_class, user_id}`. |
| 6 | Sistema | Backend calcula `price_paid = round(flight.price × price_multiplier)` e retorna o resumo do pedido. |
| 7 | Sistema | Exibe a tela "Resumo & Pagamento" com: nome do passageiro, voo (origem → destino), horário de partida, classe selecionada e **valor total em destaque**. |
| 8 | Passageiro | Revisa as informações antes de confirmar. |

### Fluxos alternativos

**FA-01A — Passageiro retorna para alterar a classe**
- No passo 7, o passageiro clica em "Voltar".
- O sistema retorna ao passo 2 (seleção de classe) sem ter criado reserva nem decrementado assentos.
- O passageiro pode selecionar uma classe diferente e reiniciar o fluxo a partir do passo 3.

**FA-01B — Passageiro compara classes antes de escolher**
- No passo 3, o sistema exibe uma tabela comparativa com os multiplicadores e valores estimados para cada classe disponível.
- O passageiro visualiza os três preços simultaneamente (`economy`, `executive`, `galaxium`) e então seleciona a classe desejada.
- O fluxo retoma no passo 4.

### Fluxos de exceção

**FE-01A — Voo esgotado entre a seleção e a abertura do modal**
- Condição: entre o passo 1 e o passo 5, outra reserva esgota a última vaga da classe.
- Sistema retorna erro `NO_SEATS_AVAILABLE` na chamada `POST /checkout`.
- Sistema exibe mensagem: _"Esta classe não possui mais vagas disponíveis. Escolha outra classe ou voo."_
- O modal retorna ao passo 2; o passageiro pode escolher outra classe.

**FE-01B — Falha de comunicação com o backend**
- Condição: `POST /checkout` retorna erro de rede (timeout, 5xx).
- Sistema exibe mensagem de erro genérica: _"Não foi possível carregar o resumo. Tente novamente."_
- O modal permanece na etapa atual sem avançar.

### Pós-condições
- Nenhuma reserva foi criada.
- Nenhum assento foi decrementado.
- O passageiro visualizou o resumo com o valor exato que será cobrado (RN-03).

### Regras de negócio aplicáveis
RN-01, RN-03, RN-04, RN-05, RN-08

---

## UC-02 — Selecionar Forma de Pagamento e Confirmar Reserva

**ID:** UC-02  
**Nome:** Selecionar Forma de Pagamento e Confirmar Reserva  
**Versão:** 1.0

### Atores
- **Primário:** Passageiro (usuário autenticado)
- **Secundário:** Backend de Reservas (`POST /book` — cria `Booking` com `price_paid`)

### Pré-condições
1. UC-01 foi concluído com sucesso — resumo de compra está visível.
2. Passageiro visualizou o valor total e o itinerário na tela de resumo.
3. O assento ainda está disponível (não decrementado — RN-05).

### Fluxo principal
| # | Ator | Ação |
|---|------|------|
| 1 | Passageiro | Na tela de resumo, visualiza as formas de pagamento disponíveis. |
| 2 | Passageiro | Seleciona a forma de pagamento: **Cartão Galáctico** ou **Créditos de Viagem**. |
| 3 | Sistema | Destaca visualmente a opção selecionada. |
| 4 | Passageiro | Clica em "Confirmar e Pagar". |
| 5 | Sistema | Envia `POST /book` com `{user_id, name, flight_id, seat_class, price_paid, payment_method}`. |
| 6 | Sistema (backend) | Valida disponibilidade do assento, cria `Booking` com `status = "booked"` e `price_paid` preenchido. |
| 7 | Sistema (backend) | Decrementa `FlightSeatClass.seats_available` em 1. |
| 8 | Sistema | Recebe `booking_id` e retorna confirmação ao frontend. |
| 9 | Sistema | Exibe recibo inline (UC-03). |

### Fluxos alternativos

**FA-02A — Passageiro paga com Créditos de Viagem**
- No passo 2, passageiro seleciona "Créditos de Viagem".
- Sistema exibe saldo disponível simulado (ex.: "⭐ 42.000 créditos disponíveis").
- Se saldo simulado for suficiente, o fluxo continua normalmente a partir do passo 4.
- O campo `payment_method = "credits"` é registrado na reserva.

**FA-02B — Passageiro fecha o modal antes de confirmar**
- Em qualquer passo entre 1 e 4, o passageiro fecha o modal (botão "X" ou tecla `Esc`).
- O sistema não cria reserva e não decrementa assentos.
- O passageiro retorna à lista de voos com o estado anterior preservado.

### Fluxos de exceção

**FE-02A — Assento esgotado entre o resumo e a confirmação**
- Condição: entre o passo 4 e o passo 6, outra reserva ocupa o último assento da classe.
- Backend retorna `ErrorResponse` com `error_code = "NO_SEATS_AVAILABLE"`.
- Sistema exibe: _"Este assento foi reservado por outro passageiro. Escolha outra classe ou voo."_
- Modal retorna à etapa de seleção de classe (UC-01, passo 2).

**FE-02B — Sessão expirada durante o checkout**
- Condição: `user_id` não é mais válido no momento do `POST /book`.
- Backend retorna `error_code = "USER_NOT_FOUND"`.
- Sistema exibe: _"Sua sessão expirou. Faça login novamente para continuar."_
- Modal é fechado; usuário é redirecionado para a tela de login.

**FE-02C — Erro interno do servidor**
- Condição: backend retorna erro 5xx.
- Sistema exibe mensagem genérica sem expor detalhes técnicos.
- Botão "Tentar novamente" repete o passo 5 com os mesmos dados.

### Pós-condições
- Reserva criada em `Booking` com `status = "booked"` e `price_paid` registrado (RN-01).
- `FlightSeatClass.seats_available` decrementado em 1 (RN-05).
- Passageiro visualiza recibo inline com `booking_id` e valor confirmado.

### Regras de negócio aplicáveis
RN-01, RN-02, RN-03, RN-05, RN-06

---

## UC-03 — Visualizar Recibo Inline Após Confirmação

**ID:** UC-03  
**Nome:** Visualizar Recibo Inline Após Confirmação  
**Versão:** 1.0

### Atores
- **Primário:** Passageiro (usuário autenticado)
- **Secundário:** —

### Pré-condições
1. UC-02 concluído com sucesso — reserva criada e `booking_id` retornado.
2. Frontend recebeu a resposta de sucesso do `POST /book`.

### Fluxo principal
| # | Ator | Ação |
|---|------|------|
| 1 | Sistema | Substitui a tela de resumo pelo recibo inline no mesmo modal. |
| 2 | Sistema | Exibe: número da reserva (`booking_id`), nome do passageiro, rota (origem → destino), data/hora de partida, classe, valor pago e forma de pagamento. |
| 3 | Sistema | Exibe botão "Ver Minhas Reservas" e botão "Fechar". |
| 4 | Passageiro | Lê as informações do recibo. |
| 5 | Passageiro | Clica em "Fechar" para voltar à lista de voos **ou** em "Ver Minhas Reservas" para navegar para a página de reservas. |

### Fluxos alternativos

**FA-03A — Passageiro navega para "Minhas Reservas"**
- No passo 5, o passageiro clica em "Ver Minhas Reservas".
- Sistema fecha o modal e navega para [`MyBookings.tsx`](../galaxium-travels/booking_system_frontend/src/pages/MyBookings.tsx).
- A nova reserva aparece no topo da lista de reservas ativas com `price_paid` visível.

**FA-03B — Passageiro copia o número da reserva**
- No passo 4, ao lado do `booking_id`, o sistema exibe um ícone de cópia.
- Ao clicar no ícone, o `booking_id` é copiado para a área de transferência.
- Sistema exibe feedback visual breve: _"Copiado!"_.

### Fluxos de exceção

**FE-03A — Dados do recibo incompletos (resposta parcial do backend)**
- Condição: a resposta do `POST /book` retorna `booking_id` mas sem `flight` details.
- Sistema exibe o `booking_id` e o valor, mas substitui campos ausentes por _"–"_.
- Uma mensagem de aviso é exibida: _"Alguns detalhes estão indisponíveis no momento. Acesse 'Minhas Reservas' para o itinerário completo."_

### Pós-condições
- Passageiro possui o `booking_id` e o valor pago registrados visualmente.
- Reserva está ativa e visível em `GET /bookings/{user_id}`.
- Nenhuma ação adicional é necessária do sistema.

### Regras de negócio aplicáveis
RN-03, RN-07

---

## UC-04 — Visualizar Valor Pago nas Reservas Existentes

**ID:** UC-04  
**Nome:** Visualizar Valor Pago nas Reservas Existentes  
**Versão:** 1.0

### Atores
- **Primário:** Passageiro (usuário autenticado)
- **Secundário:** —

### Pré-condições
1. Passageiro está autenticado.
2. Passageiro possui ao menos uma reserva com `status = "booked"` ou `"completed"`.
3. O campo `price_paid` foi preenchido na reserva (Melhoria 1 implementada).

### Fluxo principal
| # | Ator | Ação |
|---|------|------|
| 1 | Passageiro | Navega para a página "Minhas Reservas" (`MyBookings.tsx`). |
| 2 | Sistema | Carrega `GET /bookings/{user_id}` e exibe a lista de reservas. |
| 3 | Sistema | Em cada `BookingCard`, exibe o campo `price_paid` formatado (ex.: "₲ 1.250"). |
| 4 | Sistema | Na seção de reservas passadas (`pastBookings`), exibe subtotal acumulado dos valores pagos. |
| 5 | Passageiro | Visualiza o histórico financeiro de suas reservas. |

### Fluxos alternativos

**FA-04A — Reserva sem `price_paid` (reserva anterior à Melhoria 1)**
- Condição: reserva criada antes da implementação do campo `price_paid` (valor `null` ou `0`).
- Sistema exibe _"Valor não disponível"_ no lugar do preço no `BookingCard`.
- As reservas sem valor não são incluídas no subtotal acumulado.

**FA-04B — Passageiro filtra reservas por período**
- No passo 2, o passageiro seleciona um filtro de período (ex.: "Último mês").
- O subtotal acumulado é recalculado dinamicamente para refletir apenas as reservas filtradas.

### Fluxos de exceção

**FE-04A — Falha ao carregar reservas**
- Condição: `GET /bookings/{user_id}` retorna erro de rede ou 5xx.
- Sistema exibe mensagem: _"Não foi possível carregar suas reservas. Tente novamente."_
- Botão "Tentar novamente" repete a chamada.

**FE-04B — Passageiro sem reservas**
- Condição: `GET /bookings/{user_id}` retorna lista vazia.
- Sistema exibe estado vazio: _"Você ainda não tem reservas. Explore nossos voos!"_ com link para a página de voos.

### Pós-condições
- Passageiro visualizou os valores pagos em todas as suas reservas.
- Nenhuma alteração de dados foi realizada.

### Regras de negócio aplicáveis
RN-01, RN-03, RN-07

---

## UC-05 — Calcular e Registrar Preço no Momento da Reserva (Backend)

**ID:** UC-05  
**Nome:** Calcular e Registrar Preço no Momento da Reserva (Backend)  
**Versão:** 1.0

> **Natureza:** Caso de uso de sistema (sem interação direta do usuário final) — documenta o comportamento esperado do backend para garantir consistência financeira.

### Atores
- **Primário:** Backend de Reservas (serviço `booking.py`)
- **Secundário:** Banco de Dados (`Booking`, `Flight`, `FlightSeatClass`)

### Pré-condições
1. `POST /book` recebeu payload válido: `{user_id, name, flight_id, seat_class, price_paid}`.
2. `flight_id` existe em `Flight`.
3. `seat_class` existe em `FlightSeatClass` para o `flight_id` informado.
4. `FlightSeatClass.seats_available ≥ 1`.
5. `user_id` existe em `User` e `name` confere.

### Fluxo principal
| # | Ator | Ação |
|---|------|------|
| 1 | Backend | Recupera `Flight` pelo `flight_id` → obtém `flight.price`. |
| 2 | Backend | Recupera `FlightSeatClass` pelo `flight_id` + `seat_class` → obtém `price_multiplier`. |
| 3 | Backend | Calcula `price_paid = round(flight.price × price_multiplier)`. |
| 4 | Backend | Verifica que `price_paid` no payload corresponde ao calculado (tolerância: ± 1 unidade para arredondamentos). |
| 5 | Backend | Cria objeto `Booking` com: `user_id`, `flight_id`, `seat_class`, `status = "booked"`, `booking_time = utcnow()`, `price_paid`. |
| 6 | Backend | Decrementa `FlightSeatClass.seats_available` em 1 (atomicamente via transação). |
| 7 | Backend | Persiste no banco de dados e retorna `BookingOut` com todos os campos, incluindo `price_paid`. |

### Fluxos alternativos

**FA-05A — `price_paid` não informado no payload (cliente legado)**
- Condição: payload sem `price_paid` (compatibilidade com versão anterior do frontend).
- Backend calcula automaticamente `price_paid = round(flight.price × price_multiplier)` e prossegue.
- Log de aviso é registrado: `"price_paid not provided by client — calculated server-side"`.

**FA-05B — Múltiplas reservas simultâneas para o último assento**
- Condição: dois requests concorrentes para o mesmo `flight_id` + `seat_class` com 1 assento disponível.
- O banco de dados resolve via lock de transação (SELECT FOR UPDATE ou equivalente SQLAlchemy).
- Apenas uma reserva é criada; a segunda retorna `error_code = "NO_SEATS_AVAILABLE"`.

### Fluxos de exceção

**FE-05A — Divergência de preço entre frontend e backend**
- Condição: `price_paid` no payload difere do calculado em mais de 1 unidade (possível manipulação ou dessincronização de dados).
- Backend rejeita a reserva com `error_code = "PRICE_MISMATCH"`.
- Backend usa seu próprio cálculo como fonte da verdade (RN-01).

**FE-05B — Erro de persistência no banco de dados**
- Condição: falha na operação `db.commit()`.
- Backend realiza rollback completo (assento não decrementado, reserva não criada).
- Retorna HTTP 500 com mensagem genérica ao frontend.

**FE-05C — `FlightSeatClass` não encontrado para a classe solicitada**
- Condição: `seat_class` válida (`economy`/`executive`/`galaxium`) mas não cadastrada para o voo específico.
- Backend retorna `error_code = "SEAT_CLASS_NOT_FOUND"`.
- Nenhuma modificação é feita no banco.

### Pós-condições
- `Booking` criado com `price_paid` preenchido e não nulo.
- `FlightSeatClass.seats_available` decrementado em exatamente 1.
- Transação atômica: impossível ter reserva sem decremento ou decremento sem reserva.
- `booking_id` retornado ao frontend para exibição no recibo (UC-03).

### Regras de negócio aplicáveis
RN-01, RN-02, RN-04, RN-05, RN-08

---

## UC-06 — Abortar Checkout e Retornar à Lista de Voos

**ID:** UC-06  
**Nome:** Abortar Checkout e Retornar à Lista de Voos  
**Versão:** 1.0

### Atores
- **Primário:** Passageiro (usuário autenticado)
- **Secundário:** —

### Pré-condições
1. Passageiro iniciou o fluxo de checkout (UC-01 em andamento — resumo visível ou seleção de pagamento ativa).
2. Nenhuma reserva foi criada ainda.

### Fluxo principal
| # | Ator | Ação |
|---|------|------|
| 1 | Passageiro | Na tela de resumo ou seleção de pagamento, decide não prosseguir. |
| 2 | Passageiro | Clica em "Cancelar" ou fecha o modal (botão "X" / tecla `Esc`). |
| 3 | Sistema | Fecha o modal sem realizar nenhuma chamada ao backend. |
| 4 | Sistema | Retorna à lista de voos com o estado de filtros e ordenação preservado. |
| 5 | Sistema | O voo permanece disponível com o mesmo número de assentos (nenhum decremento ocorreu — RN-05). |

### Fluxos alternativos

**FA-06A — Passageiro retorna etapa por etapa (botão "Voltar")**
- No passo 2, o passageiro clica em "Voltar" (em vez de fechar diretamente).
- Sistema retorna para a etapa anterior do modal: resumo → seleção de classe → lista de voos.
- Dados preenchidos na etapa anterior são preservados (classe selecionada ainda está marcada).

**FA-06B — Passageiro reinicia com outro voo**
- Após fechar o modal no passo 3, o passageiro imediatamente seleciona outro voo.
- O sistema abre um novo `BookingModal` com os dados zerados para o novo voo.
- Não há interferência com a tentativa anterior (nenhum estado foi persistido).

### Fluxos de exceção

**FE-06A — Fechamento acidental do modal**
- Condição: passageiro clica fora do modal involuntariamente.
- Sistema exibe diálogo de confirmação: _"Deseja realmente sair? Seu resumo será descartado."_ com botões "Sim, sair" e "Continuar comprando".
- Se confirmar, o modal é fechado (retorna ao passo 3).
- Se cancelar, o modal permanece aberto na mesma etapa.

### Pós-condições
- Nenhuma reserva foi criada.
- Nenhum assento foi decrementado (RN-05).
- O passageiro está na lista de voos com estado de UI preservado.

### Regras de negócio aplicáveis
RN-02, RN-05

---

## Diagrama de fluxo — Jornada de pagamento (visão integrada dos UCs)

```
[Lista de Voos]
      │
      ▼ Seleciona voo + classe
[UC-01: Ver Resumo]  ─── "Voltar" ──► [Lista de Voos]
      │                               ▲
      │ "Ver Resumo"                  │
      ▼                               │ FE-01A / FE-01B
[POST /checkout]                      │
      │                         [Exibe erro]
      ▼
[Tela de Resumo & Pagamento]
      │
      ├── "Cancelar" / "X" ──► [UC-06: Abortar Checkout] ──► [Lista de Voos]
      │
      ▼ Seleciona forma de pagamento
[UC-02: Confirmar e Pagar]
      │
      ▼ POST /book
[UC-05: Backend — Calcular e Registrar]
      │
      ├── Erro ──► [Exibe mensagem] ──► Retry / Volta à seleção
      │
      ▼ Sucesso
[UC-03: Recibo Inline]
      │
      ├── "Fechar" ──► [Lista de Voos]
      └── "Ver Minhas Reservas" ──► [UC-04: MyBookings com price_paid]
```

---

## Cobertura dos fluxos por caso de uso

| Fluxo | UC responsável |
|-------|---------------|
| Visualização do resumo antes de confirmar | UC-01 |
| Seleção de forma de pagamento e confirmação | UC-02 |
| Exibição do recibo pós-confirmação | UC-03 |
| Histórico financeiro em "Minhas Reservas" | UC-04 |
| Cálculo e persistência do `price_paid` (backend) | UC-05 |
| Abandono do checkout sem criar reserva | UC-06 |

---

*Documento gerado com base na análise do código fonte v1.0 — Galaxium Travels*  
*Referências: [`models.py`](../galaxium-travels/booking_system_backend/models.py), [`booking.py`](../galaxium-travels/booking_system_backend/services/booking.py), [`server.py`](../galaxium-travels/booking_system_backend/server.py), [`types/index.ts`](../galaxium-travels/booking_system_frontend/src/types/index.ts)*
