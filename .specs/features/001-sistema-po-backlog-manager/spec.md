# Sistema PO + Backlog Manager — Especificação

> **Natureza deste documento**: especificação de um sistema **já construído e em operação** dentro do
> Paperclip (orquestrador de agentes em `localhost:3100`). Não há código-fonte: o "artefato" são arquivos
> de instrução de agentes (markdown) e configurações no Paperclip. Os requisitos abaixo descrevem o
> **comportamento observável** dos agentes. A validação é **funcional e manual** na UI do Paperclip
> (não há teste automatizado). Formato adaptado da skill `tlc-spec-driven`.

## Problem Statement

Transformar uma iniciativa priorizada em um backlog refinado no Azure Boards é um trabalho manual, repetitivo
e sujeito a inconsistência de padrão. O objetivo é automatizar a decomposição (Iniciativa → Epic → Feature →
PBI → Task Dev) e a materialização no Azure, preservando um **gate de aprovação humano** no meio e uma
**separação estrita de poderes** (quem lê ≠ quem escreve), para conter risco.

## Goals

- [x] Decompor automaticamente uma iniciativa em uma árvore Epic → Feature → PBI → Task Dev, com cada PBI na Definition of Ready.
- [x] Produzir dois artefatos: `decomposicao` (markdown, para revisão humana) e `decomposicao_json` (estruturado, para consumo do Backlog Manager).
- [x] Materializar a árvore inteira no Azure Boards (org `dianasandbox`, projeto `agentic-sprint`) na ordem de parentesco, com Assigned To vazio.
- [x] Garantir gate humano obrigatório (confirmation `ACCEPTED`) antes de qualquer escrita no Azure.
- [x] Separar poderes: PO read-only (`pmo-read`), Backlog Manager write-only na sandbox (`board-write`).

## Out of Scope

Explicitamente excluído. Documentado para prevenir scope creep.

| Item | Motivo |
| ---- | ------ |
| Criação de Task Teste, Plano de Teste, Teste, Issue, DT, Bug, Task RT | São de fase downstream posterior; o PO cria **apenas** Task Dev. |
| Escrita em produção (`technecloud`) | `board-write` é escopada só à org `dianasandbox` (AD-003). |
| Preenchimento de Assigned To, Story Points, Area, Iteration | Atribuição/estimativa é decisão humana pós-poker (AD-005). |
| Refinador-agente (validador da Definition of Ready) | Próximo agente, ainda não construído (questão em aberto no STATE.md). |
| Conexão roadmap → Paperclip | Coberta pela feature 002 (esqueleto). |
| Wake pós-aprovação 100% confiável via Routines/Goals | Ainda não investigado (questão em aberto no STATE.md). |

---

## Assumptions & Open Questions

Este spec documenta um sistema existente; as ambiguidades abaixo são pendências reais herdadas da construção,
não escolhas de design a fazer agora. Nada foi inventado — o que não estava no briefing está marcado como aberto.

