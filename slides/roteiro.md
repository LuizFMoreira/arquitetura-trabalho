# Roteiro de apresentação e arguição

Versão para ensaio, com **18 min estimados de exposição** e margem de 2 min para transições. O limite do enunciado para grupo de quatro é 20 min de exposição mais 5 min de perguntas. Substituir Pessoa 1–4 pelos nomes confirmados; todos devem entender o conjunto.

| Responsável | Slides | Tempo estimado | Foco da fala |
|---|---|---:|---|
| Pessoa 1 | 1–4 | 4 min | Problema, mínimos do enunciado e jornada de contratação |
| Pessoa 2 | 5–8 | 5 min | Componentes, contrato, validação e sequência |
| Pessoa 3 | 9–12 | 4,5 min | Compatibilidade, segurança, rollout e rollback |
| Pessoa 4 | 13–16 | 4,5 min | Decisões, falhas, métricas e fontes |

## Guia por trecho

**Pessoa 1:** Abrir com o conflito entre mudança rápida e milhões de apps em versões diferentes. Explicar que a nova foto do local é opcional. O app antigo pode usar descrição por texto, e nenhum payload instala código nativo. Distinguir RF1–RF5/RNF1–RNF4, que vêm do professor, das extensões do grupo.

**Pessoa 2:** Apontar a direção do diagrama: publicador valida e registra; rollout escolhe; BFF monta e assina; app verifica e renderiza; API de negócio autoriza pagamento. No contrato, destacar `schemaVersion`, `revision`, `type`+`v`, `requires` e ações tipadas. Narrar a sequência principal e o caso de payload inválido. O diagrama completo está em `../diagramas/`.

**Pessoa 3:** Explicar as duas barreiras de compatibilidade: BFF escolhe configuração compatível e app valida. Para componente desconhecido, usar fallback de componente ou tela. Para ação proibida, rejeitar a tela inteira. Descrever coorte estável, canário e kill switch. A meta de 5 min vale apenas como hipótese para clientes online que fazem nova busca; uma tela já aberta ou aparelho offline não é revogado instantaneamente.

**Pessoa 4:** Defender duas decisões com mais detalhe (D2 e D3) e citar D1/D4. Ler o quadro de falhas pela lógica impacto → sinal → recuperação, sem narrar cada célula. Explicar que métricas têm denominador e comparação com controle. Encerrar com as três perguntas orientadoras: app antigo, ação indevida e limite da flexibilidade SDUI.

## Perguntas técnicas prováveis

**1. O que acontece quando um componente novo chega a um app antigo?** O BFF tenta selecionar uma revisão compatível. Se ainda receber tipo ou versão desconhecidos, o app valida o fallback do componente; se for opcional, pode omitir. Componente essencial sem alternativa leva à tela embutida. Nada executa código nativo novo.

**2. Como um payload malicioso executaria uma ação indevida?** A proposta reduz essa possibilidade por assinatura, schema, allowlist de ações e parâmetros no cliente. `run_script` não existe no catálogo e é rejeitado. Para pagamento, a API de negócio autentica e autoriza independentemente do payload.

**3. Qual é o limite de flexibilidade?** O servidor altera texto, ordem, composição e segmentação de componentes e ações existentes. Novo componente, novo acesso a câmera/biometria ou regra de negócio nativa exige atualização do app. O contrato não aceita scripts, laços nem expressões.

**4. Por que assinar se já há TLS?** TLS protege o canal. A assinatura permite ao app verificar que o conteúdo canônico corresponde ao que o BFF assinou. Ela não corrige erro de publicação, por isso há validação, canário e rollback.

**5. Como o rollback chega em cinco minutos?** É meta hipotética. O controle de rollout muda a revisão servida, e clientes online a recebem na próxima busca. Medimos separadamente a propagação no BFF e a exposição observada no app. Offline e telas já abertas são exceções explícitas.

**6. E se o BFF ficar fora no pagamento?** O app mostra a tela embutida. Pode orientar o usuário, mas não confirma o pagamento sem rede e sem autorização da API. Para tela comum, pode usar cache válido e não expirado da mesma conta/contexto.

**7. Qual evidência provaria que o canário é seguro?** Comparar rejeições, fallback, latência e conclusão de pagamento contra uma coorte controle no mesmo período, com amostra suficiente. Os limiares do trabalho ainda são hipóteses, então piloto e ensaio são necessários.

## Antes da apresentação

- Confirmar integrantes, divisão de falas e cenário da foto opcional.
- Ensaiar ao menos duas vezes com cronômetro; reduzir explicações repetidas se exceder 20 min.
- Cada integrante responder às três perguntas orientadoras sem consultar os próprios slides.
- Conferir os links de referência e a legibilidade do PDF final no projetor.
