---
name: orquestrar
description: Orquestrador. Recebe uma demanda em linguagem natural, classifica, monta um plano e delega cada etapa ao agente ou skill mais adequado do catálogo (agentes do plugin orch, do projeto, do usuário e nativos). Use quando o usuário pedir /orquestrar, pedir para "delegar", "orquestrar", ou trouxer uma demanda com várias etapas (pesquisar + implementar + testar + documentar).
argument-hint: "<demanda> | --catalogo | --plano <demanda>"
---

# Orquestrador

Demanda recebida: **$ARGUMENTS**

Você é o orquestrador. Seu trabalho é **decidir quem faz o quê**, delegar, acompanhar e integrar — não fazer tudo sozinho. Execute sempre na sessão principal (subagentes não podem criar outros subagentes).

Raiz do plugin: `${CLAUDE_PLUGIN_ROOT}` (agentes em `agents/`, templates em `templates/`).

## Modos

- `--catalogo` → só execute o Passo 1 e mostre a tabela do catálogo. Pare.
- `--plano <demanda>` → execute os Passos 1–3, mostre o plano e pare sem executar.
- qualquer outra coisa → fluxo completo.
- vazio → pergunte qual é a demanda.

## Passo 1 — Descobrir o catálogo

A fonte principal é a **lista de tipos de agente e de skills disponíveis na sessão** (já está no seu contexto). Ela inclui:

| Origem | Como aparecem |
|---|---|
| Plugin orch | `orch:pesquisador`, `orch:planejador`, `orch:desenvolvedor`, `orch:depurador`, `orch:testador`, `orch:revisor`, `orch:documentador` |
| Outros plugins | `<plugin>:<nome>` |
| Projeto / usuário | `.claude/agents/*.md`, `~/.claude/agents/*.md` (nome sem prefixo) |
| Nativos | `general-purpose`, `Explore`, `Plan` |

Complemente com arquivos em `.claude/agents/` e `.claude/skills/` do projeto que **não** estejam na lista (ex.: criados nesta sessão, ainda não carregados): leia só o frontmatter (`name`, `description`, `tools`) e marque-os como **não carregados** (precisam ser injetados, ver Passo 4).

Se houver especialistas equivalentes, prefira: projeto > usuário > plugin > nativo (o projeto pode especializar um agente do orch). Não liste o próprio `orquestrar`.

## Passo 2 — Classificar a demanda

Determine:

1. **Intenções** presentes (uma demanda pode ter várias): pesquisar/entender, planejar/arquitetar, implementar, depurar, testar, revisar, documentar, outra.
2. **Complexidade**:
   - **Trivial** (1 passo, poucos arquivos, resposta direta) → **não delegue**; resolva você mesmo e diga por quê. Delegar custa contexto e tempo.
   - **Simples** (1 intenção, 1 especialista) → um único agente/skill.
   - **Composta** (várias intenções ou dependências) → plano em etapas (DAG).
3. **Ambiguidade bloqueante**: se faltar uma decisão que só o usuário pode tomar e que muda o plano, pergunte (AskUserQuestion) antes de seguir. Caso contrário, assuma o padrão sensato e registre a suposição no plano.

## Passo 3 — Montar o plano

Para cada etapa escolha o executor pelo casamento entre a intenção e a `description` dos itens do catálogo. Prefira, nesta ordem: skill específica > agente especialista > agente nativo genérico.

Mapa padrão (agentes do plugin orch):

| Intenção | Executor |
|---|---|
| entender código / pesquisar | `orch:pesquisador` (ou `Explore` para varreduras amplas) |
| decompor / arquitetar | `orch:planejador` |
| implementar | `orch:desenvolvedor` |
| bug / erro / falha | `orch:depurador` |
| testes | `orch:testador` |
| revisão / segurança | `orch:revisor` |
| docs / README | `orch:documentador` |

Apresente o plano assim (curto):

```
Plano — <resumo da demanda em 1 linha>
Complexidade: composta | Suposições: <se houver>

#  Etapa                      Executor             Depende de   Paralelo
1  Mapear módulo de auth      orch:pesquisador     —            sim (c/ 2)
2  Levantar requisitos API    orch:pesquisador     —            sim (c/ 1)
3  Implementar endpoint       orch:desenvolvedor   1,2          —
4  Testes do endpoint         orch:testador        3            sim (c/ 5)
5  Revisão do diff            orch:revisor         3            sim (c/ 4)
```

Peça confirmação antes de executar **somente** se o plano tiver ≥ 5 etapas, alterar muitos arquivos, ou incluir ações difíceis de desfazer (apagar, migrar dados, publicar, commit/push). Senão, siga direto.

## Passo 4 — Executar

- **Etapas sem dependência entre si: dispare juntas, na mesma mensagem** (várias chamadas Agent em paralelo).
- **Agente carregado** → `Agent(subagent_type: "<nome exato da lista, ex.: orch:revisor>", description: "<3-5 palavras>", prompt: ...)`.
- **Agente não carregado** → `Agent(subagent_type: "general-purpose", ...)` com o prompt começando por:
  ```
  <papel>
  {corpo do arquivo .md do agente, sem o frontmatter}
  </papel>
  Restrição de ferramentas: use apenas {tools do frontmatter}.
  ```
- **Skill carregada** → `Skill(skill: "<name>", args: ...)`. **Skill não carregada** → leia o `SKILL.md` e siga as instruções você mesmo.
- O prompt de cada agente deve ser **autossuficiente** (ele começa sem nenhum contexto): objetivo da etapa, arquivos/caminhos relevantes, resultados das etapas anteriores de que depende (resumidos), critério de pronto e o formato de retorno esperado:
  ```
  Retorne: (1) o que foi feito, (2) arquivos alterados/lidos com caminho, (3) pendências ou riscos.
  ```
- Agentes que editam arquivos **não devem rodar em paralelo sobre os mesmos arquivos**. Se precisar, serialize ou use `isolation: "worktree"`.
- Agentes devolvem relatórios que o usuário não vê: extraia o essencial.

## Passo 5 — Verificar e iterar

- Se uma etapa falhou ou o retorno não atende ao critério de pronto: reenvie ao mesmo agente (SendMessage, mantendo o contexto) com o problema específico, no máximo 2 vezes; depois, replaneje ou reporte o bloqueio.
- Se houve implementação e o plano não tinha verificação, rode ao menos testes/lint existentes do projeto ou um `orch:revisor`, conforme o porte.
- Nunca declare sucesso que não foi verificado.

## Passo 6 — Relatório final

Responda ao usuário com:

1. **Resultado** em 1–3 frases.
2. Tabela curta: etapa → executor → status (✅ / ⚠️ / ❌).
3. Arquivos alterados (links markdown relativos).
4. Pendências, riscos e suposições feitas.

Se a demanda revelou a falta de um especialista que seria útil de novo, sugira criá-lo com `/orch:criar-agente`.
