# Fluxo de validação no cliente

```mermaid
flowchart TD
  A[Resposta recebida] --> B{Assinatura, kid e prazo válidos?}
  B -- Não --> Z[Rejeitar tela remota]
  B -- Sim --> C{Schema major e limites suportados?}
  C -- Não --> Z
  C -- Sim --> D{Estrutura e ações permitidas?}
  D -- Não --> Z
  D -- Sim --> E{Componentes e capacidades disponíveis?}
  E -- Sim --> H[Salvar e renderizar]
  E -- Não --> F{Fallback de componente válido?}
  F -- Sim --> H
  F -- Não --> G{Componente opcional?}
  G -- Sim --> I[Omitir componente e renderizar]
  G -- Não --> Z
  Z --> J{Cache elegível e revisão não revogada?}
  J -- Sim --> K[Renderizar última definição válida]
  J -- Não --> L[Tela embutida de fallback]
```

O validador percorre toda a árvore antes de renderizar. Um fallback de componente deve respeitar os mesmos limites, schema e allowlist; a omissão de componente não pode eliminar campo obrigatório da operação de pagamento.
