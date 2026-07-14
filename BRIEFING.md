# Briefing: estruturar o projeto squad-agentes-sprint (spec-driven)

## Contexto para você, Claude Code

Este projeto documenta e versiona um sistema multiagente de refinamento de sprint que já foi construído e está funcionando dentro do Paperclip (orquestrador de agentes rodando em localhost:3100). Até agora o sistema foi construído direto na UI do Paperclip, sem versionamento. Sua tarefa é trazer tudo para este repositório, como fonte de verdade, usando o método da skill `tlc-spec-driven` que já está instalada aqui.

IMPORTANTE, leia antes de agir:
- Leia por completo o `.claude/skills/tlc-spec-driven/SKILL.md` e os references `specify.md` e `memory.md` antes de criar qualquer coisa. Siga o formato que a skill define para `.specs/`.
- Este projeto NÃO é um projeto de código. Não há código-fonte, não há testes automatizados, não há build. O "artefato" são arquivos de instrução de agentes (markdown) e configurações no Paperclip. Portanto: use as fases Specify e o STATE.md de decisões da skill, mas NÃO execute a fase Execute com testes nem o Verifier de mutação, pois não há código para testar. Adapte: "implementar" aqui é registrar a configuração; "validar" é descrever o teste funcional feito manualmente na UI do Paperclip.
- Não invente nada. Se algo não estiver neste briefing, marque como questão em aberto, não preencha por conta própria.

## O que criar

Crie esta estrutura:

.specs/
  STATE.md
  features/
    001-sistema-po-backlog-manager/
      spec.md
    002-conexao-roadmap-paperclip/
      spec.md
agentes/
  po/
    AGENTS.md
    DECOMPOSICAO.md
    SOUL.md
    HEARTBEAT.md
    TOOLS.md
  backlog-manager/
    AGENTS.md
    SOUL.md
    HEARTBEAT.md
    TOOLS.md
contratos/
  decomposicao-json-schema.md

Os arquivos da pasta `agentes/` são a fonte de verdade dos arquivos que rodam no Paperclip. O conteúdo real dos 9 arquivos será colado pela Diana depois. Por ora, crie cada arquivo com um cabeçalho (o nome do arquivo e a qual agente pertence) e um marcador `<!-- conteúdo a colar do Paperclip -->`.

## O sistema (para a spec 001 e o STATE.md)

### Visão geral
Sistema multiagente que transforma uma iniciativa priorizada em backlog refinado no Azure Boards, automaticamente, com um gate de aprovação humano no meio.

Fluxo ponta a ponta (já funciona):
1. Diana entrega uma iniciativa como task no Paperclip, com três blocos: `## Iniciativa`, `## PRD`, `## Documentação técnica`.
2. O agente PO (Product Owner de Refinamento) decompõe a iniciativa em uma árvore Epic → Feature → PBI → Task Dev, deixando cada PBI na Definition of Ready. Salva dois documentos: `decomposicao` (markdown, para a Diana revisar) e `decomposicao_json` (JSON estruturado, para o Backlog Manager consumir).
3. Diana revisa e aprova (confirmation ACCEPTED). Esse é o gate humano.
4. O PO, ao ser aprovado, faz handoff automático: cria uma task atribuída ao Backlog Manager, apontando para o `decomposicao_json`.
5. O Backlog Manager lê o JSON e materializa a árvore inteira no Azure Boards (organização dianasandbox, projeto agentic-sprint), na ordem de parentesco, com Assigned To vazio em todos os itens.

### Os agentes
- PO (Product Owner de Refinamento): decompõe e propõe. Read-only no Azure (credencial pmo-read). Adapter Claude Code, modelo Sonnet 4.6. Cinco arquivos de instrução: AGENTS.md, DECOMPOSICAO.md, SOUL.md, HEARTBEAT.md, TOOLS.md. Foi criado originalmente como um agente tipo CEO (herdou o nome de um PMO anterior) e reescrito para o papel de PO.
- Backlog Manager: materializa no Azure. Único agente com credencial de escrita (board-write). Adapter Claude Code, Sonnet 4.6. Quatro arquivos: AGENTS.md, SOUL.md, HEARTBEAT.md, TOOLS.md. Não tem DECOMPOSICAO porque não decompõe.

