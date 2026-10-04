# Changelog

Formato: [Keep a Changelog](https://keepachangelog.com/pt-BR/). Versão em `plugin/.claude-plugin/plugin.json`.

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
