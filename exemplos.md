# Prompts de teste do orquestrador

Use para validar o roteamento depois de mudar agentes/skills. Comece com `--plano` para ver a decisão sem executar.

| Prompt | Roteamento esperado |
|---|---|
| `/orch:orquestrar --catalogo` | Tabela com os 7 agentes `orch:*` + nativos + `criar-agente` |
| `/orch:orquestrar quanto é 2^10?` | Trivial → responde direto, sem delegar |
| `/orch:orquestrar --plano explique como o orquestrar decide a complexidade` | Simples → plano de 1 tarefa (`orch:pesquisador`) |
| `/orch:orquestrar --plano o marketplace.json não é reconhecido ao adicionar o marketplace` | Simples → `orch:depurador` |
| `/orch:orquestrar --plano crie um script que lista os agentes do plugin em tabela, com testes e README` | Composta → `orch:pesquisador` → `orch:desenvolvedor` → (`orch:testador` ∥ `orch:revisor`) → `orch:documentador` |
| `/orch:criar-agente agente tradutor que traduz documentação pt-BR/en` | Cria `.claude/agents/tradutor.md` no projeto |

## Especialização (rodar em projetos reais)

| Prompt | Resultado esperado |
|---|---|
| `/orch:especializar` em projeto com código | Perfil com stack/comandos com evidência, import no `CLAUDE.md`, proposta de 0–5 especialistas com multiseleção |
| `/orch:especializar` em pasta vazia | Avisa que não há código e oferece `--novo` |
| `/orch:especializar --novo "React + Vite + TS"` | Perfil `modo: novo` com comandos "recomendados", sem especialistas |
| `/orch:especializar --atualizar` após commits | Atualiza cabeçalho (commit/data); só reescreve agentes com o comentário `orch:especializar` |
| `/orch:orquestrar --plano <tarefa no domínio de um especialista>` | Etapa roteada para o especialista do projeto, não para o genérico |
| `/orch:orquestrar <tarefa>` em projeto com código e sem perfil | Sugere `/orch:especializar` uma vez e segue normalmente |

## Planos e paralelismo

| Prompt | Resultado esperado |
|---|---|
| `/orch:orquestrar --plano corrija o bug X e crie a tela Y` | **2 planos** (objetivos independentes) no mesmo grupo, salvos em `.claude/orch/planos/`, índice atualizado, nada executado |
| `/orch:orquestrar --plano <feature com pesquisa + implementação + testes + revisão>` | 1 plano; pesquisas na onda 1 em paralelo; testes e revisão em paralelo após a implementação |
| Duas tarefas de implementação que alteram o mesmo arquivo | Dependência adicionada entre elas (não rodam juntas) |
| `/orch:orquestrar --executar <id>` | Status `em-execucao`; tarefas prontas disparadas juntas; liberadas sem esperar a onda; arquivo atualizado a cada tarefa |
| Interromper no meio e rodar `--executar <id>` de novo | Retoma: pula `concluida`, revisa `em-andamento` |
| `/orch:orquestrar --planos` | Mostra o `INDICE.md` |
| `/orch:orquestrar quanto é 2^10?` | Trivial: responde direto, **não** cria plano |
