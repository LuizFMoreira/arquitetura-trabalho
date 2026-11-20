# 07 — Quadro de falhas e recuperação

Cada cenário indica impacto observado, sinal de detecção e ação de recuperação. A ordem da validação aparece em `04-fluxos.md`.

| ID | Falha e impacto | Como detectar | Recuperação | Decisão |
|---|---|---|---|---|
| F1 | `PhotoUpload` chega a app antigo ou sem câmera; campo não renderiza | `component_fallback` por versão/capacidade cresce no canário | Usar `TextField` de descrição previsto no componente; se essencial e sem alternativa, tela embutida; corrigir seleção no BFF | D2, D3 |
| F2 | Payload contém `run_script`, URL proibida ou assinatura inválida; risco de ação indevida | `validation_rejected` e `action_rejected` por motivo, com alerta de segurança | Rejeitar tela inteira antes de executar; usar cache elegível ou tela embutida; bloquear revisão e investigar publicação | D4 |
| F3 | Revisão de pagamento omite informação essencial ou trava a confirmação; queda de conclusão | Erro de renderização, fallback de tela crítica e conclusão de pagamento por revisão/coorte | Kill switch, revisão anterior validada em novas buscas; se sem resposta segura, tela embutida. API nunca confirma por conta da UI | D1, D3, D4 |
| F4 | BFF ou rede indisponível; tela remota não chega | Disponibilidade e latência da busca, `fetch_result` e uso de cache | Tela comum usa última definição válida não expirada; pagamento usa tela embutida e informa indisponibilidade da confirmação enquanto sem conexão | D2, D3 |
| F5 | Segmentação ou cache entrega revisão incompatível ou revogada; parte dos usuários recebe jornada errada | Divergência entre catálogo declarado e payload, `revision` inesperada por coorte, aumento de rejeições | Cliente rejeita; operação bloqueia revisão/segmento, corrige política e testa matriz de versões. Cache da revisão revogada deixa de ser elegível em busca online | D2, D3 |

## Ensaios antes de uma adoção real

- Reproduzir cada falha em Android e iOS, incluindo a menor versão ainda suportada.
- Verificar que nenhuma ação sensível é executada no cenário F2 e que a API de pagamento recusa pedido sem autorização própria.
- Medir o tempo de F3/F5 desde o kill switch até cessar exposição nos clientes online. Registrar cobertura e usuários fora da meta (offline ou tela já aberta).
- Testar saída e entrada de conta para assegurar que F4 nunca reutiliza cache de outro usuário.
