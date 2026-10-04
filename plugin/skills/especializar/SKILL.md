---
name: especializar
description: Especializa o orch para o projeto atual, em qualquer linguagem ou stack (web, jogos, mobile, desktop, backend...). Analisa o código existente, gera o perfil do projeto (stack, comandos, arquitetura, convenções) e propõe agentes especialistas específicos do projeto para o usuário aprovar. Use quando o usuário pedir para especializar, calibrar, "treinar" ou adaptar os agentes ao projeto, ou para atualizar o perfil.
argument-hint: "[--atualizar | --novo \"<stack pretendida>\" | --so-perfil]"
---

# Especializar o orch para este projeto

Argumentos: **$ARGUMENTS**

Templates: `${CLAUDE_PLUGIN_ROOT}/templates/perfil.md` e `${CLAUDE_PLUGIN_ROOT}/templates/especialista.md`.
Versão do orch: leia `version` em `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json`.

Saídas no projeto:
- `.claude/orch/perfil.md` — perfil do projeto
- `.claude/agents/<nome>.md` — especialistas aprovados
- linha `@.claude/orch/perfil.md` no `CLAUDE.md` da raiz (assim a sessão principal e os subagentes recebem o perfil automaticamente)

Execute na sessão principal. Não altere código do projeto — esta skill só escreve os arquivos acima.

## Passo 0 — Escolher o modo

- `--novo "<stack>"` → **Modo novo** (fim deste arquivo).
- `--atualizar` → **Modo atualizar** (fim deste arquivo).
- Sem argumento:
  - Se `.claude/orch/perfil.md` já existe → pergunte (AskUserQuestion): atualizar (recomendado) ou refazer do zero.
  - Conte arquivos de código-fonte, ignorando dependências e saídas de build (`node_modules`, `bin`, `obj`, `target`, `dist`, `build`, `.godot`, `Library`, `vendor`, `.venv`...). Se houver menos de ~5, diga que não há código suficiente para analisar e ofereça o modo `--novo`.
  - Senão → **Modo análise** (Passos 1–5).
- `--so-perfil` → Modo análise, pulando os Passos 4–5.

## Passo 1 — Reconhecimento rápido (você mesmo)

Em poucas chamadas Glob/Read, levante:

1. **Marcadores de stack** na raiz e um nível abaixo. Exemplos (não exaustivo — qualquer manifesto conta):

   | Marcador | Indica |
   |---|---|
   | `package.json`, `tsconfig.json`, `vite.config.*`, `next.config.*` | JS/TS, web/node |
   | `*.sln`, `*.csproj`, `global.json` | .NET / C# |
   | `project.godot`, `*.gd`, `*.tscn` | Godot |
   | `ProjectSettings/`, `Assets/`, `*.unity` | Unity |
   | `*.uproject`, `Source/*.Build.cs` | Unreal |
   | `pyproject.toml`, `requirements.txt`, `manage.py` | Python (Django?) |
   | `Cargo.toml` · `go.mod` · `pom.xml`/`build.gradle*` · `composer.json` · `Gemfile` · `pubspec.yaml` | Rust · Go · JVM · PHP · Ruby · Dart/Flutter |
   | `CMakeLists.txt`, `Makefile`, `meson.build` | C/C++ |
   | `Dockerfile`, `docker-compose.*`, `.github/workflows/`, `azure-pipelines.yml` | infra / CI (ótima fonte de comandos reais) |

2. **Árvore** das pastas principais (2 níveis), README, `CLAUDE.md` existente e `git log --oneline -20` (se for repositório git).
3. **Commit atual**: `git rev-parse --short HEAD` (ou "sem-git").

Com isso, defina 3 frentes de análise e os caminhos relevantes de cada uma.

## Passo 2 — Análise em paralelo

Dispare **na mesma mensagem** três `orch:pesquisador` (somente leitura), cada um com prompt autossuficiente contendo a stack detectada e os caminhos do Passo 1:

1. **Stack e comandos** — versões exatas (dos manifestos/lockfiles), como instalar, compilar, rodar, testar (todos e um só) e formatar. Fontes prioritárias: scripts dos manifestos, CI, Makefile/README. Para cada comando, informe a fonte (`caminho:linha`).
2. **Arquitetura e módulos** — pontos de entrada, módulos/camadas e suas responsabilidades, fluxo principal, e para cada tipo de artefato recorrente (ex.: endpoint, cena, componente, entidade, sistema) **um arquivo de referência** que represente bem o padrão. Identifique domínios distintos (áreas com tecnologias, regras ou pastas próprias) e o tamanho aproximado de cada um.
3. **Convenções, testes e cuidados** — nomes, estilo/linters, tratamento de erros, padrão de commits, framework e localização dos testes com um exemplo a imitar, arquivos gerados, áreas sensíveis (migrations, assets binários, configs de produção, segredos) e armadilhas.

