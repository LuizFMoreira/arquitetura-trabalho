# 06 — Quadro de decisões arquiteturais

As decisões são propostas para o cenário do trabalho, não resultados de um sistema implantado. Cada alternativa explicita o custo que motivou a escolha.

| ID | Decisão e alternativas | Escolha e justificativa | Custo e condição de revisão |
|---|---|---|---|
| D1 | Evoluir jornadas por releases nativos, feature flags, WebView ou contrato SDUI de catálogo fechado | **SDUI de catálogo fechado.** O servidor altera composição e ordem usando componentes já presentes no binário, atendendo RF1–RF3 sem esperar a adoção de um release | Exige governança de contrato, testes entre versões e fallback. Se a jornada precisar de UI arbitrária ou lógica complexa, avaliar release nativo ou tela web delimitada, com análise de segurança |
| D2 | Cada app escolher uma configuração bruta ou um BFF selecionar por capacidades | **BFF seleciona e valida resposta final.** Centraliza compatibilidade, segmentação e assinatura; o cliente valida novamente | O BFF vira dependência de disponibilidade e pode errar a segmentação. Cache e tela embutida mitigam a falha; matriz de contrato verifica a escolha |
| D3 | Falhar toda a tela diante de qualquer componente novo ou permitir fallback por componente | **Fallback por componente para campos não críticos; tela embutida para ausência essencial.** Preserva a jornada de dados em app antigo (RF5, RF7) | Fallback pode ocultar um campo importante. O catálogo marca `optional`, e a tela de pagamento tem política mais restrita: não omitir dado obrigatório nem confirmar offline |
| D4 | Confiar apenas em TLS ou verificar também assinatura da definição | **Assinatura do JSON canônico após montagem pelo BFF**, além de TLS, schema e allowlist. O app detecta alteração do conteúdo antes de renderizar (RF4, RNF2) | Rotação de chaves e canonização elevam a complexidade. Assinatura não autoriza operações nem impede que um publicador autorizado publique erro; a API revalida tudo |

## Consequências para a implementação futura

- D1 limita SDUI a conteúdo, composição e ordem de componentes e ações conhecidos. Capacidade nativa nova só chega com release do app.
- D2 exige contrato de requisição de capacidades e matriz de versões suportadas. O perfil de negócio é derivado da autenticação no BFF.
- D3 exige fallback validado recursivamente e regra explícita para componente essencial. O pagamento não depende de configuração remota vencida.
- D4 exige formato de assinatura interoperável, chave pública no app e procedimento de rotação. A escolha de RFC 8785 resolve a representação determinística do JSON; o protocolo concreto de distribuição de chaves deve ser fechado antes de implementar.

## Rastreabilidade

| Decisão | Requisitos | Falhas relacionadas |
|---|---|---|
| D1 | RF1, RF2, RF3, RNF1 | F1, F3 |
| D2 | RF3, RF6, RNF3 | F1, F4, F5 |
| D3 | RF5, RF7, RF8 | F1, F3, F4 |
| D4 | RF4, RF6, RNF2 | F2, F3 |