| Assumção / decisão | Situação atual | Racional | Confirmado? |
| ------------------ | -------------- | -------- | ----------- |
| Wake do PO após aprovação | Não é 100% confiável; *Run Heartbeat* manual força quando trava | Mecanismo nativo (Routines/Goals) não investigado | n |
| Cerca de saída formal (#3) | Não implementada além do confirmation atual | Confirmation (AD-004) é o único gate hoje | n |
| `PAPERCLIP_API_KEY` no ambiente dos agentes | Presente; agentes a descobrem sozinhos; dá escrita interna no Paperclip | A conter, ainda não contida | n |
| Datas exatas das decisões AD-001..AD-009 | Registradas com a data de documentação (2026-07-13) | Datas originais não constam do briefing; não inventadas | n |
| Status do Refinador (validador da DoR) | Resolvido: agente **planejado, não construído**. `po/TOOLS.md` ajustado para o tempo prospectivo; hoje a validação da DoR é feita pela Diana na revisão | Diana confirmou que o Refinador não existe ainda; o texto no presente era impreciso | **y** |
| Saída declarada no `po/AGENTS.md` | Resolvido: seção "Saída" agora cita os dois documentos (`decomposicao` + `decomposicao_json`), consistente com `HEARTBEAT.md §4` e `DECOMPOSICAO.md` | Alinhamento de documentação (POBM-04) | **y** |

**Open questions:** ver seção "Questões em aberto" do `.specs/STATE.md` (fonte única). As linhas acima espelham as que afetam diretamente o comportamento desta feature.

---

## User Stories

### P1: Decomposição da iniciativa pelo PO ⭐ MVP

**User Story**: Como Product Owner de Refinamento (agente PO), quero decompor uma iniciativa priorizada em uma árvore Epic → Feature → PBI → Task Dev com cada PBI na Definition of Ready, para que o backlog fique refinado e pronto para materialização.

**Why P1**: É a entrada do sistema. Sem a decomposição não há nada para revisar nem materializar.

**Acceptance Criteria**:

1. WHEN a Diana entrega uma iniciativa como task no Paperclip com os três blocos `## Iniciativa`, `## PRD` e `## Documentação técnica` THEN o PO SHALL decompô-la em uma árvore Iniciativa → Epic → Feature → PBI → Task Dev.
2. WHEN o PO decompõe THEN cada PBI SHALL conter a Definition of Ready completa em quatro partes: (1) História de Usuário "Como ator, quero objetivo, para benefício"; (2) Spec com Contexto, AS-IS e TO-BE; (3) Critérios de Aceite em Dado/Quando/Então; (4) Cenários de Teste.
3. WHEN o PO cria itens de trabalho THEN SHALL criar **apenas** Task Dev — nunca Task Teste, Plano de Teste, Teste, Issue, DT, Bug ou Task RT.
4. WHEN o PO conclui a decomposição THEN SHALL salvar dois documentos: `decomposicao` (markdown, para revisão) e `decomposicao_json` (JSON estruturado conforme `contratos/decomposicao-json-schema.md`).
5. WHEN o PO acessa o Azure THEN SHALL fazê-lo apenas em modo leitura (credencial `pmo-read`) — sem qualquer escrita.
6. WHEN algum dos três blocos essenciais falta ou está ambíguo THEN o PO SHALL parar, marcar a task como `blocked`, reportar à Diana e NÃO decompor com lacuna (não inventar conteúdo). _(especificado em `po/AGENTS.md` e `po/HEARTBEAT.md`)_

**Independent Test** (funcional, manual na UI do Paperclip): entregar uma iniciativa de teste com os três blocos e verificar que o PO produz os dois documentos, que a árvore tem os quatro níveis, que cada PBI tem as quatro partes da DoR e que nenhum item foi escrito no Azure.

---

### P2: Gate humano de aprovação

**User Story**: Como Diana (humana), quero revisar e aprovar a decomposição antes de qualquer escrita, para que nada entre no Azure sem autorização.

**Why P2**: Ponto de controle de risco. Sem ele a separação de poderes perde a âncora humana.

**Acceptance Criteria**:

1. WHEN a decomposição está pronta THEN o sistema SHALL aguardar uma revisão humana antes de materializar qualquer item no Azure.
2. WHEN a Diana aprova a decomposição (confirmation `ACCEPTED`) THEN esse aceite SHALL funcionar como a autorização de escrita.
3. WHEN não há aprovação THEN o sistema SHALL NOT escrever nada no Azure.

**Independent Test** (funcional, manual): deixar uma decomposição pronta sem aprovar e confirmar que nada é escrito no Azure; aprovar e confirmar que o fluxo avança.

---

### P2: Handoff automático PO → Backlog Manager

**User Story**: Como agente PO, quero, ao ser aprovado, criar automaticamente uma task para o Backlog Manager com o `decomposicao_json` copiado inteiro no corpo, para que a materialização siga sem repasse manual.

**Why P2**: Liga o gate humano à materialização; é o que torna o fluxo "ponta a ponta".

**Acceptance Criteria**:

1. WHEN o PO detecta a aprovação (confirmation `ACCEPTED`) THEN SHALL criar uma task atribuída ao Backlog Manager com o conteúdo inteiro do `decomposicao_json` copiado no corpo da task (a árvore viaja embutida, não por referência).
2. WHEN o handoff ocorre THEN a instrução que o dispara SHALL residir no `HEARTBEAT.md` do PO (AD-006).
3. WHEN o Paperclip não reacorda o PO de forma confiável após a aprovação THEN um *Run Heartbeat* manual SHALL poder forçar o wake (comportamento conhecido, ver STATE.md).

**Independent Test** (funcional, manual): aprovar uma decomposição e verificar que surge a task para o Backlog Manager com o `decomposicao_json` copiado no corpo; se não surgir, disparar *Run Heartbeat* e confirmar que aparece.

---

### P1: Materialização no Azure pelo Backlog Manager ⭐ MVP

**User Story**: Como Backlog Manager (agente escritor), quero ler o `decomposicao_json` e materializar a árvore inteira no Azure Boards na ordem de parentesco, para que o backlog refinado exista de fato no board.

**Why P1**: É a saída do sistema — sem ela a decomposição não vira backlog real.

**Acceptance Criteria**:

1. WHEN o Backlog Manager recebe a task de handoff THEN SHALL ler o `decomposicao_json` e materializar a árvore na org `dianasandbox`, projeto `agentic-sprint`.
2. WHEN materializa THEN SHALL criar os itens de cima para baixo na ordem de parentesco (Epic → Feature → PBI → Task), passando o ID do pai como parent do filho.
3. WHEN cria qualquer item THEN SHALL deixar Assigned To vazio em todos (AD-005).
4. WHEN mapeia tipos THEN `epic`→"Epic", `feature`→"Feature", `pbi`→"Product Backlog Item", `task_dev`→"Task".
5. WHEN escreve no Azure THEN SHALL usar exclusivamente a credencial `board-write`, escopada a *Work Items Read & Write* na org `dianasandbox` apenas, com expiração curta (AD-003) — nunca produção (`technecloud`).
6. WHEN o `decomposicao_json` está ausente, malformado ou fora do schema THEN o Backlog Manager SHALL parar, comentar o problema na task e reportar à Diana — sem adivinhar nem consertar. _(especificado em `backlog-manager/HEARTBEAT.md` §2)_
7. WHEN a criação de qualquer item falha THEN o Backlog Manager SHALL parar imediatamente, NÃO continuar criando os itens seguintes, e reportar os IDs já criados e onde parou (falha parcial auditável). _(especificado em `backlog-manager/HEARTBEAT.md` §3, `SOUL.md`, `TOOLS.md`)_
8. WHEN a árvore desta aprovação já foi materializada por ele antes THEN o Backlog Manager SHALL NOT recriá-la (idempotência) e SHALL encerrar limpo, sem duplicar. _(especificado em `backlog-manager/HEARTBEAT.md` §0)_
9. WHEN o Backlog Manager é acordado por uma task de handoff THEN SHALL ler o `decomposicao_json` embutido no corpo da própria task de handoff e materializar a partir dele — SEM seguir ponteiro, SEM acessar outra task e SEM verificar aprovação em outro lugar. A existência da task de handoff é a autorização (o PO só a cria após a aprovação da Diana). _(especificado em `backlog-manager/HEARTBEAT.md` §0–2, alinhado ao `po/HEARTBEAT.md` §5; ver AD-010)_

**Independent Test** (funcional, manual): a partir de um `decomposicao_json` conhecido, rodar o Backlog Manager e conferir no Azure Boards que a árvore inteira aparece com parentesco correto, tipos corretos e Assigned To vazio em todos.

---

### P2: Contenção e separação de credenciais

**User Story**: Como responsável pela segurança do sistema, quero que cada agente carregue só o privilégio do seu papel e que a contenção venha de permissão + run policy, para conter o estrago mesmo com skip permissions ligado.

**Why P2**: É a garantia de segurança transversal do sistema; sustenta AD-002, AD-003, AD-007, AD-009.

**Acceptance Criteria**:

1. WHEN os agentes são configurados THEN o PO SHALL ter só `pmo-read` e o Backlog Manager só `board-write`, em secrets separados (AD-002).
2. WHEN uma credencial é injetada THEN SHALL ser via secret do Paperclip como env var `AZURE_DEVOPS_EXT_PAT` — não via `az login`, que não alcança o runtime do adapter (AD-007).
3. WHEN os agentes rodam THEN as cercas de contenção SHALL estar ativas: heartbeat sem intervalo (só wake on demand), *Continue after max-turn* OFF, *create-agents* OFF, *create/import-skills* OFF, *max concurrent* = 1 (AD-009).
4. WHEN o método de decomposição é empacotado THEN SHALL ser um arquivo chapado (`DECOMPOSICAO.md`), não uma skill do Paperclip (AD-008).

**Independent Test** (funcional, manual): auditar a config de cada agente confirmando as credenciais escopadas, a injeção via env var e os toggles de contenção. **Nota**: auditoria de config sozinha não pega o vetor real de acesso (`Bash` + `az`); complementar com teste funcional (aprendizado do STATE.md).

---

## Edge Cases

- WHEN a iniciativa chega sem um dos três blocos (`## Iniciativa` / `## PRD` / `## Documentação técnica`) THEN o PO para, marca a task como `blocked` e reporta à Diana — sem preencher a lacuna (ver POBM-18; especificado em `po/AGENTS.md` e `po/HEARTBEAT.md`).
- WHEN o `decomposicao_json` chega ausente, malformado ou fora do schema THEN o Backlog Manager para e reporta, sem adivinhar (ver POBM-19).
- WHEN um item falha a meio da materialização THEN o Backlog Manager para na hora e reporta os IDs já criados, para permitir limpeza/retomada manual (ver POBM-20).
- WHEN o mesmo wake de aprovação dispara duas vezes THEN o Backlog Manager reconhece que a árvore já foi criada e encerra sem duplicar (ver POBM-21).
- WHEN o Paperclip não reacorda o PO após a aprovação THEN *Run Heartbeat* manual força o wake (comportamento conhecido, não é falha da instrução).
- WHEN a sessão do Claude Code está instável ("Not logged in / resume session unavailable") THEN o handoff pode *parecer* quebrado embora a instrução esteja correta → estabilizar o login é pré-requisito de teste.
- WHEN os agentes descobrem `PAPERCLIP_API_KEY` no ambiente THEN ganham poder de escrita interno no Paperclip → superfície a conter (questão em aberto).

---

## Requirement Traceability

IDs rastreáveis do comportamento especificado. Como não há fase de código, o "Phase" reflete o estágio real:
**Config** = configurado no Paperclip; **Validado** = teste funcional manual feito.

| Requirement ID | Story | Phase | Status |
| -------------- | ----- | ----- | ------ |
| POBM-01 | P1: Decomposição pelo PO | Config | Em operação |
| POBM-02 | P1: Decomposição pelo PO (DoR em 4 partes) | Config | Em operação |
| POBM-03 | P1: Decomposição pelo PO (só Task Dev) | Config | Em operação |
| POBM-04 | P1: Decomposição pelo PO (2 documentos) | Config | Em operação |
| POBM-05 | P1: Decomposição pelo PO (PO read-only) | Config | Em operação |
| POBM-06 | P2: Gate humano (aguarda revisão) | Config | Em operação |
| POBM-07 | P2: Gate humano (ACCEPTED = autorização) | Config | Em operação |
| POBM-08 | P2: Handoff automático (cria task p/ BM) | Config | Em operação (wake instável) |
| POBM-09 | P2: Handoff automático (instrução no HEARTBEAT) | Config | Em operação |
| POBM-10 | P1: Materialização (lê JSON, ordem de parentesco) | Config | Em operação |
| POBM-11 | P1: Materialização (Assigned To vazio) | Config | Em operação |
| POBM-12 | P1: Materialização (mapa de tipos) | Config | Em operação |
| POBM-13 | P1: Materialização (só board-write, só sandbox) | Config | Em operação |
| POBM-14 | P2: Contenção (credenciais separadas) | Config | Em operação |
| POBM-15 | P2: Contenção (injeção via env var) | Config | Em operação |
| POBM-16 | P2: Contenção (cercas de run policy) | Config | Em operação |
| POBM-17 | P2: Contenção (DECOMPOSICAO como arquivo chapado) | Config | Em operação |
| POBM-18 | P1: Decomposição (bloco faltando → blocked/reporta) | Config | Em operação |
| POBM-19 | P1: Materialização (JSON ausente/malformado → para/reporta) | Config | Em operação |
| POBM-20 | P1: Materialização (falha de item → para/reporta IDs) | Config | Em operação |
| POBM-21 | P1: Materialização (idempotência, não duplica) | Config | Em operação |
| POBM-22 | P1: Materialização (lê JSON embutido no corpo do handoff; existência do handoff = autorização) | Config | Em operação (corrigido) |

**ID format:** `POBM-[NUMBER]`

**Status values (adaptado):** Em operação (configurado e funcionando) · Instável (funciona mas com ressalva conhecida) · Pendente.

**Coverage:** 22 requisitos, todos mapeados a comportamento configurado no Paperclip; validação funcional manual descrita por story (não há testes automatizados). POBM-18..21 foram promovidos de "questão em aberto" para requisitos após os arquivos de agente especificarem os comportamentos de erro/idempotência. POBM-22 registra a resolução da dessincronização do handoff pela **Forma B** (AD-010): o PO copia o `decomposicao_json` inteiro no corpo da task de handoff, o BM o lê do próprio corpo, e a existência da task de handoff é a autorização (o BM não navega entre tasks nem verifica confirmation ACCEPTED).

---

## Success Criteria

Como sabemos que a feature está bem-sucedida (o sistema já opera nestes termos):

- [x] Uma iniciativa com os três blocos entra e sai como árvore refinada no Azure Boards, ponta a ponta.
- [x] Nenhum item é escrito no Azure sem confirmation `ACCEPTED` da Diana.
- [x] Todos os itens materializados têm Assigned To vazio e parentesco correto.
- [x] PO nunca escreve; Backlog Manager nunca escreve fora da org `dianasandbox`.
- [ ] Wake pós-aprovação 100% confiável — **ainda não atingido** (ver questões em aberto).
