# Decomposição de backlog (método do PO)

Referência de método. O PO consulta este arquivo sempre que for decompor uma iniciativa. A hierarquia e os nomes de item são os do time; o método é o padrão ágil consolidado (INVEST, SPIDR, Definition of Ready).

## Hierarquia-alvo

Iniciativa (entrada, portfólio) -> Epic -> Feature -> PBI -> Task Dev (saída).

Regra de tamanho: o nível desce conforme o item encolhe. Epic é grande demais para um sprint. Feature é uma capacidade entregável. PBI cabe em um sprint e é a unidade central. Task Dev é passo de execução.

Não crie acima do Epic nem abaixo ou fora de Task Dev. Não invente tipo novo.

## Passo a passo

1. Leia os três blocos da task: Iniciativa, PRD, Documentação técnica. Se faltar algo essencial, PARE e reporte.
2. Enquadre o Epic. Um Epic por iniciativa. Objetivo estratégico: o benefício macro da demanda.
3. Identifique as Features. Cada capacidade ou grupo de requisito do PRD que entrega valor vira uma Feature. Regra prática: um requisito do PRD tende a virar uma Feature.
4. Quebre cada Feature em PBIs por CORTE VERTICAL. Cada PBI leva um pouco de cada camada e entrega funcionalidade ponta a ponta. NUNCA quebre por camada técnica (um PBI de UI, outro de banco). Se um PBI ficar grande demais, use SPIDR e os gatilhos de fatiamento (abaixo).
5. Escreva o conteúdo de cada PBI até a Definition of Ready (abaixo).
6. Derive as Task Dev de cada PBI: os passos de implementação. Só Task Dev.
7. Self-check de cada PBI contra INVEST e a DoR. Confirme: todo critério de aceite do PRD está coberto por ao menos um PBI? Itens de "fora de escopo" do PRD NÃO viraram trabalho?

## Definition of Ready (o PBI só está pronto com os quatro)

1. História de Usuário: "Como <ator>, quero <objetivo>, para <benefício>."
2. Spec: Contexto da demanda, mais AS-IS (situação atual) e TO-BE (situação desejada), no nível que a demanda pedir. Técnico quando a demanda é técnica, de processo ou negócio quando não. Use a Documentação técnica para o AS-IS/TO-BE técnico.
3. Critérios de Aceite: em Dado/Quando/Então, testáveis.
4. Cenários de Teste: registrados no PBI (shift-left, o teste nasce no refinamento). Você NÃO cria a Task Teste; só registra os cenários dentro do PBI.

A DoR é piso de qualidade, não cartório. Detalhe a fundo o que vai para a sprint, não o backlog inteiro.

## INVEST (qualidade de cada PBI)

- Independent: não se sobrepõe nem impõe ordem.
- Negotiable: captura a essência, não fecha o detalhe.
- Valuable: entrega valor sozinho.
- Estimable: detalhe suficiente para estimar.
- Small: cabe em um sprint.
- Testable: dá para escrever teste. Por isso os cenários de teste.

Se um PBI falha em algum, reveja o corte.

## SPIDR (como quebrar um PBI grande)

Quase todo PBI grande divide por uma destas cinco dimensões:
- Spike: separar pesquisa ou incerteza da implementação.
- Path: happy path primeiro, exceções depois.
- Interface: por canal ou interface (ex: um fluxo por integração).
- Data: por variação de dado.
- Rules: por regra de negócio.

Aplique mantendo cada fatia vertical e valiosa.

## Quando um PBI está grande demais (gatilhos de fatiamento)

O corte vertical não é desculpa para PBIs grandes. Depois de montar um PBI, verifique se ele acumula fatias verticais que poderiam ser entregues e testadas separadamente. Os sinais de que um PBI deve ser quebrado em dois ou mais:

- Mistura caminho feliz com resiliência. Se o PBI cobre "fazer a coisa funcionar" E "tratar falha, timeout, idempotência, reprocessamento" no mesmo item, separe. O caminho feliz é uma fatia entregável; a resiliência é outra.
- Cobre mais de um verbo de negócio. Se o PBI faz detectar E emitir, ou emitir E exibir, provavelmente são PBIs diferentes.
- Acumula muitos critérios de aceite ou muitas Task Dev. Não é regra fixa, mas se um PBI passa de cerca de 4-5 critérios de aceite ou 5 Task Dev, releia: quase sempre há duas fatias verticais ali dentro. Use isso como sinal para revisar, não como limite automático.

Ao fatiar, cada PBI resultante ainda deve passar no INVEST inteiro: continuar vertical, valioso e testável sozinho. NUNCA fatie por camada técnica só para diminuir o tamanho. Se ao quebrar você criar um PBI que não entrega valor sozinho, o corte está errado: volte e quebre por outra dimensão (SPIDR).

Se após revisar você concluir que o PBI grande é de fato indivisível sem virar corte por camada, mantenha-o e registre na justificativa por que não foi fatiado. Um PBI grande bem justificado é aceitável; um PBI grande por falta de análise não.

## Anti-padrões (não faça)

- Quebrar por camada técnica. Sempre corte vertical.
- Detalhar o backlog inteiro. Só o que vai para a sprint.
- Inventar tipo de item. Se não encaixa, PARE e sinalize.
- Preencher lacuna com suposição. Falta de info significa PARAR e reportar.
- Gerar trabalho a partir do "fora de escopo" do PRD.

## Saída estruturada (JSON para o Backlog Manager)

Além do documento `decomposicao` em markdown (para a Diana revisar), emita a MESMA árvore em JSON, salvo como documento `decomposicao_json`. O markdown é para humano ler; o JSON é para o Backlog Manager materializar no Azure. Os dois representam exatamente a mesma árvore: se divergirem, é erro.

Schema exato (não invente campos, não omita campos):

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

Regras do JSON:
- É a mesma árvore do markdown, sem diferença de conteúdo.
- `description` carrega Contexto, AS-IS e TO-BE, no mesmo texto (campo Description do Azure).
- `acceptance_criteria` carrega os Critérios de Aceite e os Cenários de Teste juntos (campo Acceptance Criteria do Azure).
- `tasks_dev` é uma lista de títulos de Task, cada um prefixado com o número sequencial.
- Não inclua Assigned To, Story Points, Area, Iteration ou qualquer campo de atribuição ou estimativa.
- JSON válido e parseável. Sem comentários, sem vírgula sobrando, sem texto fora do JSON.