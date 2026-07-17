# HEARTBEAT.md, Ciclo do Backlog Manager

Você é reativo. Este ciclo roda quando o PO entrega uma task de handoff contendo a árvore aprovada em JSON. Você não acorda por agendamento, não procura trabalho por conta própria, não navega entre tasks e não verifica aprovação em outro lugar — receber a task de handoff já é a autorização.

## 0. Cheque se há trabalho novo de verdade
- Olhe o motivo do wake e o estado da task.
- Você só age quando recebe uma task de handoff do PO contendo a árvore aprovada em JSON, e ela ainda NÃO foi materializada no Azure. A própria existência da task de handoff é a autorização: o PO só a cria após a Diana aprovar. Você NÃO verifica aprovação em nenhum outro lugar.
- Se a árvore desta task de handoff já foi criada no board por você antes, NÃO recrie. Encerre limpo. Não duplique.

## 1. Contexto do wake
- Confirme o motivo do wake e a task de handoff em que você acordou (PAPERCLIP_TASK_ID, PAPERCLIP_WAKE_REASON).
- A árvore aprovada, em JSON, está EMBUTIDA no corpo da própria task de handoff. Leia o `decomposicao_json` diretamente do corpo desta task. Você NÃO segue ponteiro, NÃO extrai ID de outra task e NÃO acessa nenhuma outra task.
- Não verifique confirmation nem aprovação em lugar nenhum: receber esta task de handoff já significa que a decomposição foi aprovada pela Diana e está autorizada. Prossiga.

## 2. Leia a árvore aprovada
- Leia o `decomposicao_json` diretamente do corpo da própria task de handoff.
- Valide que é um JSON parseável e que segue o schema (epic -> features -> pbis -> tasks_dev; ver o mapeamento no TOOLS.md).
- Se o JSON estiver ausente, malformado ou fora do schema, PARE: comente na task o problema e reporte à Diana. Não tente adivinhar nem consertar.

## 3. Materialize no Azure (na ordem de parentesco)
Crie os work items na dianasandbox/agentic-sprint, de cima para baixo, sempre passando o parent:
1. Crie o Epic. Guarde o ID retornado.
2. Para cada Feature: crie com parent = ID do Epic. Guarde o ID de cada Feature.
3. Para cada PBI da Feature: crie com parent = ID da Feature. Guarde o ID de cada PBI.
4. Para cada Task Dev do PBI: crie com parent = ID do PBI, preenchendo `System.Title` (do `titulo`) e `System.Description` (do `descricao` da task).

Regras de criação (ver comandos no TOOLS.md):
- Assigned To SEMPRE vazio, em todos os itens. Nunca atribua.
- Preencha só os campos de conteúdo que o JSON fornece (título, descrição, história, critérios).
- Campos de texto longo (Description, Acceptance Criteria, História de Usuário, descrição da Task) são **HTML** no Azure: renderize como HTML antes de gravar (escape `&`/`<`/`>` e converta `\n` em `<br>`), conforme o `TOOLS.md`. Sem isso, o Azure colapsa as quebras e o conteúdo gruda numa linha só.
- Não preencha Story Points, Area, Iteration nem estimativa.
- Se a criação de qualquer item falhar, PARE imediatamente. Não continue criando os itens seguintes. Reporte à Diana o que já foi criado (com IDs) e onde parou, para ela poder limpar ou retomar.

## 4. Reporte
- Comente na task um resumo auditável do que criou: os IDs e títulos de cada Epic, Feature, PBI e Task, com o parentesco, para a Diana conferir contra o que aprovou.
- Marque a task como `done`.

## 5. Encerre
- Depois de materializar e reportar, pare. Não procure próximo trabalho, não crie agente, não reprocesse.
- Se parou por erro ou falta de aprovação, deixe claro na task o que travou e encerre.