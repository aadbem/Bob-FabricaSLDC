# Resumo — Casos de uso detalhados: Melhoria 1 (Pagamento Simulado)

**Data:** 2025-07-25  
**Arquivo gerado:** `casos-de-uso-detalhados/melhoria-1-fluxo-pagamento-simulado-resumo-compra.md`

## O que foi feito

Gerados 6 casos de uso detalhados para a Melhoria 1 do Galaxium Travels ("Fluxo de pagamento simulado com resumo de compra"), com base em:
- Leitura de `evolucao-projeto/melhorias-funcionais.md` (especificação da melhoria)
- Leitura de `models.py`, `services/booking.py`, `server.py` e `types/index.ts` (comportamento atual do sistema)

## Casos de uso gerados

| ID | Nome |
|----|------|
| UC-01 | Visualizar Resumo de Compra Antes de Confirmar |
| UC-02 | Selecionar Forma de Pagamento e Confirmar Reserva |
| UC-03 | Visualizar Recibo Inline Após Confirmação |
| UC-04 | Visualizar Valor Pago nas Reservas Existentes |
| UC-05 | Calcular e Registrar Preço no Momento da Reserva (Backend) |
| UC-06 | Abortar Checkout e Retornar à Lista de Voos |

## Estrutura de cada UC

Cada caso de uso contém: ID, nome, versão, atores (primário/secundário), pré-condições, fluxo principal (tabela), mínimo 2 fluxos alternativos, fluxos de exceção, pós-condições e regras de negócio aplicáveis.

## Lacunas mapeadas do sistema atual

- `Booking` sem campo `price_paid`
- `POST /book` cria reserva sem etapa de checkout
- `BookingRequest` (frontend) sem `price_paid`
- `MyBookings.tsx` sem exibição de valor pago

## Decisões de design documentadas

- 8 regras de negócio globais (RN-01 a RN-08) definidas no cabeçalho
- Diagrama de fluxo ASCII integrando todos os UCs
- Tabela de cobertura mapeando fluxo → UC responsável
