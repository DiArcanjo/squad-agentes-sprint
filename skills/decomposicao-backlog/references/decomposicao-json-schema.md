# Schema do `decomposicao_json`

Saída estruturada da decomposição. O agente de refinamento emite a MESMA árvore em dois formatos:
o documento `decomposicao` (markdown, para a Diana revisar) e o `decomposicao_json` (este schema, para o
Backlog Manager materializar no Azure). Os dois representam exatamente a mesma árvore: se divergirem, é erro.

> **Contrato compartilhado.** Este schema é o contrato entre o **PO** (produtor) e o **Backlog Manager**
> (consumidor). A cópia canônica de referência do time vive em `contratos/decomposicao-json-schema.md`;
> esta cópia acompanha a skill para que ela seja autossuficiente na importação. **Mantenha as duas em sincronia**
> (ver questão em aberto no `.specs/STATE.md` sobre consolidar a duplicação).

## Schema (não invente campos, não omita campos)

```json
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
```

## Regras do JSON

- É a **mesma árvore** do documento `decomposicao` (markdown), sem diferença de conteúdo.
- O **aninhamento define o parentesco**: `features` dentro do `epic`, `pbis` dentro da feature, `tasks_dev`
  dentro do pbi. O Backlog Manager cria de cima para baixo, passando o ID do pai como parent do filho.
- `description` carrega **Contexto, AS-IS e TO-BE juntos** (mapeia para o campo Description do Azure).
- `acceptance_criteria` carrega os **Critérios de Aceite e os Cenários de Teste juntos** (mapeia para o campo
  Acceptance Criteria do Azure).
- `tasks_dev` é uma **lista de títulos de Task**, cada um prefixado com o número sequencial. Cada título vira
  um work item Task.
- **NÃO** inclua `Assigned To`, `Story Points`, `Area`, `Iteration` ou qualquer campo de atribuição ou estimativa.
- JSON válido e parseável. Sem comentários, sem vírgula sobrando, sem texto fora do JSON.

## Mapeamento de tipos JSON → Azure

| Nó no JSON | Tipo de work item no Azure |
| ---------- | -------------------------- |
| `epic`     | Epic |
| `feature`  | Feature |
| `pbi`      | Product Backlog Item |
| `task_dev` | Task |
