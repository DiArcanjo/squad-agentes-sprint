# STATE

> Memória do projeto **squad-agentes-sprint**: sistema multiagente de refinamento de sprint
> construído e em operação dentro do Paperclip (orquestrador em `localhost:3100`). Este repositório
> é a fonte de verdade da configuração dos agentes, trazida da UI do Paperclip para versionamento.
>
> Adaptação da skill `tlc-spec-driven`: este NÃO é um projeto de código. Não há build, testes
> automatizados nem código-fonte. Usamos as fases **Specify** e o **STATE.md** de decisões; NÃO
> executamos a fase Execute com testes nem o Verifier de mutação. "Implementar" aqui é registrar a
> configuração dos agentes; "validar" é descrever o teste funcional feito manualmente na UI do Paperclip.

## Decisions

### AD-001
- **Decision**: O agente escritor no Azure Boards se chama **Backlog Manager**.
- **Reason**: Soa natural no vocabulário ágil e é estreito o suficiente para conter o escopo (só materializar backlog).
- **Trade-off**: Descartados "Escritor"/"Publicador" (soavam informais) e "Scrum Master" (papel largo demais, carregaria autoridade de processo indesejada justo no agente que detém a credencial de escrita).
- **Scope**: Feature 001 — nomenclatura e identidade do agente escritor.
- **Date**: 2026-07-13
- **Status**: active

### AD-002
- **Decision**: Separação de poderes por agente — o PO só lê (credencial `pmo-read`), o Backlog Manager só escreve (credencial `board-write`).
- **Reason**: Isola privilégios; permite revogar a escrita sem derrubar a leitura, pois as credenciais ficam em secrets separados.
- **Trade-off**: Mais credenciais para gerenciar; handoff explícito entre os dois agentes em vez de um agente único onipotente.
- **Scope**: Feature 001 — modelo de permissões dos dois agentes.
- **Date**: 2026-07-13
- **Status**: active

### AD-003
- **Decision**: A escrita acontece **apenas na sandbox**. `board-write` é escopada a *Work Items Read & Write* na organização `dianasandbox` apenas, com expiração curta.
- **Reason**: Contém o estrago mesmo com skip permissions ligado; nunca toca produção (`technecloud`).
- **Trade-off**: Necessário renovar a credencial com frequência (expiração curta); nenhum caminho automático para produção.
- **Scope**: Feature 001 — escopo da credencial `board-write`.
- **Date**: 2026-07-13
- **Status**: active

### AD-004
- **Decision**: **Gate humano obrigatório**. Nada vai para o Azure sem aprovação da Diana (confirmation `ACCEPTED`). O aceite da decomposição é a autorização de escrita.
- **Reason**: Um humano revisa a árvore Epic→Feature→PBI→Task antes de qualquer materialização; o aceite funciona como assinatura de autorização.
- **Trade-off**: O fluxo não é 100% autônomo — depende de uma ação humana no meio.
- **Scope**: Feature 001 — ponto de controle entre decomposição (PO) e materialização (Backlog Manager).
- **Date**: 2026-07-13
- **Status**: active

### AD-005
- **Decision**: **Assigned To sempre vazio**. O Backlog Manager cria todos os itens no Azure sem atribuir ninguém.
- **Reason**: Atribuição é decisão humana, feita manualmente após o poker de estimativa.
- **Trade-off**: Passo manual adicional depois da materialização; o agente não fecha o ciclo de alocação.
- **Scope**: Feature 001 — comportamento de escrita do Backlog Manager e contrato `decomposicao_json`.
- **Date**: 2026-07-13
- **Status**: active

### AD-006
- **Decision**: **Handoff automático** via instrução no `HEARTBEAT.md` do PO. Ao detectar a aprovação (confirmation `ACCEPTED`), o PO cria automaticamente a task para o Backlog Manager com a árvore aprovada (`decomposicao_json`) copiada inteira no corpo da task (formato do carregamento detalhado em AD-010).
- **Reason**: Evita um passo manual de repasse entre aprovação e materialização; mantém o fluxo fluindo depois do gate humano.
- **Trade-off**: Depende do Paperclip reacordar o PO de forma confiável após a aprovação — o que hoje não é 100% garantido (ver Aprendizados e Questões em aberto).
- **Scope**: Feature 001 — transição PO → Backlog Manager.
- **Date**: 2026-07-13
- **Status**: active

