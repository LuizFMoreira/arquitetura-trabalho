# Plano de trabalho — Desafio 09 (SDUI)

Objetivo: 25/25. A nota premia **coerência** entre requisitos, diagramas, decisões e falhas, não volume. Todo documento deve referenciar os mesmos IDs (RF1–5, RNF1–4, D1–Dn, F1–Fn).

## Mapa nota → entrega

| Critério (pts) | O que entrega | Arquivo |
|---|---|---|
| Problema e requisitos (4) | Contexto, RF/RNF, restrições, hipóteses, glossário | `docs/01-problema-requisitos.md` |
| Arquitetura (7) | Contrato, catálogo versionado, componentes, segurança de ações, fallback | `docs/02-contrato-sdui.md`, `docs/03-arquitetura.md` |
| Diagramas e fluxos (5) | Componentes; sequência (principal + exceção); fluxo de validação | `diagramas/`, `docs/04-fluxos.md` |
| Justificativas, trade-offs, falhas (5) | ≥ 2 decisões (usar 3–4), ≥ 3 falhas (usar 5), rollout/rollback, métricas e alertas | `docs/05-…`, `06-…`, `07-…`, `08-…` |
| Apresentação e arguição (4) | Slides, roteiro, Q&A, ensaio | `slides/` |

## Agentes de revisão (`.claude/agents/`)

| Agente | Quando usar | Pergunta que responde |
|---|---|---|
| `arquiteto-senior` | Após cada documento técnico | "Tem furo técnico?" |
| `professor-avaliador` | Fim de cada fase e antes da entrega | "Que nota levaríamos? Falta algo obrigatório?" |
| `analista-mercado` | Fase 1 (referências) e antes dos slides | "Isso é realista? Que fontes sustentam?" |
| `apresentacao-senior` | Quando houver rascunho dos slides | "Está claro e cabe em 20 min?" |
| `roteirista-senior` | Quando os slides estiverem estáveis | "O que cada um fala? Como respondemos?" |

## Fases

### Fase 0 — Setup (hoje)
- [ ] Todos os 4 aceitaram o convite do repositório.
- [ ] Estrutura de pastas + README + este plano commitados.
- [ ] Definir **cenário concreto** do exemplo (sugestão: jornada de *contratação de serviço* com formulário e pagamento, tela crítica).
- [ ] Definir formato dos slides (sugestão: Marp ou Google Slides → PDF).
- [ ] Dividir responsáveis (abaixo).

### Fase 1 — Fundamentos
- [ ] `01-problema-requisitos`: RF/RNF, restrições, hipóteses e riscos.
- [ ] `referencias.md` com fontes verificadas (rodar `analista-mercado`).
- [ ] Glossário (SDUI, BFF, capability, contrato, fallback).
- **Gate:** `professor-avaliador` na fase.

### Fase 2 — Núcleo da arquitetura
- [ ] `02-contrato-sdui`: schema de tela, componente, ação, versão; regra de campos desconhecidos; `minAppVersion` / `requiredCapabilities`.
- [ ] Catálogo de componentes com versão e fallback por componente.
- [ ] `03-arquitetura` + diagrama de componentes.
- [ ] Modelo de segurança: allowlist de ações tipadas, assinatura do payload, validação de schema, autorização no backend.
- [ ] Exemplos JSON em `exemplos/` (tela válida, inválida, componente novo em app antigo).
- **Gate:** `arquiteto-senior` + `professor-avaliador`.

### Fase 3 — Fluxos, operação e falhas
- [ ] `04-fluxos`: busca → validação → renderização → erro; diagrama de sequência (principal + exceção).
- [ ] `05-rollout-rollback`: canário por % / segmento / versão, kill switch, última config boa em cache, **exemplo de mudança de jornada sem publicar versão**.
- [ ] `06-decisoes`: usar 3–4 decisões (ex.: JSON próprio × GraphQL/protocolo existente; validação no cliente × só no servidor; fallback por componente × por tela; payload assinado × TLS apenas).
- [ ] `07-falhas`: 5 falhas com impacto, detecção e recuperação.
- [ ] `08-metricas-alertas`: métrica, limiar (como **hipótese**), alerta, ação.
- **Gate:** `arquiteto-senior` + `professor-avaliador`.

### Fase 4 — Slides e roteiro
- [ ] Rascunho dos slides (12–16 de conteúdo).
- [ ] Revisão do `apresentacao-senior`.
- [ ] Roteiro + banco de Q&A (`roteirista-senior`) em `slides/roteiro.md`.
- [ ] Divisão das falas entre os 4.

### Fase 5 — Ensaio e entrega
- [ ] Ensaio cronometrado ≥ 2x (alvo 18–19 min).
- [ ] **Arguição cruzada**: cada um responde sobre a parte dos outros.
- [ ] Rodada final do `professor-avaliador`.
- [ ] Exportar PDF, conferir referências, tag no git `entrega`.

## Divisão sugerida (preencher nomes)

| # | Responsável | Foco |
|---|---|---|
| 1 | _a definir_ | Problema, requisitos, contrato e catálogo |
| 2 | _a definir_ | Arquitetura, segurança de ações, diagrama de componentes |
| 3 | _a definir_ | Fluxos, sequência, rollout/rollback, exemplo de jornada |
| 4 | _a definir_ | Decisões, falhas, métricas/alertas, slides e referências |

Todos revisam e todos precisam dominar o conjunto.

## Regras do time
- Uma branch por pessoa/tema; PR na `main` com 1 revisão.
- Commits pequenos, mensagem `docs: …` / `diagrama: …` / `slides: …`.
- IDs consistentes (RF, RNF, D, F) em todos os arquivos.
- Número sem medição = "hipótese".
- Referência só entra se tiver URL verificada.
- Se usar IA, conferir e saber explicar.

## Riscos do próprio trabalho
| Risco | Mitigação |
|---|---|
| Documentos incoerentes entre si | IDs padronizados + gate do professor-avaliador |
| Arquitetura vaga ("só um JSON") | Contrato concreto + exemplos válidos/inválidos |
| Segurança superficial | Allowlist de ações + assinatura + autorização no backend |
| Estourar 20 min | Ensaio cronometrado, corte de slides |
| Alguém não dominar o todo | Arguição cruzada na Fase 5 |