### O padrão de decomposição (do time real, Squad Plugins)
Hierarquia: Iniciativa (portfólio) → Epic → Feature → PBI → Task Dev.
Definition of Ready de um PBI, quatro partes: (1) História de Usuário no formato "Como ator, quero objetivo, para benefício"; (2) Spec com Contexto, AS-IS e TO-BE; (3) Critérios de Aceite em Dado/Quando/Então; (4) Cenários de Teste. Método ancorado em INVEST (Bill Wake), corte vertical, SPIDR (Mike Cohn). O PO cria só Task Dev; não cria Task Teste, Plano de Teste, Teste, Issue, DT, Bug, Task RT (esses são de fase posterior, downstream).

## Decisões de arquitetura (para o STATE.md, formato AD-NNN)

- AD-001: Backlog Manager (nome do agente escritor). Descartados "Escritor"/"Publicador" (soavam informais) e "Scrum Master" (papel largo demais, autoridade de processo indesejada no agente que carrega a credencial de escrita). "Backlog Manager" soa natural no ágil e é estreito o suficiente para conter.
- AD-002: Separação de poderes por agente. PO só lê (pmo-read), Backlog Manager só escreve (board-write). Credenciais em secrets separados, para poder revogar a escrita sem derrubar a leitura.
- AD-003: Escrita só na sandbox. board-write é escopado a Work Items Read & Write na org dianasandbox apenas, expiração curta. Nunca produção (technecloud). Contém o estrago mesmo com skip permissions ligado.
- AD-004: Gate humano obrigatório. Nada vai para o Azure sem aprovação da Diana (confirmation ACCEPTED). O aceite da decomposição é a autorização de escrita.
- AD-005: Assigned To sempre vazio. O Backlog Manager cria todos os itens sem atribuir ninguém. Atribuição é decisão humana, feita manualmente após o poker de estimativa.
- AD-006: Handoff automático via instrução no HEARTBEAT do PO. O PO detecta a aprovação (confirmation ACCEPTED) e cria a task para o Backlog Manager. Decidido manter automático.
- AD-007: Credencial injetada via secret do Paperclip como env var (AZURE_DEVOPS_EXT_PAT), não via az login. Descoberta: o az login feito no terminal NÃO alcança o runtime do adapter; só a env var injetada funciona.
- AD-008: Método de decomposição como arquivo chapado (DECOMPOSICAO.md) e não como skill do Paperclip, porque o Paperclip não permite criar pasta na aba de arquivos (skill exige pasta).
- AD-009: Cercas de contenção dos agentes: heartbeat sem intervalo (só wake on demand), Continue after max-turn OFF, create-agents OFF, create/import-skills OFF, max concurrent 1. A contenção vem de permissão e run policy, não de um toggle só.

## Aprendizados (para o STATE.md ou LESSONS)

- O vetor de acesso real de um agente é Bash + CLI (az), não MCP. Auditoria de config sozinha não pega; só o teste funcional pega.
- Instabilidade da sessão do Claude Code (erros "Not logged in / resume session unavailable") faz o handoff parecer quebrado quando na verdade a instrução está certa. Estabilizar o login é pré-requisito para testar comportamento.
- O Paperclip não reacorda o PO de forma 100% confiável após a aprovação; quando trava, Run Heartbeat manual força. Mecanismo nativo de automação do Paperclip (abas Routines e Goals) ainda não foi investigado como solução robusta.
- Identidade errada induz comportamento errado: um agente nomeado CEO puxa "ship over deliberate" e age sozinho. Corrigir a persona (SOUL.md) resolve.

## Questões em aberto (para o STATE.md, handoff)