### AD-007
- **Decision**: A credencial é injetada via **secret do Paperclip como env var** (`AZURE_DEVOPS_EXT_PAT`), não via `az login`.
- **Reason**: Descoberta empírica — o `az login` feito no terminal NÃO alcança o runtime do adapter; só a env var injetada funciona.
- **Trade-off**: A credencial vive como variável de ambiente no runtime do adapter (superfície a conter); depende do mecanismo de secrets do Paperclip.
- **Scope**: Feature 001 — mecanismo de injeção de credencial para ambos os agentes.
- **Date**: 2026-07-13
- **Status**: active

### AD-008
- **Decision**: O método de decomposição vive como **arquivo chapado** (`DECOMPOSICAO.md`), não como skill do Paperclip.
- **Reason**: O Paperclip não permite criar pasta na aba de arquivos, e uma skill exige pasta.
- **Trade-off**: Não aproveita o mecanismo de skills do Paperclip (descoberta/versionamento nativo); é apenas um arquivo de instrução lido pelo PO.
- **Scope**: Feature 001 — forma de empacotar o método de decomposição do PO.
- **Date**: 2026-07-13
- **Status**: active

### AD-009
- **Decision**: **Cercas de contenção** dos agentes: heartbeat sem intervalo (só *wake on demand*), *Continue after max-turn* OFF, *create-agents* OFF, *create/import-skills* OFF, *max concurrent* = 1.
- **Reason**: A contenção vem de permissão e run policy combinadas, não de um único toggle. Impede que os agentes se auto-expandam ou rodem em loop.
- **Trade-off**: Menos autonomia operacional; wake precisa ser disparado (por evento ou manualmente via Run Heartbeat).
- **Scope**: Feature 001 — run policy e permissões de ambos os agentes no Paperclip.
- **Date**: 2026-07-13
- **Status**: active

### AD-010
- **Decision**: No handoff PO → Backlog Manager, a árvore aprovada viaja **EMBUTIDA** (Forma B): o PO copia o `decomposicao_json` inteiro no corpo da task de handoff, e a **existência dessa task é a autorização** do BM — o BM lê o JSON do próprio corpo, não navega para outra task nem verifica confirmation `ACCEPTED`.
- **Reason**: Torna o BM autossuficiente na task em que acorda. Concentra a confiança da autorização em um único fato verificável (o PO só cria a task de handoff após a Diana aprovar), eliminando a navegação entre tasks e a checagem de aprovação remota — que eram fonte de dessincronização e faziam o BM parar por "sem aprovação".
- **Trade-off**: O `decomposicao_json` fica duplicado (na task de decomposição original e no corpo do handoff); se a Diana editar a decomposição após o handoff, a cópia embutida não reflete a mudança. Descartada a **Forma A** (BM segue ponteiro/ID para a task de decomposição original), que evita a duplicação mas reintroduz a navegação entre tasks e a leitura da aprovação em outro lugar.
- **Scope**: Feature 001 — mecanismo de transporte e autorização do handoff. Refina AD-006.
- **Date**: 2026-07-13
- **Status**: active

---

## Aprendizados

Lições operacionais colhidas durante a construção manual do sistema no Paperclip. Registradas aqui porque
não há camada LESSONS automatizada neste projeto (sem código para verificar). Referenciam decisões acima.

- **O vetor de acesso real de um agente é `Bash` + CLI (`az`), não MCP.** Auditoria de configuração sozinha não pega esse caminho; só o teste funcional pega. → Reforça AD-002/AD-003 (contenção por credencial escopada) e AD-009 (contenção por run policy).
- **Instabilidade da sessão do Claude Code** (erros "Not logged in / resume session unavailable") faz o handoff *parecer* quebrado. Estabilizar o login é pré-requisito para testar comportamento. → Contexto para AD-006. **Ressalva (13/07/2026)**: a passada de consistência revelou que nem toda falha de handoff era da sessão — havia uma dessincronização real de instrução entre os dois lados do handoff (o `po/HEARTBEAT.md §5` e o `backlog-manager/HEARTBEAT.md` discordavam sobre onde ficavam a aprovação e o `decomposicao_json`). Resolvido pela **Forma B** (ver AD-010 e POBM-22): o PO copia o `decomposicao_json` inteiro no corpo da task de handoff; o BM lê do próprio corpo; e a existência da task de handoff é a autorização (o BM não navega entre tasks nem checa confirmation ACCEPTED). Lição: não atribuir toda instabilidade de comportamento à sessão sem antes cruzar as instruções dos dois lados do handoff.
- **O Paperclip não reacorda o PO de forma 100% confiável após a aprovação.** Quando trava, *Run Heartbeat* manual força o wake. O mecanismo nativo de automação do Paperclip (abas *Routines* e *Goals*) ainda não foi investigado como solução robusta. → Contexto para AD-006; ver Questões em aberto.
- **Identidade errada induz comportamento errado.** Um agente nomeado CEO puxa "ship over deliberate" e age sozinho. Corrigir a persona (`SOUL.md`) resolve. O PO herdou o nome de um PMO/CEO anterior e foi reescrito para o papel de PO. → Contexto para AD-001 e para o `SOUL.md` do PO.

