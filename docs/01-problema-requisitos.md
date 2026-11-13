# 01 — Problema, requisitos e restrições

## 1. Contexto
Um aplicativo de serviços tem **milhões de instalações em Android e iOS**. O negócio quer alterar **formulários, jornadas e componentes visuais** sem esperar a publicação de uma nova versão nas lojas.

## 2. O problema central
Duas forças conflitam:

| Força | Consequência |
|---|---|
| Agilidade: mudar a UI sem passar pelas lojas | Servidor passa a controlar o que o app mostra e faz |
| Fragmentação: milhões de apps em versões diferentes | Apps antigos **não entendem** componentes e capacidades novas |
| Risco operacional: uma configuração errada vai a todos de uma vez | Pode **bloquear uma tela crítica** (ex.: pagamento) sem forma rápida de corrigir |
| Superfície de ataque: o servidor descreve ações | Payload malicioso ou corrompido poderia disparar ação indevida |

**Pergunta de projeto:** como entregar flexibilidade de servidor *sem* abrir mão de compatibilidade, segurança e reversão?

## 3. Objetivo
Projetar uma arquitetura **Server-Driven UI (SDUI)** que preserve compatibilidade, segurança e capacidade de reversão. É projeto no papel: documentar e defender decisões, não implementar.

## 4. Requisitos funcionais

| ID | Requisito | Observação de projeto |
|---|---|---|
| RF1 | Obter do servidor a definição estruturada de telas e componentes | Contrato versionado (`02-contrato-sdui`) |
| RF2 | Renderizar componentes conhecidos em diferentes versões do app | Catálogo versionado; cada app só renderiza o que conhece |
| RF3 | Permitir experiências segmentadas por perfil ou versão | Segmentação resolvida no servidor, com `appVersion` e capacidades enviadas pelo cliente |
| RF4 | Executar ações somente a partir de contratos autorizados | Allowlist de ações tipadas no cliente |
| RF5 | Apresentar telas de fallback para configurações inválidas ou incompatíveis | Fallback por componente, por tela e embutido no app |

## 5. Requisitos não funcionais

| ID | Requisito | Como será verificável |
|---|---|---|
| RNF1 | Garantir compatibilidade retroativa de contratos | Regras de evolução do schema; testes de contrato contra versões antigas |
| RNF2 | Limitar possibilidades de execução não autorizada a partir do servidor | Sem código/script no payload; ações só via allowlist; autorização no backend |
| RNF3 | Permitir rollback rápido de configurações problemáticas | Rollback sem depender das lojas; tempo-alvo **(hipótese)** a definir em `05-rollout-rollback` |
| RNF4 | Medir erros de renderização, desempenho e disponibilidade | Métricas e alertas em `08-metricas-alertas` |

## 6. Restrições do enunciado
1. **SDUI não adiciona código nativo arbitrário sem atualização do app.** O servidor só compõe o que o binário já sabe renderizar e executar.
2. É obrigatório **definir catálogo e versionamento de componentes**.
3. É preciso **considerar funcionalidades que dependem de capacidades nativas ausentes** (ex.: câmera, biometria, versão mínima do SO).

## 7. Hipóteses de projeto
Hipóteses assumidas pela equipe; precisam ser sustentadas ou ajustadas nos demais documentos.

| ID | Hipótese |
|---|---|
| H1 | O app envia ao servidor sua versão, plataforma e **lista de capacidades/versões de componentes** que suporta. |
| H2 | Existe um backend intermediário (BFF) que monta e valida a definição de tela antes de entregá-la. |
| H3 | A base instalada tem cauda longa de versões antigas, e algumas nunca serão atualizadas. |
| H4 | Telas **críticas** (ex.: pagamento) têm fluxo de contingência próprio e política de mudança mais restritiva. |
| H5 | Nenhum número de desempenho neste trabalho foi medido; todos serão apresentados como **hipóteses**. |

## 8. Escopo
**Dentro:** contrato, catálogo, validação, renderização, fallback, ações autorizadas, segmentação, rollout/rollback, métricas.
**Fora:** implementação, motor de design visual, editor visual de telas (CMS) para o time de negócio, e regras de negócio de cada serviço.

## 9. Perguntas que a arquitetura precisa responder
Perguntas orientadoras do enunciado, com o documento que responderá cada uma:

| Pergunta | Resposta em |
|---|---|
| O que ocorre quando um componente novo chega a um app antigo? | `02-contrato-sdui`, `04-fluxos` |
| Como impedir que um payload de UI execute uma ação indevida? | `03-arquitetura` (segurança de ações) |
| Qual é o limite entre flexibilidade e complexidade da SDUI? | `06-decisoes` |

## 10. Rastreabilidade (a preencher nas próximas fases)

| Requisito | Componente(s) | Decisão | Falha relacionada | Métrica |
|---|---|---|---|---|
| RF1 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |
| RF2 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |
| RF3 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |
| RF4 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |
| RF5 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |
| RNF1 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |
| RNF2 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |
| RNF3 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |
| RNF4 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |

## 11. Pendências
- [ ] Definir o **cenário concreto** do exemplo de jornada (sugestão: contratação de serviço com formulário e pagamento).
- [ ] Revisão do `professor-avaliador` sobre este documento.
