# Orch — desenvolvimento do plugin

Este repositório é o plugin **orch** (orquestrador de agentes) e o seu marketplace.

- O que é instalado nos projetos dos usuários fica em `plugin/`. Arquivos fora dele (README, exemplos, este CLAUDE.md) não são distribuídos.
- Componentes são namespaced: skills viram `/orch:<nome>`, agentes viram `orch:<nome>`. Referências cruzadas nos `.md` devem usar o nome com prefixo.
- Para caminhos de arquivos do próprio plugin dentro de skills/agentes, use `${CLAUDE_PLUGIN_ROOT}`.
- Ao mudar o comportamento: atualize `CHANGELOG.md` e incremente `version` em `plugin/.claude-plugin/plugin.json` (usuários só recebem a atualização quando a versão muda).
- Testar localmente sem publicar: `claude --plugin-dir ./plugin`, ou `/plugin marketplace add F:/DESENVOLVIMENTO/Claude/Orch` (marketplace local carrega os arquivos no lugar). Valide com `claude plugin validate .` e rode os casos de `exemplos.md`.

@F:/DESENVOLVIMENTO/Claude/doc/CLAUDE.md
