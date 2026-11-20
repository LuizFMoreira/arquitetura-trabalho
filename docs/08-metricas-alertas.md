# 08 — Métricas, alertas e resposta

Todos os limiares abaixo são **hipóteses iniciais** (H5). Não há medição neste projeto no papel. Antes de adotá-los, um piloto precisa medir a linha de base, volume mínimo e variação normal por plataforma, versão e tela.

| ID | Métrica e cálculo | Limiar inicial proposto | Alerta e ação |
|---|---|---|---|
| M1 | Taxa de rejeição de payload = `validation_rejected / respostas recebidas`, por `revision` e versão | > 1% em 5 min com pelo menos 100 respostas | Pausar canário; inspecionar motivo; revogar revisão se afetar tela crítica |
| M2 | Taxa de fallback = `telas com fallback / aberturas de tela`, separando componente, cache e tela embutida | > 2% em 10 min ou dobro da coorte controle, com pelo menos 100 aberturas | Conferir catálogo e segmentação; voltar revisão se conclusão cair |
| M3 | Disponibilidade da busca = respostas válidas do BFF / buscas concluídas ou com falha | < 99,5% em 10 min, com pelo menos 200 buscas | Acionar operação do BFF; manter tela embutida; bloquear expansão |
| M4 | p95 do tempo entre abertura e tela renderizada, por plataforma/versão | > 1 s em 10 min com pelo menos 100 aberturas (RNF5) | Comparar com controle; verificar tamanho de payload, rede e renderização; pausar rollout |
| M5 | Conclusão de pagamento = confirmações autorizadas / tentativas iniciadas, por revisão e coorte | Queda relativa > 5% ante controle em 30 min com amostra suficiente definida no piloto | Kill switch da revisão crítica e investigação com produto e operação |
| M6 | Exposição após rollback = buscas que ainda recebem revisão revogada / buscas elegíveis após acionamento | > 5% após 5 min entre clientes online que rebuscaram (RNF3) | Verificar propagação de política, cache e BFF; forçar tela embutida se necessário |
| M7 | Ações recusadas = `action_rejected` por tipo, revisão e origem | Qualquer ação desconhecida em produção | Alerta de segurança, bloquear revisão e analisar trilha de publicação |

## Instrumentação e interpretação

- Eventos carregam `screenId`, `revision`, plataforma, versão do app, coorte e código de erro. Não enviam dados pessoais, campos do formulário ou payload integral.
- Painel compara canário e controle na mesma janela. Falha de observabilidade bloqueia expansão, pois inviabiliza verificar a revisão.
- Uma taxa sem denominador ou com amostra insuficiente não justifica conclusão estatística; operação pode interromper por risco evidente, mas deve registrar a incerteza.
- O relógio de rollback começa no acionamento do kill switch. Medir separadamente alteração no BFF e observação no cliente. Dispositivos offline e telas já abertas são contabilizados à parte.
- Teste de disponibilidade e latência deve usar dispositivos e condições de rede representativos; metas não medidas continuam hipóteses.

## Rastreabilidade

| Requisito | Métricas |
|---|---|
| RF2, RF5, RF7, RNF1 | M1, M2 |
| RF3 | M1, M2, M6 |
| RF4, RF6, RNF2 | M1, M7 |
| RF8, RNF3 | M2, M6 |
| RNF4 | M1–M7 |
| RNF5 | M3, M4 |
