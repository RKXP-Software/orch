# Planejamento — execução manual por ondas e painel paralelo

Status: **implementado** (plugin 0.5.0 e app 0.4.0); este documento registra o desenho e as decisões. Escopo principal no repositório `OrchApp` (`F:\DESENVOLVIMENTO\Claude\OrchApp`), com uma mudança pequena no plugin (este repositório).

## Problema

Hoje o app executa o plano inteiro com `/orch:orquestrar --executar <id>` numa única sessão: o plugin decide sozinho o que roda em paralelo (até 4) e o usuário só escolhe um modelo para tudo. Consequências:

- Gasto de tokens sem controle (tudo automático).
- Não dá para trocar de modelo no meio do plano (ex.: Opus na onda 1, Sonnet nas demais).
- Não dá para escolher quais tarefas de uma onda rodam, nem rodar uma de cada vez.
- Não há como acompanhar tarefas paralelas individualmente: os subagentes ficam dentro de uma conversa só.

## Decisões de arquitetura

1. **O app passa a ser o agendador no modo manual.** Cada tarefa vira **uma sessão própria** do Agent SDK (`GerenciadorSessoes.iniciar`), com seu modelo, sua conversa e suas aprovações. É isso que viabiliza escolher modelo por onda e acompanhar cada tarefa isoladamente.
2. **Dois modos de execução do plano**, escolhidos no detalhe do plano (padrão vem das configurações):
   - **Automático** — comportamento atual (`--executar`), mantido.
   - **Manual por ondas** — novo, descrito abaixo.
3. **Quem escreve o JSON do plano no modo manual é o app**, não as sessões. Várias sessões editando `.claude/orch/planos/<id>.json` ao mesmo tempo gerariam corrida de escrita. O app marca `em-andamento` ao iniciar a tarefa e `concluida`/`falhou` ao terminar, grava `resultado` (resumo final da sessão), `tentativas` e eventos. Gravação serializada por plano (fila no main).
4. **Plugin ganha o modo `--tarefa <plano-id> <Tn>`** (versão 0.5.0): executa **só** aquela tarefa com o `executor` definido, sem agendar nem tocar no JSON, e termina com um resumo curto do resultado. O app monta esse prompt. Atualizar `CHANGELOG.md`, `version` em `plugin/.claude-plugin/plugin.json`, a skill `orquestrar` e o caso novo em `exemplos.md`. O contrato `orch.plano/1` não muda (só quem escreve).
5. **Limite de paralelismo vale no processo principal**, não só na tela, e **nos dois modos**. Configuração `maxParalelo` (1–6, padrão 3) aplicada em `GerenciadorSessoes`. No manual, o que excede o limite entra numa **fila FIFO** no main e é disparado quando uma sessão termina. No automático, o limite é passado ao plugin (`--paralelo <n>`, substitui o "até 4" fixo).
6. **Isolamento do trabalho é escolha do usuário**; o padrão é a **mesma pasta** (tarefas paralelas compartilham o diretório, com aviso de conflito de arquivos). Alternativas opcionais: branch dedicada ao plano ou git worktree por tarefa (ver Parte C).

## Parte A — Execução manual por ondas (aba Planos)

Em `Planos.tsx` / `DetalhePlano`:

- Seletor de modo: Automático | Manual por ondas.
- Em cada onda (coluna), cada cartão de tarefa ganha **checkbox**:
  - habilitado só se a tarefa está `pronta` (pendente, dependências concluídas) ou `falhou` (para tentar de novo);
  - desabilitado com dica ("depende de T2") nas demais;
  - **sem bloqueio por limite**: dá para marcar mais tarefas que `maxParalelo`; as que excedem entram numa **fila** e iniciam sozinhas, em ordem, quando uma vaga abre. O cartão mostra "Na fila (posição N)" e a fila pode ser cancelada por tarefa.
- Cabeçalho da onda: "Selecionar todas as prontas", **seletor de modelo da onda** e botão **Executar selecionadas (n)**. Sem seleção, o botão fica desabilitado.
- Modelo: três níveis, do mais específico ao mais geral — **modelo da tarefa** (seletor no próprio cartão), **modelo da onda** (aplicado às tarefas sem modelo próprio) e padrão das configurações. Trocável a qualquer momento, o que permite Opus → Sonnet no meio do plano. O modelo escolhido por tarefa fica gravado na sessão e no evento do plano.
- Aviso (não bloqueio) se duas tarefas selecionadas têm `arquivos` sobrepostos: risco de conflito de escrita.
- Cartão de tarefa em execução mostra o modelo usado e um atalho "Abrir conversa".
- Tarefa `falhou`: ações "Tentar de novo" (incrementa `tentativas`) e "Pular" (`pulada`).
- Status do plano derivado: `em-execucao` enquanto houver sessão ativa, `parcial` se parou com pendências, `concluido` quando tudo termina.
- Ao reabrir o app com sessões que não existem mais, tarefa `em-andamento` sem sessão viva é marcada "interrompida" e volta a poder ser selecionada.

## Parte B — Acompanhar execuções paralelas

1. **Barra lateral direita da conversa**: ao abrir uma sessão, a aba da direita lista as tarefas do plano em execução (id, título, modelo, status, indicador de "aguardando você"). Clicar abre a conversa daquela tarefa.
2. **Visão em grade ("Paralelo")**: nova aba por projeto (ou modo dentro de Execuções) que mostra de **1 a 6 conversas lado a lado em colunas**.
   - Controle de layout 1–6 colunas; padrão = número de sessões ativas, limitado a 6.
   - Cada coluna é a conversa completa da tarefa (log, aprovações/perguntas, caixa de mensagem, interromper), em **modo compacto** — `Sessao.tsx` (537 linhas) hoje tem painel lateral de 300 px; extrair um componente de conversa reutilizável e esconder o painel lateral no modo compacto.
   - Cabeçalho da coluna: tarefa, modelo, status, custo/contexto. Colunas com pendência (aprovação ou pergunta) ganham destaque e a notificação existente continua valendo.
   - Se houver mais sessões que colunas, as extras ficam num seletor; cada coluna escolhe qual sessão mostra.
