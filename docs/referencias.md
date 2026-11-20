# Referências técnicas consultadas

As fontes abaixo sustentam escolhas de método e segurança. Os percentuais, limites e tempos da proposta são hipóteses do grupo e não são apresentados como resultados dessas fontes.

| Fonte primária | Uso na proposta |
|---|---|
| [RFC 8785 — JSON Canonicalization Scheme](https://www.rfc-editor.org/rfc/rfc8785.html) | Define representação determinística do JSON antes de assinar e verificar o payload (D4) |
| [JSON Schema — documentação oficial](https://json-schema.org/docs) | Base para validação estrutural do contrato; validação semântica de ações e capacidades continua necessária |
| [OWASP MASVS-AUTH](https://mas.owasp.org/MASVS/07-MASVS-AUTH/) | Referência de autenticação e autorização no app móvel |
| [OWASP ASVS](https://asvs.owasp.org/) | Referência de controles da API e do BFF, inclusive autorização independente da UI |
| [Firebase Remote Config — rollouts](https://firebase.google.com/docs/remote-config/rollouts) | Exemplo real de exposição gradual, segmentação, monitoramento e rollback; a proposta não depende desse produto |
| [OpenTelemetry — signals](https://opentelemetry.io/docs/concepts/signals/) | Vocabulário de métricas e eventos para observabilidade da solução |

Consultadas em 20/11/2026. Antes da entrega, a equipe deve confirmar que os links permanecem acessíveis e alinhar a forma de citação ao padrão pedido pelo professor.
