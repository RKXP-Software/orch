<!-- orch:perfil · versao-orch: {versao} · analisado-em: {AAAA-MM-DD} · commit: {sha curto ou "sem-git"} · modo: {analise|novo} -->
# Perfil do projeto — {nome}

{1 parágrafo: o que o projeto é, para quem, estado atual (protótipo, produção, legado).}

## Stack

| Item | Valor | Evidência |
|---|---|---|
| Linguagem(ns) | {ex.: C# 12, GDScript} | {arquivo} |
| Framework / engine | {ex.: Godot 4.3, ASP.NET Core 8, React 18} | {arquivo} |
| Gerenciador de pacotes | {ex.: npm, NuGet, cargo} | {arquivo} |
| Banco / serviços | {se houver} | {arquivo} |

## Comandos

| Ação | Comando | Status |
|---|---|---|
| Instalar dependências | `{...}` | {verificado / não verificado} |
| Build | `{...}` | |
| Rodar | `{...}` | |
| Testes (todos) | `{...}` | |
| Teste único | `{...}` | |
| Lint / format | `{...}` | |

Status "verificado" = comando executado nesta análise com sucesso. Sem comando conhecido, escreva "não encontrado" — nunca invente.

## Estrutura

| Caminho | Responsabilidade |
|---|---|
| `{src/...}` | {...} |

## Arquitetura

- Pontos de entrada: `{caminho}` — {o que inicia}
- Fluxo principal: {em poucas linhas, ex.: input → controller → service → repositório}
- Padrões de referência (copie destes ao criar algo novo): `{caminho}` — {padrão}

## Convenções

- Nomes: {classes, arquivos, pastas}
- Estilo: {formatter/linter e regras relevantes, ou "seguir o código existente"}
- Erros / logs: {como o projeto trata}
- Commits / branches: {se houver padrão visível no git log}

## Testes

- Framework: {...} · Local: `{...}` · Exemplo a imitar: `{caminho}`
- {Como mockar dependências, fixtures, dados de teste}

## Cuidados

- {Arquivos gerados que não devem ser editados à mão}
- {Áreas sensíveis: migrations, assets binários, configurações de produção, segredos}
- {Armadilhas conhecidas}

## Especialistas do projeto

| Agente | Quando usar |
|---|---|
| `{nome}` | {domínio / caminhos} |
