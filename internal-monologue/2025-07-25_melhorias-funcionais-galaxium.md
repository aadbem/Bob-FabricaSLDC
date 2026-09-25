# Melhorias Funcionais — Galaxium Travels

**Data:** 2025-07-25  
**Tarefa:** Propor as 5 melhorias de negócio mais impactantes para o sistema Galaxium Travels, priorizadas por valor de negócio e esforço, gravadas em `evolucao-projeto/melhorias-funcionais.md`.

## O que foi feito

1. Inspecionados os arquivos relevantes do projeto:
   - `models.py` — sem `price_paid` em `Booking`, sem `loyalty_points` em `User`
   - `services/booking.py` — ciclo de vida limitado a `booked → cancelled`, sem rebooking
   - `services/flight.py` — nenhuma busca por texto ou ordenação
   - `server.py` — sem endpoint de checkout, notificações ou stats de usuário
   - `Flights.tsx` — campo `searchTerm` declarado no tipo mas nunca implementado
   - `MyBookings.tsx` — sem agregações ou gamificação
   - `seed.py` — contexto de dados demo e estrutura de tabelas

2. Identificado que já existia `evolucao-projeto/melhorias-priorizadas.md` (foco técnico/segurança). Novo documento criado como complemento focado em **funcionalidades de negócio**.

## 5 melhorias definidas

| # | Melhoria | Esforço | Fase |
|---|----------|---------|------|
| 1 | Fluxo de pagamento simulado + resumo de compra | 1,5 sprints | Agora |
| 2 | Alteração e reemissão de reserva (rebooking) | 1 sprint | Agora |
| 3 | Busca inteligente e recomendações de voo | 1,5 sprints | Próximo |
| 4 | Notificações e confirmação pós-reserva | 1 sprint | Próximo |
| 5 | Painel do passageiro com histórico e fidelidade | 2 sprints | Depois |

## Arquivo gerado

`evolucao-projeto/melhorias-funcionais.md`
