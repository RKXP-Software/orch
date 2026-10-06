# Changelog

Formato: [Keep a Changelog](https://keepachangelog.com/pt-BR/). Versão em `plugin/.claude-plugin/plugin.json`.

## [0.4.0] — 2026-10-05

### Adicionado
- Estado do plano legível por máquina: `.claude/orch/planos/<id>.json` (schema `orch.plano/1`), gravado junto com o `.md` a cada mudança — status do plano e das tarefas, início/fim, resultados e eventos. É o contrato com o app [orch-app](https://github.com/RKXP-Software/orch-app).
- Template `plano.json`.

### Alterado
- O plano passa a ser atualizado também ao **iniciar** cada tarefa (`em-andamento` + horário), não só ao concluir.

## [0.3.0] — 2026-10-04

### Adicionado
- Planos persistidos: cada plano é salvo em `.claude/orch/planos/<id>.md` no projeto, com índice em `INDICE.md` — memória do que foi planejado e implementado.
- Decomposição: a demanda vira um ou mais planos (um por objetivo independente; fases para planos com mais de ~8 tarefas), cada um com tarefas T1..Tn, dependências, arquivos de escrita previstos e critério de pronto.
- Agendamento por dependências: tarefas prontas disparam em paralelo (até 4), e as liberadas começam assim que suas dependências terminam; conflitos de escrita viram dependência.
- Modos `--executar <id>` (executa/retoma plano salvo) e `--planos` (lista o índice).
- Template `plano.md`.

### Alterado
- `--plano` agora salva os planos com status `planejado` em vez de só exibir.
- O arquivo do plano é atualizado a cada tarefa (status, resultado, registro de execução) e ao final (resultado final).

## [0.2.0] — 2026-10-04

### Adicionado
- Skill `especializar`: analisa o projeto (qualquer linguagem/stack), gera `.claude/orch/perfil.md` e propõe até 5 especialistas do projeto para aprovação. Modos `--atualizar`, `--novo` e `--so-perfil`.
- Templates `perfil.md` e `especialista.md`.

### Alterado
- `orquestrar`: lê o perfil (Passo 0), prioriza especialistas do projeto no domínio deles, inclui comandos de build/teste nos prompts e sugere especializar/atualizar quando útil.
- Agentes genéricos seguem o perfil do projeto quando existir.
- `criar-agente` usa o template de especialista em projetos com perfil.

## [0.1.0] — 2026-10-04

### Adicionado
- Skill `orquestrar`: catálogo, classificação (trivial/simples/composta), plano com dependências e paralelismo, execução, verificação e relatório.
- Skill `criar-agente` e templates de agente/skill.
- Agentes: pesquisador, planejador, desenvolvedor, depurador, testador, revisor, documentador.
- Distribuição como plugin com marketplace próprio.
