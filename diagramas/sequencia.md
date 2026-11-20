# Diagrama de sequência — jornada e exceção

```mermaid
sequenceDiagram
    actor U as Usuário
    participant A as App SDUI
    participant B as BFF SDUI
    participant R as Rollout e registro
    participant K as Serviço de chaves
    participant P as API de pagamento
    U->>A: Abre contratação
    A->>B: Busca tela + versão, catálogo e capacidades
    B->>R: Resolve coorte e revisão compatível
    R-->>B: Revisão publicada
    B->>B: Valida resposta final
    B->>K: Assina JSON canônico
    K-->>B: Assinatura e kid
    B-->>A: Tela assinada + revision
    A->>A: Verifica assinatura, prazo, schema, componentes e ações
    alt Definição válida
        A->>A: Salva última definição válida e renderiza
        U->>A: Confirma pagamento
        A->>A: Valida ação submit_form
        A->>P: Pedido autenticado
        P->>P: Valida dados e autoriza
        P-->>A: Resultado da operação
        A-->>U: Confirmação ou erro de negócio
    else Configuração inválida ou incompatível
        A->>A: Registra motivo e verifica cache elegível
        alt Cache elegível e tela não revogada
            A-->>U: Renderiza última definição válida
        else Sem cache seguro
            A-->>U: Mostra tela embutida de fallback
        end
    end
```

Na tela crítica, a tentativa de usar cache exige consulta online da política de revogação. Se a consulta falhar, o app usa a tela embutida. O resultado de pagamento vem da API, nunca do payload SDUI.
