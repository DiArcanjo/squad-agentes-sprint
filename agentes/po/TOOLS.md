# TOOLS.md, PO

## Azure DevOps (somente leitura)
- Organização: dianasandbox. Projeto: agentic-sprint.
- Autenticação: o token é injetado no ambiente como AZURE_DEVOPS_EXT_PAT. Você NÃO roda `az devops login`; os comandos `az devops` e `az boards` já autenticam pela variável.
- SEMPRE passe org e projeto explícitos nos comandos, porque o runtime não tem defaults configurados. Exemplo:
  `az boards query --organization https://dev.azure.com/dianasandbox --project agentic-sprint --wiql "..."`
- Escopo read-only. Você consulta work items para contexto e referência. NÃO escreve, cria, altera ou move item nesta fase. Se uma tarefa parecer exigir escrita, PARE e reporte à Diana.
- Se a autenticação falhar ou a variável não estiver presente, PARE e reporte. Nunca busque outra credencial, outro token ou caminho alternativo.

## Refinador (planejado, ainda não construído)
- O Refinador é um agente PLANEJADO — ainda não existe. Quando existir, vai auditar a prontidão dos PBIs que você produz (validador da Definition of Ready): conferir se História de Usuário, Spec, Critérios de Aceite e Cenários de Teste estão presentes e coerentes. A ideia é "um produz, o outro valida".
- Por ora, a validação da DoR é feita pela Diana na revisão. Não conte com um Refinador ativo hoje.

## Registro
Conforme adquirir e usar ferramentas novas, registre aqui o nome, o propósito e o escopo de acesso.