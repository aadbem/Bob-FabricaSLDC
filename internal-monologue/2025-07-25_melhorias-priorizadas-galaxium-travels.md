# Melhorias priorizadas — Galaxium Travels

## Tarefa
Propor as 5 melhorias mais impactantes para o sistema Galaxium Travels, priorizadas por valor de negócio e esforço, e salvar em `evolucao-projeto/`.

## O que foi feito
- Lido `GT_documentacao.md` (arquitetura completa, modelos, serviços, frontend, testes, seed)
- Lidos monólogos anteriores para contexto de análises passadas
- Identificados os 5 principais pontos de risco/oportunidade:
  1. Autenticação fraca (nome+email sem senha) — risco de segurança crítico
  2. Seed apaga dados no startup — inviabiliza produção
  3. Zero testes de frontend — risco de regressão silenciosa
  4. Filtros de voos apenas no cliente — não escala
  5. Sem logs estruturados nem métricas — observabilidade zero
- Criado `evolucao-projeto/melhorias-priorizadas.md` com: problema, solução, benefício, esforço e dependências para cada melhoria
- Incluído mapa de dependências e roadmap "Agora / Próximo / Depois"

## Decisão de design
Autenticação foi posicionada em "Próximo" (não "Agora") por depender de testes de frontend estáveis — trade-off explicitado no documento.
