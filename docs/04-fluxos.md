# 04 — Busca, validação, renderização e erro

Os diagramas editáveis estão em `../diagramas/sequencia.md` e `../diagramas/validacao.md`. O fluxo implementa RF1, RF2, RF4–RF8 e RNF1–RNF4.

## Fluxo principal

1. Ao abrir `contratacao.dados`, o app informa plataforma, versão, schema, catálogo, ações e capacidades ao BFF. O perfil usado pelo servidor vem de autenticação confiável, não de uma string escolhida livremente pelo cliente.
2. O controle de rollout resolve coorte estável e revisão elegível. O BFF carrega a revisão, reduz a resposta ao que a versão suporta, valida o resultado final e o assina.
3. O app verifica assinatura, `kid`, prazo, `schemaVersion`, tamanho, estrutura, capacidades, componentes e ações. A ordem detalhada está no diagrama de validação.
4. Uma resposta aceita é armazenada como última definição válida, renderizada e identificada por `revision` nas métricas.
5. Ao acionar `submit_form`, o app valida tipo e parâmetros outra vez; a API de negócio autentica, valida os dados e autoriza a transação. A UI apenas solicita a operação.

## Exceção: componente incompatível ou payload inválido

O app tenta primeiro o fallback do componente, desde que esse fallback também passe na validação e não contenha ação recusada. Um componente opcional sem fallback pode ser omitido. Componente essencial sem alternativa, assinatura inválida, schema major não suportada ou ação não permitida invalida a tela remota inteira. Neste caso, o app procura uma última definição válida da **mesma tela e contexto**, ainda dentro de `expiresAt` e não revogada. Sem ela, mostra a tela embutida. Para pagamento, a confirmação exige rede e a tela embutida; não se confirma pagamento a partir de configuração vencida.

Um payload com `run_script` é recusado antes de qualquer toque. O exemplo em `../exemplos/payload-invalido-acao-nao-autorizada.json` tem também assinatura ausente e URL não permitida, logo falha já na primeira etapa. O cliente registra motivo principal e demais motivos detectáveis sem executar conteúdo.

## Política de cache e revisão

- Salvar somente após validação completa. Separar por conta, tela, plataforma, major do schema, versão do app e capacidades; limpar no logout.
- Para uso offline não crítico, exigir assinatura previamente válida, `expiresAt` futuro e idade no cache de no máximo 24 h (hipótese). O limite efetivo é o menor entre esses dois prazos.
- Ao abrir tela crítica online, consultar o BFF antes de mostrar a configuração remota. Se a revisão estiver revogada ou a busca falhar, usar a tela embutida. A API de pagamento sempre exige autenticação e conexão.
- Um kill switch pode desabilitar o uso de uma revisão no BFF imediatamente; a disseminação para clientes depende da próxima busca. Dispositivos offline ficam fora da meta de rollback em 5 min.

## Eventos mínimos

Cada tentativa registra `screenId`, `revision` quando conhecida, plataforma, versão do app, etapa, resultado e classe de erro. Eventos: `fetch_result`, `validation_rejected`, `component_fallback`, `embedded_fallback`, `action_rejected`, `payment_result`. A observabilidade agrega por coorte e versão, sem registrar campos preenchidos ou dados de pagamento. Limiar e resposta operacional estão em `08-metricas-alertas.md`.