- Cerca #3 (gate de saída formal) ainda não implementada além do confirmation atual.
- Investigar Routines/Goals do Paperclip para tornar o wake pós-aprovação confiável.
- PAPERCLIP_API_KEY no ambiente dos agentes: dá poder de escrita interno no Paperclip, os agentes a descobrem sozinhos. A estudar e conter.
- Refinador-agente (validador da Definition of Ready) decidido como próximo agente, ainda não construído. Deve ser ancorado em determinismo máximo (regras explícitas, saída estruturada).
- Conexão roadmap → Paperclip (feature 002): a Diana já fez a feature no roadmap para gerar uma key. Direção pretendida: roadmap empurra a iniciativa para o Paperclip via API. Pendência conhecida: o roadmap está hospedado na nuvem (Vercel) e o Paperclip roda em localhost, então há um problema de rede a resolver (túnel ou exposição pública).

## Schema do contrato decomposicao_json (para contratos/decomposicao-json-schema.md)

Este é o contrato entre o PO (produtor) e o Backlog Manager (consumidor). O PO salva a árvore neste formato como documento `decomposicao_json`; o Backlog Manager lê e materializa no Azure.

Schema:

{
  "iniciativa": "<id ou título da iniciativa>",
  "epic": {
    "titulo": "<título do Epic>",
    "descricao": "<objetivo estratégico do Epic>",
    "features": [
      {
        "titulo": "<título da Feature>",
        "descricao": "<contexto de negócio da Feature>",
        "pbis": [
          {
            "titulo": "<título do PBI>",
            "historia_usuario": "Como <ator>, quero <objetivo>, para <benefício>.",
            "description": "Contexto da demanda: <...>\n\nAS-IS: <...>\n\nTO-BE: <...>",
            "acceptance_criteria": "Critérios de Aceite:\n1. Dado <...>, quando <...>, então <...>\n\nCenários de Teste:\n- Cenário 1 — <...>",
            "tasks_dev": [
              "1 - <título da Task Dev>",
              "2 - <título da Task Dev>"
            ]
          }
        ]
      }
    ]
  }
}

Regras do contrato:
- É a mesma árvore do documento `decomposicao` (markdown), sem diferença de conteúdo.
- O aninhamento define o parentesco: features dentro do epic, pbis dentro da feature, tasks_dev dentro do pbi. O Backlog Manager cria de cima para baixo, passando o ID do pai como parent do filho.
- `description` carrega Contexto, AS-IS e TO-BE juntos (mapeia para o campo Description do Azure).
- `acceptance_criteria` carrega os Critérios de Aceite e os Cenários de Teste juntos (mapeia para o campo Acceptance Criteria do Azure).
- `tasks_dev` é uma lista de títulos de Task, cada um prefixado com número sequencial. Vira um work item Task por título.
- O JSON NÃO inclui Assigned To, Story Points, Area, Iteration ou qualquer campo de atribuição ou estimativa.

Mapeamento de tipos JSON para Azure:
- epic vira tipo "Epic"
- feature vira tipo "Feature"
- pbi vira tipo "Product Backlog Item"
- task_dev vira tipo "Task"

## Tarefa

1. Leia o SKILL.md e os references specify.md e memory.md por completo.
2. Crie a estrutura de pastas acima.
3. Escreva o `.specs/STATE.md` com as decisões (AD-001 a AD-009), os aprendizados e as questões em aberto, no formato que a skill define para o STATE.md.
4. Escreva a `spec.md` da feature 001 (sistema-po-backlog-manager), documentando o sistema que já funciona, com requisitos rastreáveis por ID, adaptando o formato ao fato de que é configuração de agente, não código.
5. Crie o esqueleto da `spec.md` da feature 002 (conexao-roadmap-paperclip) com o problema, o objetivo e as questões em aberto conhecidas, marcada como não iniciada.
6. Escreva o `contratos/decomposicao-json-schema.md` com o schema e as regras acima.
7. Crie os 9 arquivos em `agentes/po/` e `agentes/backlog-manager/` com cabeçalho e o marcador `<!-- conteúdo a colar do Paperclip -->`.
8. NÃO faça commit. Ao terminar, liste tudo que criou e me diga o que precisa de mim (o conteúdo real dos 9 arquivos de agente).