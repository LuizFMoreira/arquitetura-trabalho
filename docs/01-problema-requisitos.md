# 01 — Problema, requisitos e restrições

## 1. Contexto
Um aplicativo de serviços tem **milhões de instalações em Android e iOS**. O negócio quer alterar **formulários, jornadas e componentes visuais** sem esperar a publicação de uma nova versão nas lojas.

## 2. O problema central
Quatro forças conflitam:

| Força | Consequência |
|---|---|
| Agilidade: mudar a UI sem passar pelas lojas | O servidor passa a controlar o que o app mostra e faz |
| Fragmentação: milhões de apps em versões diferentes | Apps antigos **não entendem** componentes e capacidades novas |
| Risco operacional: uma configuração errada chega a todos de uma vez | Pode **bloquear uma tela crítica** (ex.: pagamento) sem forma rápida de corrigir |
| Superfície de ataque: o servidor descreve ações | Payload malicioso ou corrompido poderia disparar ação indevida |

**Pergunta de projeto:** como entregar flexibilidade de servidor *sem* abrir mão de compatibilidade, segurança e reversão?

## 3. Objetivo
Projetar uma arquitetura **Server-Driven UI (SDUI)** que preserve compatibilidade, segurança e capacidade de reversão. É projeto no papel: documentar e defender decisões, não implementar.

## 4. Por que SDUI (e não as alternativas)
Resumo para a defesa. A decisão completa, com trade-offs, entra em `06-decisoes` (D1).

| Alternativa | Por que não basta |
|---|---|
| Release nativo mais frequente | Continua dependendo de revisão das lojas e da adoção pelo usuário; apps antigos permanecem antigos |
| Feature flags | Só ligam e desligam o que já está no binário; não mudam composição de tela nem ordem da jornada |
| WebView / telas web | Experiência menos nativa, mais difícil de integrar com capacidades do dispositivo e com superfície de ataque maior (conteúdo e script remotos) |
| **SDUI com catálogo fechado** | Muda composição, texto, ordem e regras de jornada usando só o que o binário já sabe renderizar. Custo: complexidade de contrato e governança |

## 5. Atores

| Ator | Interesse |
|---|---|
| Usuário final | Completar a jornada sem erro, em qualquer versão do app |
| Time de produto/negócio | Alterar formulários e jornadas rapidamente |
| Time de plataforma (backend/BFF) | Manter contrato, validação, segmentação e rollout |
| Times mobile (Android/iOS) | Manter renderizador e catálogo de componentes |
| Operação/SRE | Detectar problema e reverter rápido |
| Segurança | Impedir execução indevida a partir do payload |

## 6. Cenário de referência
Será usado em contrato, exemplos, falhas e rollout. **Jornada de contratação de serviço**: o usuário escolhe um serviço, preenche um formulário de dados e confirma o pagamento. A tela de **pagamento é crítica** (H4). O exemplo de mudança sem nova publicação (doc 05) adiciona um passo opcional ao formulário.

## 7. Requisitos funcionais
Prioridade: **M** = deve ter, **S** = deveria ter.

| ID | Requisito | Prio. | Observação de projeto |
|---|---|---|---|
| RF1 | Obter do servidor a definição estruturada de telas e componentes | M | Contrato versionado (`02-contrato-sdui`) |
| RF2 | Renderizar componentes conhecidos em diferentes versões do app | M | Catálogo versionado; cada app só renderiza o que conhece |
| RF3 | Permitir experiências segmentadas por perfil ou versão | M | O cliente informa versão e capacidades; o servidor escolhe a definição compatível |
| RF4 | Executar ações somente a partir de contratos autorizados | M | Allowlist de ações tipadas no cliente (camada 1 de segurança, ver §10) |
| RF5 | Apresentar telas de fallback para configurações inválidas ou incompatíveis | M | Níveis de fallback definidos em `04-fluxos` |
| RF6 | Validar a definição de tela **no cliente** antes de renderizar | M | Schema, versão, componentes e ações; rejeitar o que for inválido |
| RF7 | Tratar funcionalidades cujas capacidades nativas estão ausentes | M | Capacidades declaradas pelo cliente; sem a capacidade, o servidor oferece alternativa ou o cliente usa fallback |
| RF8 | Manter a **última definição válida** em cache e usá-la quando o servidor falhar ou estiver sem rede | S | Base do rollback no cliente e do comportamento offline |

## 8. Requisitos não funcionais
Valores numéricos são **hipóteses** (H5): não foram medidos e servem de meta de projeto.

| ID | Requisito | Critério de aceitação | Verificação |
|---|---|---|---|
| RNF1 | Compatibilidade retroativa de contratos | Nenhuma mudança de contrato quebra versões do app ainda suportadas; campos desconhecidos são ignorados | Testes de contrato contra as versões suportadas |
| RNF2 | Limitar execução não autorizada a partir do servidor | 100% das ações executadas pertencem à allowlist; payload sem código executável; falha de assinatura ou schema bloqueia a renderização | Testes negativos com payloads maliciosos; log de ações rejeitadas |
| RNF3 | Rollback rápido | Config problemática revertida para a maior parte dos usuários em até **5 min** (hipótese), sem publicar app | Simulação de rollback; kill switch; TTL de cache |
| RNF4 | Medir erros de renderização, desempenho e disponibilidade | Métricas por tela, componente, versão e plataforma, com alerta e ação associada | Painel e alertas em `08-metricas-alertas` |
| RNF5 | Desempenho aceitável da obtenção e renderização | Meta inicial: p95 abaixo de **1 s** (hipótese) para obter e renderizar uma tela | Medição em protótipo ou piloto |

