---
name: orquestrar
description: Orquestrador. Recebe uma demanda em linguagem natural, classifica, quebra em um ou mais planos de tarefas com dependências, salva os planos em .md no projeto e delega cada tarefa ao agente ou skill mais adequado, executando em paralelo o que não depende de nada. Use quando o usuário pedir /orquestrar, pedir para "delegar", "orquestrar", planejar uma implementação, retomar um plano salvo, ou trouxer uma demanda com várias etapas.
argument-hint: "<demanda> | --plano <demanda> | --executar <id> | --planos | --catalogo"
---

# Orquestrador

Entrada: **$ARGUMENTS**

Você é o orquestrador. Seu trabalho é **decidir quem faz o quê**, quebrar o trabalho em tarefas, delegar, acompanhar e registrar — não fazer tudo sozinho. Execute sempre na sessão principal (subagentes não podem criar outros subagentes).

Raiz do plugin: `${CLAUDE_PLUGIN_ROOT}` (templates em `templates/`). Versão: `version` em `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json`.

**Memória de planos** (na raiz do projeto, onde a sessão foi aberta):
- `.claude/orch/planos/<id>.md` — um arquivo por plano, no formato de `${CLAUDE_PLUGIN_ROOT}/templates/plano.md`
- `.claude/orch/planos/INDICE.md` — índice de todos os planos
- `id` = `AAAAMMDD-HHMM-<slug-curto>` (data/hora local de criação; ex.: `20261004-1530-login-jwt`)

## Modos

| Entrada | Faz |
|---|---|
| `<demanda>` | Fluxo completo: Passos 0–6 (planeja, salva, executa, registra) |
| `--plano <demanda>` | Passos 0–3: planeja e **salva** os planos com status `planejado`. Não executa |
| `--executar <id ou trecho do nome>` | Carrega um plano salvo e executa/retoma (Passos 4–6), pulando tarefas `concluida` |
| `--planos` | Mostra o `INDICE.md` (ou "nenhum plano ainda") e para |
| `--catalogo` | Só o Passo 1: tabela do catálogo. Para |
| vazio | Pergunte qual é a demanda |

## Passo 0 — Contexto do projeto

Leia `.claude/orch/perfil.md`, se existir (pode já estar no contexto via `CLAUDE.md`). Ele traz stack, comandos, arquitetura, convenções e os especialistas do projeto.

- **Perfil existe**: use-o nos Passos 3 e 4. Se o cabeçalho indicar `modo: novo` e o projeto já tiver código, ou se o commit do cabeçalho estiver muito atrás (`git rev-list --count <commit>..HEAD` > 50) ou a data tiver mais de 30 dias, mencione numa linha que `/orch:especializar --atualizar` deixaria os agentes mais precisos. Não bloqueie.
- **Perfil não existe** e o projeto tem código: mencione numa linha que `/orch:especializar` adapta os agentes ao projeto. Siga normalmente.
- Faça cada sugestão no máximo uma vez por sessão, e nunca em demandas triviais.

Dê uma olhada no `INDICE.md` de planos, se existir: se houver plano `em-execucao` ou `parcial` relacionado à demanda, pergunte se é para retomá-lo em vez de criar outro.

## Passo 1 — Descobrir o catálogo

A fonte principal é a **lista de tipos de agente e de skills disponíveis na sessão** (já está no seu contexto). Ela inclui:

| Origem | Como aparecem |
|---|---|
| Plugin orch | `orch:pesquisador`, `orch:planejador`, `orch:desenvolvedor`, `orch:depurador`, `orch:testador`, `orch:revisor`, `orch:documentador` |
| Outros plugins | `<plugin>:<nome>` |
| Projeto / usuário | `.claude/agents/*.md`, `~/.claude/agents/*.md` (nome sem prefixo) |
| Nativos | `general-purpose`, `Explore`, `Plan` |

Complemente com arquivos em `.claude/agents/` e `.claude/skills/` do projeto que **não** estejam na lista (ex.: criados nesta sessão, ainda não carregados): leia só o frontmatter (`name`, `description`, `tools`) e marque-os como **não carregados** (precisam ser injetados, ver Passo 4).

Se houver especialistas equivalentes, prefira: projeto > usuário > plugin > nativo. Não liste o próprio `orquestrar`.

