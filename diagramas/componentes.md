# Diagrama de componentes — SDUI

```mermaid
flowchart LR
  subgraph Operacao[Publicação e operação]
    PUB[Publicador de configurações]
    REG[(Registro de revisões)]
    ROL[Controle de rollout e kill switch]
    OBS[Observabilidade]
  end
  subgraph Servidor[Servidor]
    BFF[BFF SDUI]
    KEY[Serviço de chaves]
    API[API de negócio e pagamento]
  end
  subgraph App[App Android ou iOS]
    NET[Busca e negociação]
    VAL[Validação de assinatura, contrato e ações]
    REN[Renderizador e catálogo local]
    CACHE[(Última definição válida)]
    EMB[Tela embutida de fallback]
  end
  PUB -->|revisão aprovada| REG
  PUB -->|política de exposição| ROL
  REG -->|revisão imutável| BFF
  ROL -->|seleção por coorte| BFF
  KEY -->|assinatura da resposta| BFF
  NET -->|versão, catálogo, capacidades| BFF
  BFF -->|payload assinado| VAL
  VAL -->|válido| REN
  VAL -->|salva após validar| CACHE
  CACHE -->|falha recuperável| REN
  VAL -->|sem alternativa segura| EMB
  REN -->|ação tipada| API
  API -->|resultado autorizado| REN
  BFF --> OBS
  VAL --> OBS
  API --> OBS
```

Setas indicam dependências lógicas, não protocolos específicos. O app não recebe a chave privada. A API de negócio autoriza a operação independentemente do resultado da validação local.
