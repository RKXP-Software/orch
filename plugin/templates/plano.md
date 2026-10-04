<!-- orch:plano · id: {AAAAMMDD-HHMM-slug} · status: {planejado|em-execucao|concluido|parcial|cancelado} · criado: {AAAA-MM-DD HH:MM} · commit-inicial: {sha|sem-git} · versao-orch: {versao} -->
# {Título do plano}

**Demanda original:** {texto do usuário, literal}
**Grupo:** {id do grupo, se a demanda gerou vários planos — senão "—"} · **Depende dos planos:** {ids ou "—"}

## Objetivo

{1–3 frases: o que estará entregue quando este plano terminar.}

## Suposições e decisões

- {Suposição assumida por falta de informação, ou decisão tomada com o usuário}

## Tarefas

| ID | Tarefa | Executor | Depende de | Arquivos (escrita) | Status |
|---|---|---|---|---|---|
| T1 | {verbo + objeto} | `orch:pesquisador` | — | — (leitura) | pendente |
| T2 | {…} | `orch:pesquisador` | — | — (leitura) | pendente |
| T3 | {…} | `orch:desenvolvedor` | T1, T2 | `src/...` | pendente |
| T4 | {…} | `orch:testador` | T3 | `tests/...` | pendente |
| T5 | {…} | `orch:revisor` | T3 | — (leitura) | pendente |

Status: `pendente` · `em-andamento` · `concluida` · `falhou` · `pulada`

## Ondas de execução

```
Onda 1: T1 ∥ T2
Onda 2: T3            (após T1, T2)
Onda 3: T4 ∥ T5       (após T3)
```

Caminho crítico: T1 → T3 → T4. Uma tarefa começa assim que **todas** as suas dependências terminam — não espera a onda inteira.

## Detalhe das tarefas

### T1 — {título}
- **Executor:** `{agente}`
- **Faz:** {o que exatamente, com caminhos}
- **Pronto quando:** {critério verificável}
- **Resultado:** {preenchido na execução: resumo, arquivos, verificação}

### T2 — {título}
…

## Registro de execução

| Quando | Evento |
|---|---|
| {AAAA-MM-DD HH:MM} | Plano criado |

## Resultado final

{Preenchido ao concluir: o que foi entregue, arquivos alterados, verificação (testes/build), pendências e riscos.}