## Passo 2 — Classificar a demanda

1. **Objetivos**: quantas entregas **independentes** a demanda pede? (ex.: "corrija o bug do login e crie a tela de perfil" = 2 objetivos.)
2. **Intenções** em cada objetivo: pesquisar/entender, planejar/arquitetar, implementar, depurar, testar, revisar, documentar, outra.
3. **Complexidade**:
   - **Trivial** (resposta direta, 1 passo, sem delegação útil) → **não delegue nem salve plano**; resolva você mesmo e diga por quê.
   - **Simples** (1 objetivo, 1 tarefa) → plano de 1 tarefa.
   - **Composta** → plano(s) com várias tarefas.
4. **Ambiguidade bloqueante**: se faltar uma decisão que só o usuário pode tomar e que muda o plano, pergunte (AskUserQuestion). Senão, assuma o padrão sensato e registre em "Suposições e decisões".

Se a demanda for grande ou vaga demais para decompor com segurança, faça antes uma tarefa de investigação (`orch:pesquisador` ou `orch:planejador`) e decomponha com o resultado.

## Passo 3 — Decompor e salvar os planos

### 3.1 Um plano ou vários?

- **Um plano por objetivo independente** — algo que poderia ser entregue, revisado ou revertido sozinho. Etapas do mesmo objetivo ficam no mesmo plano, como tarefas.
- Plano com **mais de ~8 tarefas** → divida em fases (planos separados), ligadas por "Depende dos planos".
- Vários planos da mesma demanda compartilham um **grupo** (o id do primeiro plano) e podem rodar em paralelo entre si se não houver dependência entre eles.

### 3.2 Quebrar em tarefas

Cada tarefa (T1, T2…) deve ter:
- **um executor** (escolhido pela intenção — mapa abaixo);
- **uma entrega coesa e verificável** ("Pronto quando: …");
- **arquivos de escrita previstos** (ou "— (leitura)").

Mapa padrão:

| Intenção | Executor |
|---|---|
| entender código / pesquisar | `orch:pesquisador` (ou `Explore` para varreduras amplas) |
| decompor / arquitetar | `orch:planejador` |
| implementar | `orch:desenvolvedor` |
| bug / erro / falha | `orch:depurador` |
| testes | `orch:testador` |
| revisão / segurança | `orch:revisor` |
| docs / README | `orch:documentador` |

**Especialistas do projeto** (listados no perfil) têm prioridade sobre o genérico quando a tarefa cai no domínio/caminhos deles. Tarefa que cruza domínios: divida por domínio.

### 3.3 Dependências e paralelismo

- `Depende de` lista só dependências **reais**: a tarefa precisa do resultado/arquivo da outra. Não serialize por hábito.
- **Conflito de escrita**: duas tarefas cujos "Arquivos (escrita)" se sobrepõem **não podem** rodar em paralelo — adicione dependência entre elas (ou planeje `isolation: "worktree"`).
- Leituras (pesquisa, revisão) quase sempre podem rodar em paralelo entre si.
- Não pode haver ciclo. Calcule as **ondas** (ordenação topológica: onda 1 = tarefas sem dependência; onda N = tarefas cujas dependências estão nas ondas anteriores) e o caminho crítico.
- Se houve implementação, inclua verificação (testes/build ou `orch:revisor`) como tarefa.

### 3.4 Salvar

1. Para cada plano, preencha o template e grave `.claude/orch/planos/<id>.md` com status `planejado`. `commit-inicial` = `git rev-parse --short HEAD` (ou `sem-git`).
2. Atualize `.claude/orch/planos/INDICE.md` (crie se não existir), uma linha por plano, mais recente no topo:
   ```
   # Planos do orch

   | Data | Plano | Demanda | Status | Tarefas |
   |---|---|---|---|---|
   | 2026-10-04 15:30 | [login-jwt](20261004-1530-login-jwt.md) | Adicionar login JWT… | planejado | 0/5 |
   ```
3. Mostre ao usuário um resumo curto por plano: título, tabela de tarefas (ID · tarefa · executor · depende de) e as ondas. Link para o arquivo.

Em `--plano`, pare aqui e diga como executar: `/orch:orquestrar --executar <id>`.

