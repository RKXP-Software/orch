# Orch — orquestrador de agentes para Claude Code

Plugin que recebe uma demanda em linguagem natural, classifica, monta um plano e **delega cada etapa ao especialista certo**, rodando em paralelo o que for independente.

```
/orch:orquestrar adicione autenticação JWT na API, com testes e documentação
```

## O que vem no plugin

| Tipo | Nome | Função |
|---|---|---|
| Skill | `/orch:orquestrar` | Classifica, planeja, delega, verifica e relata. Modos `--catalogo` e `--plano` |
| Skill | `/orch:especializar` | Analisa o projeto e adapta o orch a ele: perfil + especialistas do projeto |
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

### Atualizações

Novas versões chegam sem reinstalar, quando o `version` do `plugin.json` muda:

- **Automático (recomendado):** `/plugin` → aba *Marketplaces* → `orch-marketplace` → *Enable auto-update*. Marketplaces de terceiros vêm com auto-update **desligado** por padrão.
- **Manual:** `claude plugin update orch@orch-marketplace` no terminal, ou pela aba *Installed* do `/plugin`.

Depois de atualizar, rode `/reload-plugins` ou abra uma nova sessão.

### Pré-configurar um projeto para o time

Adicione ao `.claude/settings.json` do projeto. Quem abrir o projeto recebe o convite para instalar, e as atualizações chegam sozinhas (`autoUpdate`):

```json
{
  "extraKnownMarketplaces": {
    "orch-marketplace": {
      "source": { "source": "github", "repo": "RKXP-Software/orch" },
      "autoUpdate": true
    }
  },
  "enabledPlugins": {
    "orch@orch-marketplace": true
  }
}
```

## Especializar para o seu projeto

Os agentes do orch funcionam em qualquer linguagem. Para que conheçam **o seu** projeto (stack, comandos, arquitetura, convenções), rode na raiz dele:

```
/orch:especializar
```

1. Detecta a stack (web, .NET, Godot, Unity, Python, Rust, C++...) e analisa o código com 3 pesquisadores em paralelo.
2. Grava `.claude/orch/perfil.md` e o importa no `CLAUDE.md` — todos os agentes passam a segui-lo.
3. Propõe até 5 especialistas do projeto (ex.: `cenas-e-nodes`, `api`, `banco-e-migrations`); você escolhe quais criar em `.claude/agents/`.

| Variação | Uso |
|---|---|
| `/orch:especializar --atualizar` | Atualiza perfil e especialistas gerados após mudanças no código (os seus agentes manuais não são tocados) |
| `/orch:especializar --novo "Godot 4 + C#"` | Projeto ainda sem código: perfil a partir da stack pretendida |
| `/orch:especializar --so-perfil` | Só o perfil, sem especialistas |

Versione `.claude/orch/` e `.claude/agents/` no git do projeto para o time compartilhar.

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
│  ├─ skills/especializar/
│  └─ templates/                     modelos de agente, skill, perfil e especialista
├─ exemplos.md                       prompts de teste do roteamento
├─ CHANGELOG.md
└─ CLAUDE.md                         instruções para desenvolver o plugin
```
