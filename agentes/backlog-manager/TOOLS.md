# TOOLS.md, Backlog Manager

## Azure DevOps (leitura e escrita, só sandbox)
- Organização: dianasandbox. Projeto: agentic-sprint.
- Autenticação: o token é injetado no ambiente como AZURE_DEVOPS_EXT_PAT. Você NÃO roda `az devops login`; os comandos `az boards` já autenticam pela variável.
- SEMPRE passe org e projeto explícitos nos comandos, porque o runtime não tem defaults.
- Você tem escrita, mas SÓ na dianasandbox/agentic-sprint. Nunca escreva na technecloud nem em qualquer outro lugar. Se um alvo apontar para fora da sandbox, PARE e reporte.
- Se a autenticação falhar ou a variável não estiver presente, PARE e reporte. Nunca busque outra credencial ou caminho alternativo.

## Mapeamento de tipos (JSON -> Azure)
- epic -> tipo "Epic"
- feature -> tipo "Feature"
- pbi -> tipo "Product Backlog Item"
- task_dev -> tipo "Task"

## Mapeamento de campos (JSON -> Azure)
- titulo -> System.Title (texto puro)
- descricao (Epic/Feature) -> System.Description (campo HTML)
- historia_usuario (PBI) -> campo de História de Usuário do PBI (campo HTML)
- description (PBI) -> System.Description (Contexto + AS-IS + TO-BE) (campo HTML)
- acceptance_criteria (PBI) -> Microsoft.VSTS.Common.AcceptanceCriteria (critérios + cenários de teste juntos) (campo HTML)
- tasks_dev (lista) -> cada item vira um work item Task: `titulo` -> System.Title; `descricao` -> System.Description (campo HTML)

Assigned To (System.AssignedTo): NUNCA preencha. Deixe vazio em todos os itens.

## Renderização HTML dos campos de texto longo (OBRIGATÓRIO)
`System.Description`, `Microsoft.VSTS.Common.AcceptanceCriteria` e o campo de História de Usuário são campos **HTML** no Azure. O JSON traz esses valores em texto puro com quebras de linha (`\n`). Se você gravar o texto cru, o Azure **colapsa as quebras** e o conteúdo gruda numa linha só (bug observado no critério de aceite). Antes de gravar, **renderize como HTML**:
1. Escape os caracteres especiais, nesta ordem: `&` -> `&amp;`, depois `<` -> `&lt;` e `>` -> `&gt;`.
2. Converta cada quebra de linha `\n` em `<br>`.

Aplique a: `description` e `acceptance_criteria` (PBI), `historia_usuario` (PBI) e `descricao` (Task). NÃO aplique aos títulos (`System.Title` é texto puro).

## Comandos de referência (az boards)
Criar um item (exemplo genérico):
`az boards work-item create --organization https://dev.azure.com/dianasandbox --project agentic-sprint --type "Product Backlog Item" --title "<titulo>" --fields "System.Description=<...>" "Microsoft.VSTS.Common.AcceptanceCriteria=<...>"`

Criar uma Task (com título e descrição; a descrição já renderizada em HTML):
`az boards work-item create --organization https://dev.azure.com/dianasandbox --project agentic-sprint --type "Task" --title "<titulo>" --fields "System.Description=<descricao em HTML>"`

Vincular a um parent (após criar, com os dois IDs):
`az boards work-item relation add --organization https://dev.azure.com/dianasandbox --id <id_filho> --relation-type parent --target-id <id_pai>`

Capture o ID retornado na criação de cada item para usar como parent dos filhos.

## Regras de segurança
- Escrita só na sandbox. Nunca produção.
- Assigned To sempre vazio.
- Uma árvore por aprovação. Não duplique.
- Em qualquer falha, PARE e reporte com os IDs já criados. Não continue às cegas.