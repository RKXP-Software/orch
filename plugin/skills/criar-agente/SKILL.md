---
name: criar-agente
description: Cria um novo agente ou skill a partir dos templates do orch, no projeto atual ou no usuário, para que o orquestrador passe a delegar para ele. Use quando o usuário pedir para criar/adicionar um agente, especialista ou skill.
argument-hint: "<agente|skill> <nome> <o que ele faz> [--usuario]"
---

# Criar agente ou skill

Pedido: **$ARGUMENTS**

Templates: `${CLAUDE_PLUGIN_ROOT}/templates/agente.md` e `${CLAUDE_PLUGIN_ROOT}/templates/skill.md`.

1. Determine o tipo (agente = executa uma etapa isolada com contexto próprio; skill = procedimento reutilizável seguido na sessão atual). Se não estiver claro, recomende um e siga.
2. Destino:
   - padrão → projeto atual: `.claude/agents/<nome>.md` ou `.claude/skills/<nome>/SKILL.md`
   - `--usuario` → todos os projetos da máquina: `~/.claude/agents/` ou `~/.claude/skills/`
   - Nunca escreva dentro de `${CLAUDE_PLUGIN_ROOT}`: é o cache do plugin e é sobrescrito em atualizações. Para incluir o agente no próprio orch, ele deve ser adicionado no repositório do plugin (`plugin/agents/`).
3. Nome em kebab-case, em português. Verifique se já existe algo equivalente (lista de agentes da sessão, `.claude/agents`, `~/.claude/agents`); se existir, proponha especializar/atualizar em vez de duplicar.
4. Copie o template e preencha. Se o projeto tiver `.claude/orch/perfil.md` e o agente for de um domínio do projeto, use `${CLAUDE_PLUGIN_ROOT}/templates/especialista.md` (sem o comentário `orch:especializar`, pois foi pedido pelo usuário) e acrescente-o à seção "Especialistas do projeto" do perfil. A `description` é o que o orquestrador usa para rotear: diga **o que faz** e **quando usar**, com palavras que o usuário usaria na demanda.
5. Dê ao agente apenas as ferramentas necessárias (somente leitura = `Read, Grep, Glob`).
6. Informe ao usuário que o `/orch:orquestrar` já encontra o novo item nesta sessão (por injeção), e que ele fica disponível pelo nome a partir da próxima sessão.
