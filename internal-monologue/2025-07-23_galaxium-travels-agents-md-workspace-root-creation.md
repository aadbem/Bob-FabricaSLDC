# Resumo: Aprimoramento e Criação de AGENTS.md na Raiz do Workspace

## O que foi feito

As orientações para agentes de IA no projeto `galaxium-travels` foram aprimoradas e devidamente estabelecidas na raiz real do workspace, garantindo compatibilidade imediata com novas sessões e inicializações de agentes na raiz do repositório:

1. **Investigação de Stack e Extensões (IBM Z / COBOL)**:
   - Realizada varredura detalhada no repositório por arquivos COBOL, JCL, PL/I, Assembler ou REXX, bem como arquivos de dicionário de dados (`bobz/DD.json` ou similar).
   - Confirmado que o repositório **não** é um projeto de mainframe (IBM Z), tratando-se exclusivamente de um monorepo Web moderno de turismo espacial (`galaxium-travels/` contendo Python FastAPI no backend e React 19/TypeScript no frontend).

2. **Criação e Refinamento do `AGENTS.md` (Principal na Raiz)**:
   - Criado o arquivo `AGENTS.md` diretamente sob a raiz do workspace (`/Users/albertobem/Softwares/Bob_Labs/Bob-Fabrica/AGENTS.md`).
   - Prefixado com o cabeçalho obrigatório: `# AGENTS.md\n\nThis file provides guidance to agents when working with code in this repository.`
   - Aplicada a **limpeza agressiva** de dados óbvios (removidas instruções genéricas de pacotes, configurações padrão de frameworks ou comandos básicos do npm/pip).
   - Documentados os desvios surpreendentes do projeto (retornos com HTTP 200 contendo `ErrorResponse` em falhas de negócio, a necessidade do type guard `isErrorResponse` no frontend, e a tabela `FlightSeatClass` como única fonte da verdade de vagas).
   - Incluídos os caminhos corretos e comandos para execução de testes via pytest do ambiente virtual (`.venv/bin/pytest`), evitando erros por falta de dependências globais.

3. **Criação das Regras Específicas por Modo na Raiz (`.bob/rules-*`)**:
   - Criadas as pastas `.bob/rules-agent/`, `.bob/rules-ask/` e `.bob/rules-plan/` diretamente no diretório raiz do workspace.
   - Criados os respectivos arquivos `AGENTS.md` de modo focado:
     - **Modo Agent** (`.bob/rules-agent/AGENTS.md`): Focado em regras críticas de escrita de código (imports planos, uso de `model_validate`, tratamento de exceções em MCP, type guard de erros no frontend, requisitos de JSDoc em funções públicas).
     - **Modo Ask** (`.bob/rules-ask/AGENTS.md`): Focado em contexto não-óbvio de navegação e busca (duas portas, protocolo FastMCP em `/mcp`, tipagem unificada em `src/types/index.ts`, volatilidade do `booking.db` por seed e estilos customizados de Tailwind).
     - **Modo Plan** (`.bob/rules-plan/AGENTS.md`): Focado em restrições arquiteturais e acoplamento (colocalização de processos FastAPI/FastMCP, falta de migrações em banco via Alembic, ausência de segurança/sessão e monkeypatching essencial nos testes).

4. **Validação de Comandos (Dry Run)**:
   - Validado o teste específico do backend usando o binário de venv correspondente, com execução e resultado `PASSED` com sucesso total.
   - Analisados os alertas e status do lint no frontend.
