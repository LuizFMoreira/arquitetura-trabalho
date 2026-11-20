# Desafio 09 — SDUI: o aplicativo que recebe funcionalidades sem nova publicação

Trabalho prático de **Arquitetura de Software** (5º período, Engenharia de Software — PUC Minas).
Projeto de arquitetura Server-Driven UI que preserva **compatibilidade, segurança e capacidade de reversão**. É um projeto no papel: o foco é projetar, documentar e defender decisões.

## Equipe
| Integrante | GitHub |
|---|---|
| Luiz F. Moreira | [@LuizFMoreira](https://github.com/LuizFMoreira) |
| _a preencher_ | [@karenjoilly11](https://github.com/karenjoilly11) |
| _a preencher_ | [@joaovictorz10](https://github.com/joaovictorz10) |
| Davi Nunes| [@Davii13](https://github.com/Davii13) |

## Estrutura
```
docs/          documentação técnica (um arquivo por item obrigatório) e PLANO.md
diagramas/     diagramas de componentes e de sequência (Mermaid)
exemplos/      exemplos de payload do contrato (válidos e inválidos)
slides/        apresentação HTML editável, roteiro e PDF
```

Comece por [docs/PLANO.md](docs/PLANO.md) (fases, responsáveis e regras do time).

## Materiais da proposta

- [Arquitetura](docs/03-arquitetura.md), [fluxos](docs/04-fluxos.md), [rollout e rollback](docs/05-rollout-rollback.md).
- [Decisões](docs/06-decisoes.md), [falhas](docs/07-falhas.md), [métricas](docs/08-metricas-alertas.md) e [referências](docs/referencias.md).
- Diagramas editáveis em Mermaid: [componentes](diagramas/componentes.md), [sequência](diagramas/sequencia.md), [validação](diagramas/validacao.md).
- [Apresentação em PDF](slides/apresentacao.pdf), [fonte HTML](slides/apresentacao.html) e [roteiro](slides/roteiro.md).

## Como contribuir
- Uma branch por tema/pessoa; PR na `main` com pelo menos 1 revisão.
- Mensagens de commit: `docs: …`, `diagrama: …`, `slides: …`, `chore: …`.
- IDs padronizados em todos os arquivos: `RF1–8`, `RNF1–5`, `D1…` (decisões), `F1…` (falhas), `M1…` (métricas). RF6–8 e RNF5 ampliam os mínimos do enunciado.
- Número sem medição é **hipótese** e deve ser marcado como tal.
- Referência só entra com URL verificada.