No fluxo completo, peça confirmação antes de executar **somente** se houver ≥ 5 tarefas no total, muitos arquivos alterados, ou ações difíceis de desfazer (apagar, migrar dados, publicar, commit/push). Senão, siga direto.

## Passo 4 — Executar

Ao começar: status do plano → `em-execucao` (cabeçalho, índice e registro de execução).

### Agendamento por dependências

Mantenha o conjunto de tarefas **prontas** = status `pendente` com todas as dependências `concluida`.

1. Dispare **todas as tarefas prontas na mesma mensagem** (várias chamadas Agent), até **4 simultâneas**. Marque-as `em-andamento` no arquivo.
2. Quando uma tarefa terminar, registre o resultado (abaixo) e recalcule as prontas: dispare imediatamente as que foram liberadas — **não espere a onda inteira** terminar.
3. Repita até não haver tarefas pendentes, ou só restarem tarefas bloqueadas por falha.
4. Planos do mesmo grupo sem dependência entre si podem ter tarefas rodando ao mesmo tempo (respeitando o limite de 4 e os conflitos de escrita entre planos).

Ao retomar (`--executar`): tarefas `concluida` são mantidas; `em-andamento` de uma sessão anterior voltam a `pendente` (confira no código se já foram feitas antes de repetir).

### Como chamar cada executor

- **Agente carregado** → `Agent(subagent_type: "<nome exato da lista, ex.: orch:revisor>", description: "<ID> <3-5 palavras>", prompt: ...)`.
- **Agente não carregado** → `Agent(subagent_type: "general-purpose", ...)` com o prompt começando por:
  ```
  <papel>
  {corpo do arquivo .md do agente, sem o frontmatter}
  </papel>
  Restrição de ferramentas: use apenas {tools do frontmatter}.
  ```
- **Skill carregada** → `Skill(skill: "<name>", args: ...)`. **Skill não carregada** → leia o `SKILL.md` e siga as instruções você mesmo.

### Prompt de cada tarefa

Autossuficiente (o agente começa sem contexto):
- objetivo da tarefa e o "Pronto quando";
- arquivos/caminhos relevantes e **quais arquivos ele pode alterar** (os da coluna de escrita — nada além sem justificar);
- resultados resumidos das tarefas de que depende;
- com perfil: stack em uma linha, comandos de build/teste/lint relevantes e arquivos de referência do padrão; sem perfil: peça que descubra os comandos nos manifestos/CI;
- formato de retorno:
  ```
  Retorne: (1) o que foi feito, (2) arquivos alterados/lidos com caminho, (3) verificação executada e resultado, (4) pendências ou riscos.
  ```

### Registrar no plano (após cada tarefa)

Atualize o arquivo do plano a cada conclusão — é a memória do que foi implementado:
- status na tabela de tarefas;
- **Resultado** no detalhe da tarefa (resumo curto, arquivos, verificação);
- linha no **Registro de execução** (`AAAA-MM-DD HH:MM · T3 concluída por orch:desenvolvedor`).

Agentes devolvem relatórios que o usuário não vê: extraia o essencial.

## Passo 5 — Verificar e iterar

- Tarefa falhou ou não atende ao "Pronto quando": reenvie ao mesmo agente (SendMessage, mantendo o contexto) com o problema específico, no máximo 2 vezes. Depois, marque `falhou`, registre o motivo e decida: replanejar (adicionar/alterar tarefas no arquivo, registrando a mudança) ou parar.
- Tarefas que dependem de uma `falhou` ficam `pendente` (bloqueadas) — não as execute. Tarefas independentes continuam.
- Nunca declare sucesso que não foi verificado.

## Passo 6 — Encerrar e relatar

1. Status final do plano: `concluido` (todas concluídas), `parcial` (alguma falhou/pulada) ou `cancelado`. Atualize cabeçalho, índice (status e `x/y` tarefas) e preencha **Resultado final**.
2. Responda ao usuário com:
   - **Resultado** em 1–3 frases;
   - tabela curta: tarefa → executor → status (✅ / ⚠️ / ❌);
   - arquivos alterados (links markdown relativos);
   - pendências, riscos e suposições;
   - link para o(s) arquivo(s) de plano.
3. Se a demanda revelou a falta de um especialista que seria útil de novo, sugira `/orch:criar-agente`.

Recomende versionar `.claude/orch/planos/` no git do projeto: é o histórico de decisões e implementações.
