# 05 — Rollout, rollback e mudança de jornada

## Revisões e publicação

Cada tela tem uma revisão imutável (`revision`). Uma política separada escolhe qual revisão entregar por plataforma, versão do app, segmento e coorte estável. A coorte usa hash de um identificador pseudônimo estável, tela e sal da campanha; o mesmo usuário não alterna aleatoriamente entre revisões a cada busca. A política guarda revisão anterior, responsável, horário, motivo e resultado da validação.

Antes de ativar uma revisão, o publicador exige schema válido, catálogo e ações permitidas, teste da matriz de versões suportadas, exemplo de fallback e revisão humana para pagamento. Os JSONs em `../exemplos/` são demonstrativos: assinaturas como `<assinatura-destacada>` não passam na verificação real. Em produção, o BFF assina a resposta final, após segmentação.

## Mudança sem nova publicação do app

**Antes:** `contratacao.dados` pede endereço e data e leva a `contratacao.pagamento` (`../exemplos/tela-contratacao-valida.json`).

**Depois:** uma nova revisão adiciona a foto opcional do local antes do pagamento (`../exemplos/componente-novo-app-antigo.json`). O app recente que conhece `PhotoUpload v1` e `camera.capture.v2` mostra a foto. Para app antigo, o BFF prefere entregar a alternativa `TextField v2` de descrição; caso receba a revisão nova por erro de segmentação, o app usa o fallback do componente. O campo não altera a autorização nem os dados obrigatórios do pagamento. A ordem da jornada muda por configuração; nenhum código novo é instalado. Um componente realmente novo ainda exige publicação de app para ser renderizado nativamente.

## Exposição progressiva

| Etapa | Coorte proposta (hipótese) | Critério para avançar |
|---|---:|---|
| Teste interno | equipe | Contrato e fallback verificados em Android/iOS e versões suportadas |
| Canário | 1% | 30 min sem alerta crítico, com volume mínimo definido após piloto |
| Expansão 1 | 10% | Erros e conclusão da jornada próximos do grupo controle |
| Expansão 2 | 50% | Mesmos critérios por plataforma e versão |
| Geral | 100% | Responsável registra decisão e mantém revisão anterior disponível |

Percentuais e duração são hipóteses de operação, não resultados medidos. Uma tela de pagamento requer observação separada, aprovação humana e possibilidade de interromper a etapa sem ampliar a coorte. A taxa de erro precisa ser lida com denominador de tentativas, não apenas contagem de eventos.

## Reversão

1. Alerta ou operador congela o avanço e aciona kill switch da revisão por tela e segmento.
2. Controle de rollout aponta novas buscas para a revisão anterior validada. O BFF deixa de servir a revisão problemática e registra a mudança.
3. Na próxima abertura online da tela, o app busca a política atual. Para tela crítica, revalida antes de renderizar e nunca usa revisão revogada em cache. Se o BFF estiver indisponível, abre a tela embutida e a API de pagamento continua exigindo conexão.
4. Operação acompanha `revision` servida, fallback, erros e conclusão da jornada até estabilizar. A revisão defeituosa fica bloqueada para republicação automática.

A meta de **até 5 min para a maior parte dos usuários online ativos** é hipótese de RNF3. Ela exige propagação rápida do kill switch e nova busca ao abrir tela crítica; não garante atualizar telas já abertas, usuários offline ou quem não reabre a jornada. O ensaio do rollback deve medir tempo entre acionamento, mudança no BFF e queda da exposição no cliente. O resultado precisa informar cobertura e p95, não apenas o tempo do servidor.

## Teste de mesa proposto

Publicar revisão nova no canário, injetar `PhotoUpload` incompatível e depois uma ação proibida. Confirmar: app antigo usa fallback do componente no primeiro caso; no segundo rejeita a tela inteira; kill switch interrompe novas entregas; revisão anterior volta; métricas identificam a revisão defeituosa. Testar também falta de rede e cache expirado. Nenhum desses resultados foi medido neste projeto no papel.