## 9. Restrições do enunciado
1. **SDUI não adiciona código nativo arbitrário sem atualização do app.** O servidor só compõe o que o binário já sabe renderizar e executar.
2. É obrigatório **definir catálogo e versionamento de componentes**.
3. É preciso **considerar funcionalidades que dependem de capacidades nativas ausentes** (ex.: câmera, biometria, versão mínima do SO). Tratadas em RF7.

## 10. Hipóteses de projeto e decisões em aberto

| ID | Hipótese | Situação |
|---|---|---|
| H1 | O app envia ao servidor versão, plataforma e **versões de componentes e capacidades** que suporta | Base do RF3 e RF7 |
| H2 | Existe um backend intermediário (BFF) que monta e valida a definição | **Candidata a decisão** D2, com alternativas em `06-decisoes` |
| H3 | A base instalada tem cauda longa de versões antigas, e algumas nunca serão atualizadas | Premissa do problema |
| H4 | Telas **críticas** têm política de mudança mais restrita e fallback próprio | **Candidata a decisão** D3, com alternativas em `06-decisoes` |
| H5 | Nenhum número de desempenho deste trabalho foi medido | Todos os valores são hipóteses |

**Segurança em camadas (defesa em profundidade):** o servidor filtra a definição pela versão e capacidades do cliente (RF3); o cliente valida e só executa ações da allowlist (RF4, RF6); o backend autoriza de novo toda operação sensível, independentemente da UI (RNF2). A falha de uma camada não abre as demais.

## 11. Riscos

| ID | Risco | Mitigação prevista |
|---|---|---|
| R1 | Configuração errada derruba tela crítica | Validação, rollout progressivo, kill switch, cache da última boa (RF8) |
| R2 | Componente novo em app antigo | Negociação de versões, fallback por componente (RF2, RF5, RF7) |
| R3 | Payload tenta executar ação indevida | Allowlist, assinatura, autorização no backend (RF4, RNF2) |
| R4 | Contrato cresce até virar uma "linguagem de programação em JSON" | Catálogo fechado, sem lógica arbitrária, governança de mudanças |
| R5 | Difícil testar a combinação de versões × configurações | Testes de contrato, matriz de versões suportadas, rollout canário |
| R6 | Dívida de contrato: versões antigas impedem evoluir | Política de suporte e descontinuação de versões |

## 12. Escopo
**Dentro:** contrato, catálogo, validação, renderização, fallback, ações autorizadas, segmentação, rollout e rollback, métricas.
**Fora:** implementação, editor visual de telas para o negócio, regras de negócio de cada serviço.

## 13. Perguntas que a arquitetura precisa responder

| Pergunta | Resposta em |
|---|---|
| O que ocorre quando um componente novo chega a um app antigo? | `02-contrato-sdui`, `04-fluxos` |
| Como impedir que um payload de UI execute uma ação indevida? | `03-arquitetura` (segurança de ações) |
| Qual é o limite entre flexibilidade e complexidade da SDUI? | `06-decisoes` |

## 14. Glossário

| Termo | Significado neste trabalho |
|---|---|
| SDUI | Server-Driven UI: o servidor descreve a tela e o app a renderiza com componentes que já possui |
| Contrato | Schema versionado que define telas, componentes, ações e versões |
| Catálogo | Lista versionada dos componentes e ações que o app sabe renderizar e executar |
| Capacidade | Recurso nativo ou versão de componente que o dispositivo oferece (ex.: câmera, biometria) |
| BFF | Backend for Frontend: camada que monta e valida a definição de tela para o app |
| Fallback | Alternativa exibida quando um componente, tela ou configuração não pode ser usada |
| Rollout progressivo | Liberação gradual de uma configuração para fatias crescentes de usuários |
| Kill switch | Interruptor para desativar rapidamente uma configuração problemática |
| Allowlist | Lista fechada do que é permitido executar; o resto é rejeitado |

## 15. Rastreabilidade (preencher conforme cada documento sair)

| Requisito | Componente(s) | Decisão | Falha | Métrica |
|---|---|---|---|---|
| RF1 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |
| RF2 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |
| RF3 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |
| RF4 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |
| RF5 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |
| RF6 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |
| RF7 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |
| RF8 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |
| RNF1 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |
| RNF2 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |
| RNF3 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |
| RNF4 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |
| RNF5 | _TODO_ | _TODO_ | _TODO_ | _TODO_ |

## 16. Pendências
- [ ] Equipe confirmar o cenário de referência (§6) e as hipóteses H1–H4.
- [ ] Revisão do `arquiteto-senior` sobre este documento.
