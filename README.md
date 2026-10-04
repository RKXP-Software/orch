# Orch — orquestrador de agentes para Claude Code

Plugin que recebe uma demanda em linguagem natural, classifica, monta um plano e **delega cada etapa ao especialista certo**, rodando em paralelo o que for independente.

```
/orch:orquestrar adicione autenticação JWT na API, com testes e documentação
```

## O que vem no plugin

| Tipo | Nome | Função |
|---|---|---|
| Skill | `/orch:orquestrar` | Classifica, planeja, delega, verifica e relata. Modos `--catalogo` e `--plano` |
| Skill | `/orch:criar-agente` | Cria novos agentes/skills no projeto a partir dos templates |
| Agente | `orch:pesquisador` | Leitura: entender código, pesquisar na web |
| Agente | `orch:planejador` | Leitura: decompor e arquitetar |
| Agente | `orch:desenvolvedor` | Implementar |
| Agente | `orch:depurador` | Bugs e causa raiz |
| Agente | `orch:testador` | Escrever e rodar testes |
| Agente | `orch:revisor` | Leitura: revisão de bugs e segurança |
| Agente | `orch:documentador` | README e documentação |

## Instalação

Dentro do Claude Code (terminal ou app desktop):

```
/plugin marketplace add RKXP-Software/orch
/plugin install orch@orch-marketplace
```

Também aceita a URL git (`https://github.com/RKXP-Software/orch.git`) ou uma pasta local (`/plugin marketplace add F:/caminho/Orch`).

Na instalação, escolha o escopo:
- **user** — disponível em todos os seus projetos
- **project** — gravado em `.claude/settings.json` do projeto (vai para o git, o time recebe junto)
- **local** — só você, só neste projeto

Atualizar: `/plugin marketplace update orch-marketplace`.

### Pré-configurar um projeto para o time

Adicione ao `.claude/settings.json` do projeto. Quem abrir o projeto recebe o convite para instalar:

```json
{
  "extraKnownMarketplaces": {
    "orch-marketplace": {
      "source": { "source": "github", "repo": "RKXP-Software/orch" }
    }
  },
  "enabledPlugins": {
    "orch@orch-marketplace": true
  }
}
```

## Personalizar por projeto

O orquestrador usa também os agentes do próprio projeto (`.claude/agents/`) e do usuário (`~/.claude/agents/`), com prioridade sobre os do plugin. Para criar um: `/orch:criar-agente agente <nome> <o que faz>`.

## Estrutura do repositório

```
Orch/
├─ .claude-plugin/marketplace.json   catálogo (aponta para ./plugin)
├─ plugin/                           o plugin que é instalado
│  ├─ .claude-plugin/plugin.json     nome, versão, metadados
│  ├─ agents/                        7 especialistas
│  ├─ skills/orquestrar/  skills/criar-agente/
│  └─ templates/                     modelos de agente e skill
├─ exemplos.md                       prompts de teste do roteamento
├─ CHANGELOG.md
└─ CLAUDE.md                         instruções para desenvolver o plugin
```