---

## Questões em aberto

Pendências conhecidas no momento desta documentação. Não inventar respostas — cada item permanece aberto até ser resolvido.

- **Cerca #3 (gate de saída formal)** ainda não implementada além do confirmation atual (AD-004).
- **Investigar Routines/Goals do Paperclip** para tornar o wake pós-aprovação confiável (relacionado a AD-006 e ao aprendizado sobre o PO não reacordar).
- **`PAPERCLIP_API_KEY` no ambiente dos agentes**: dá poder de escrita interno no Paperclip, e os agentes a descobrem sozinhos. A estudar e conter.
- **Refinador-agente** (validador da Definition of Ready) decidido como próximo agente, ainda não construído. Deve ser ancorado em determinismo máximo (regras explícitas, saída estruturada). **Discrepância detectada (13/07/2026)**: o `agentes/po/TOOLS.md` já descreve o Refinador como validador **operacional** ("O Refinador audita a prontidão dos PBIs que você produz… um produz, o outro valida"), o que conflita com "ainda não construído". A confirmar com a Diana: o Refinador já existe, ou o texto no TOOLS.md é instrução prospectiva sobre um colaborador futuro? Enquanto não resolvido, tratar como não construído.
- **Conexão roadmap → Paperclip (feature 002)**: ver `features/002-conexao-roadmap-paperclip/spec.md`. Pendência de rede conhecida — roadmap na nuvem (Vercel), Paperclip em localhost.
- **Reference name do campo "História de Usuário" do PBI no Azure (não confirmado)**: o reference name interno do campo customizado não foi confirmado na sandbox `dianasandbox`. No `agentes/backlog-manager/TOOLS.md` o mapeamento de `historia_usuario` (PBI) está genérico ("campo de História de Usuário do PBI"). No teste E2E de 09/07/2026 a escrita funcionou, então ou o campo existe com um nome que o `az` aceitou, ou a história caiu na Description. **Pendência**: rodar `az boards work-item show` em um PBI criado, identificar onde a história foi parar e qual o reference name real do campo, e então fixar esse nome exato no `TOOLS.md` do Backlog Manager. **Não é bloqueante** (a escrita funciona) — é precisão de mapeamento. Relacionado a AD-005 e ao requisito POBM-12 da feature 001.

---

## Handoff

- **Feature**: Estruturação inicial do repositório (`.specs/`, `agentes/`, `contratos/`) a partir do `BRIEFING.md`.
- **Phase / Task**: Specify — documentar sistema existente. Feature 001 especificada; feature 002 é esqueleto (não iniciada).
- **Completed**: `STATE.md` (decisões AD-001..AD-009, aprendizados, questões em aberto); `features/001-sistema-po-backlog-manager/spec.md`; esqueleto `features/002-conexao-roadmap-paperclip/spec.md`; `contratos/decomposicao-json-schema.md`; 9 arquivos-esqueleto em `agentes/po/` e `agentes/backlog-manager/`.
- **In-progress** (file:line): none.
- **Next step**: Diana cola o conteúdo real dos 9 arquivos de agente (fonte de verdade vinda do Paperclip) sobre os marcadores `<!-- conteúdo a colar do Paperclip -->`.
- **Blockers**: Conteúdo real dos 9 arquivos de agente depende da Diana. Feature 002 depende de resolver a exposição de rede (localhost ↔ Vercel).
- **Uncommitted files**: toda a árvore `.specs/`, `agentes/`, `contratos/` (não commitado — instrução do briefing foi NÃO fazer commit).
- **Branch**: main
