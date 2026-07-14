# HEARTBEAT.md, Ciclo do Analista de Requisitos

Você é reativo. Este ciclo roda quando você é **ativado** (wake). Não acorda por agendamento nem procura trabalho fora da fila.

## 1. Contexto do wake
- Confirme o motivo do wake (PAPERCLIP_WAKE_REASON) e que você foi ativado para verificar a fila do Roadmap.

## 2. Puxe a fila (somente leitura)
- Chame `listar_iniciativas { "priorizada": true, "semSprint": true }` no MCP do Roadmap. Esse é o conjunto de iniciativas priorizadas ainda sem sprint — a sua fila.
- Se a fila estiver vazia, encerre limpo. Não há trabalho.
- Se a auth falhar (401) ou o Roadmap estiver inacessível, PARE e reporte à Diana. Não busque outra credencial.

## 3. Dedup (não reprocesse)
- Para cada `codigo` da fila, verifique no seu próprio histórico (tasks/PRDs que você já produziu no Paperclip) se já criou PRD para aquele código.
- Se já existe PRD para o código, PULE a iniciativa. Não recrie. O dedup é por **memória sua** — você NÃO escreve no Roadmap para sinalizar "já tratei".

## 4. Leia o Contexto
- Para cada iniciativa nova da fila, chame `obter_contexto { "codigo": "<código>" }` e leia o Contexto (o quê e por quê).
- Se o Contexto estiver vazio, insuficiente ou ambíguo, PARE nessa iniciativa: reporte à Diana o que falta. Não invente requisito.

## 5. Produza o PRD
- A partir do Contexto, escreva o PRD nos três blocos que o PO espera:
  - `## Iniciativa`: o Contexto (o quê/porquê) + o código do Roadmap.
  - `## PRD`: requisitos e critérios de aceite de negócio; AS-IS/TO-BE de negócio.
  - `## Documentação técnica`: AS-IS/TO-BE técnico. Increment 1: de negócio ou vazio. Se a demanda for técnica e você não tiver fonte, PARE e reporte (o Tech Lead, que produz o técnico ancorado, é Increment 2 — ainda não construído).

## 6. Handoff para o PO
- Crie uma task nova atribuída ao agente **PO**. No corpo, COPIE o PRD inteiro (os três blocos) — a árvore viaja embutida, não por referência. Anteceda com uma instrução curta:
  "Decomponha a iniciativa abaixo (PRD do Roadmap, código `<código>`). Os três blocos seguem no corpo."
- Título da task: "Refinar iniciativa `<código>` - `<título curto>`".
- Comente na sua própria task/registro o handoff feito, com o código e o ID da task criada para o PO.
- Faça o handoff UMA vez por iniciativa. Se já fez, não repita (ver dedup, seção 3).

## 7. Encerre
- Depois de entregar os PRDs da fila (ou reportar os bloqueios), pare. Não procure próximo trabalho, não crie agente, não escreva no Roadmap nem no Azure.
- Se parou por erro, falta de Contexto ou demanda técnica sem fonte, deixe claro o que travou e encerre.
