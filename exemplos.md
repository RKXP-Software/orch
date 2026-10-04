# Prompts de teste do orquestrador

Use para validar o roteamento depois de mudar agentes/skills. Comece com `--plano` para ver a decisão sem executar.

| Prompt | Roteamento esperado |
|---|---|
| `/orch:orquestrar --catalogo` | Tabela com os 7 agentes `orch:*` + nativos + `criar-agente` |
| `/orch:orquestrar quanto é 2^10?` | Trivial → responde direto, sem delegar |
| `/orch:orquestrar --plano explique como o orquestrar decide a complexidade` | Simples → `orch:pesquisador` |
| `/orch:orquestrar --plano o marketplace.json não é reconhecido ao adicionar o marketplace` | Simples → `orch:depurador` |
| `/orch:orquestrar --plano crie um script que lista os agentes do plugin em tabela, com testes e README` | Composta → `orch:pesquisador` → `orch:desenvolvedor` → (`orch:testador` ∥ `orch:revisor`) → `orch:documentador` |
| `/orch:criar-agente agente tradutor que traduz documentação pt-BR/en` | Cria `.claude/agents/tradutor.md` no projeto |