3. Atalho do detalhe do plano: "Ver em paralelo" abre a grade já com as sessões do plano.

## Parte C — Configurações

Em `Configuracoes.tsx` e `Configuracao` (`src/shared/tipos.ts`):

- `maxParalelo: number` (1–6, padrão 3), com validação no main ao salvar.
- `modoExecucaoPadrao: 'automatico' | 'manual'`.
- `isolamento: 'mesma-pasta' | 'branch' | 'worktree'` (padrão `mesma-pasta`), também ajustável por plano na hora de executar:
  - `mesma-pasta`: nada é criado; vale o aviso de conflito de arquivos.
  - `branch`: o app cria/troca para uma branch do plano (ex.: `orch/<id-do-plano>`) antes da primeira tarefa; tarefas continuam na mesma pasta.
  - `worktree`: uma worktree por tarefa; o merge de volta segue `mergeWorktree` (`manual` com botão, ou `automatico` ao concluir; conflito sempre interrompe e avisa). Fica para a fase 6.
- O limite `maxParalelo` vale também no modo automático (decidido).

## Mudanças por arquivo (mapa)

| Área | Arquivo | Mudança |
|---|---|---|
| Tipos | `src/shared/tipos.ts` | `Configuracao.maxParalelo/modoExecucaoPadrao`; `NovaSessao` e `Sessao` com `plano?` e `tarefa?` |
| Lógica pura | `src/shared/plano.ts` | `tarefasSelecionaveis(plano)`, `validarSelecao(plano, ids, rodando, max)`, `conflitosDeArquivos(plano, ids)`, `promptTarefa(plano, id)` |
| Main | `src/main/sessoes.ts` | limite `maxParalelo`; ligar sessão↔tarefa; ao encerrar, avisar o fim com resumo |
| Main | `src/main/planos.ts` | escrita do estado da tarefa (início, fim, resultado, evento) serializada por plano |
| Main/IPC | `src/main/index.ts`, `src/preload/*` | `planos:executarTarefas(projeto, planoId, ids, modelo)`, `planos:tentarDeNovo`, `planos:pular` |
| UI | `Planos.tsx` | checkboxes, seletor de modo, modelo por onda, botões |
| UI | `Sessao.tsx` → componente compacto | reaproveitar conversa; lista de tarefas na aba direita |
| UI | novo `Paralelo.tsx` + `App.tsx` | aba/visão em grade 1–6 colunas |
| UI | `Configuracoes.tsx`, `Inicio.tsx` (tutorial) | novos campos e explicação |
| Plugin | `plugin/skills/orquestrar`, `CHANGELOG.md`, `plugin.json`, `exemplos.md` | modo `--tarefa`, parâmetro `--paralelo <n>`, v0.5.0 |

## Fases sugeridas (cada uma entrega valor sozinha)

1. **Plugin 0.5.0**: modo `--tarefa`, parâmetro `--paralelo` + casos de teste em `exemplos.md`.
2. **Fundação no app**: tipos, `maxParalelo` (config + fila e validação no main), vínculo sessão↔tarefa, escrita serializada do estado da tarefa. Testes das funções puras em `tests/plano.test.ts`.
3. **Execução manual por ondas**: modo, checkboxes, fila, modelo por onda e por tarefa, retry/pular, recuperação de tarefa órfã.
4. **Painel de acompanhamento**: lista de tarefas na aba direita da conversa.
5. **Grade paralela 1–6**: extração do componente de conversa compacto + `Paralelo.tsx`.
6. **Isolamento e polimento**: opção `branch`, depois `worktree`; tutorial; aviso de conflito de arquivos.

## Critérios de pronto

- Plano com onda 1 (1 tarefa) e onda 2 (3 tarefas): executo a T1 com Opus, depois marco só T2 e T3 e executo com Sonnet; T4 fica parada até eu marcar.
- Com `maxParalelo = 2`, marco 3 tarefas: duas iniciam e a terceira aparece "Na fila" e começa sozinha quando uma termina. Nunca há mais de 2 sessões ativas, nem no modo automático.
- Tarefa com modelo próprio usa esse modelo; sem modelo próprio, usa o da onda.
- Três tarefas paralelas aparecem na grade em 3 colunas; consigo responder a uma aprovação em cada uma sem afetar as outras.
- Fechar e reabrir o app não deixa tarefa presa em `em-andamento`.
- O modo automático continua funcionando como hoje.
- `npm run typecheck`, `npm test` e `npx electron-vite build` passam; `claude plugin validate .` passa no plugin.

## Decisões tomadas

1. `maxParalelo` vale nos dois modos (no automático, repassado ao plugin).
2. Excedentes entram numa fila e iniciam quando houver vaga (nada fica bloqueado).
3. Padrão: mesma pasta; branch ou worktree são opções do usuário (configuração e por execução).
4. Modelo por tarefa entra na primeira versão, além do modelo por onda.
5. Merge da worktree de volta é escolha do usuário: `mergeWorktree: 'manual' | 'automatico'` (padrão `manual`), nas configurações e ajustável por execução. Manual = botão "Mesclar" no cartão da tarefa concluída; automático = o app mescla ao concluir e, em caso de conflito, para e avisa (tarefa fica "aguardando merge").

## Ainda em aberto

- Nada pendente.