Peça a cada um: respostas com evidência (`caminho:linha`), sem copiar blocos grandes de código, e "não encontrado" quando for o caso.

## Passo 3 — Escrever o perfil

1. Preencha o template de perfil com os resultados. Regras:
   - **Referencie arquivos em vez de copiar código** — o código muda, a referência continua útil.
   - Nada inventado: o que não foi confirmado fica como "não encontrado" ou "não verificado".
   - Máximo ~150 linhas. É contexto carregado em toda sessão — seja denso.
2. **Verificação de comandos (opcional e barata):** execute apenas comandos rápidos e sem efeitos colaterais (ex.: listar testes, `--version`, lint de um arquivo). Não rode builds longos, não instale dependências, não suba servidores. Marque "verificado" só o que rodou com sucesso.
3. Grave em `.claude/orch/perfil.md` com o cabeçalho de metadados preenchido (`modo: analise`).
4. Garanta a linha `@.claude/orch/perfil.md` no `CLAUDE.md` da raiz (crie o arquivo se não existir; não duplique a linha; não altere o resto do conteúdo).

## Passo 4 — Propor especialistas

Proponha um especialista apenas para domínios que atendam **a todos** os critérios:
- têm tecnologia, regras ou padrões próprios que um agente genérico erraria;
- são uma parte relevante do código ou concentram tarefas recorrentes;
- têm fronteira clara (pastas/arquivos) para não se sobrepor a outro especialista.

Limites: **no máximo 5** por projeto (prefira 2–3). Projeto pequeno ou homogêneo pode não precisar de nenhum — diga isso; o perfil já basta. Não proponha especialistas que só repetem os genéricos (`orch:testador`, `orch:revisor`...) sem conhecimento específico.

Exemplos de bons especialistas: jogo Godot → `cenas-e-nodes`, `shaders`; web → `frontend-react`, `api`, `banco-e-migrations`; .NET em camadas → `dominio`, `infraestrutura`.

Apresente uma tabela (nome · domínio e caminhos · por que vale · ferramentas · somente leitura?) e pergunte com AskUserQuestion (`multiSelect: true`) quais criar. Se já existir em `.claude/agents/` um agente com o mesmo nome **não** gerado pelo orch, não sobrescreva: proponha outro nome.

## Passo 5 — Gerar os aprovados

Para cada especialista aprovado, preencha o template de especialista e grave em `.claude/agents/<nome>.md`:
- `description` com o domínio, tecnologias e **caminhos** — é o que o orquestrador usa para rotear.
- Regras aprendidas com arquivo de exemplo; arquivo de referência para criar coisas novas; comando de verificação do domínio.
- Ferramentas mínimas (domínio só de análise → `Read, Grep, Glob`).
- Mantenha o comentário `<!-- orch:especializar ... -->` logo após o frontmatter: é como o `--atualizar` reconhece os agentes que pode reescrever.

Atualize a seção "Especialistas do projeto" do perfil.

## Relatório

Informe: stack detectada, comandos encontrados (e quais foram verificados), especialistas criados, arquivos gravados (links), e que os especialistas ficam disponíveis pelo nome **na próxima sessão** (nesta, o `/orch:orquestrar` os usa por injeção). Sugira versionar `.claude/orch/` e `.claude/agents/` no git do projeto para o time compartilhar.

---

## Modo atualizar

1. Leia o cabeçalho de `.claude/orch/perfil.md` (commit e data). Sem perfil → rode o modo análise.
2. Se há git: `git diff --stat <commit>..HEAD` e `git log --oneline <commit>..HEAD`. Sem git: compare a árvore e os manifestos com o perfil.
3. Se as mudanças forem pequenas e localizadas, analise você mesmo só o que mudou; se forem grandes (novas pastas, mudança de stack/manifestos), rode o Passo 2 só para as frentes afetadas.
4. Atualize o perfil (novo commit/data no cabeçalho) e reescreva **somente** agentes em `.claude/agents/` que contenham o comentário `orch:especializar`. Agentes escritos pelo usuário nunca são alterados.
5. Se surgiu um domínio novo que mereça especialista, ou um especialista perdeu o sentido, proponha criar/remover (com aprovação, como no Passo 4).
6. Relate o que mudou no perfil e nos agentes.

## Modo novo

Projeto sem código (ou quase): não há o que analisar, mas um perfil curto já orienta desde o primeiro arquivo.

1. Use a stack informada em `--novo`. Se faltar algo essencial (linguagem, framework/engine, tipo de app), pergunte com AskUserQuestion — no máximo 3 perguntas.
2. Opcional: um `orch:pesquisador` para levantar os comandos e a estrutura de pastas padrão/recomendada da stack na documentação oficial.
3. Grave o perfil com `modo: novo`, marcando comandos e estrutura como "recomendado (projeto ainda vazio)". Não crie especialistas.
4. Garanta o import no `CLAUDE.md` e recomende rodar `/orch:especializar --atualizar` quando o projeto tiver código real.
