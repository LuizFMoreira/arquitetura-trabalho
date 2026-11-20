# 03 — Arquitetura e responsabilidades

## Escopo e cenário

A proposta atende RF1–RF8 e RNF1–RNF5 de `01-problema-requisitos.md`. A jornada de referência é a contratação de um serviço: dados do serviço, eventual foto do local e confirmação de pagamento. A composição das telas pode mudar no servidor; tipos de componentes, ações e capacidades nativas dependem de uma versão do app que os implemente.

O diagrama editável está em `../diagramas/componentes.md`. O contrato de dados e o catálogo inicial estão em `02-contrato-sdui.md`.

## Componentes

| Elemento | Responsabilidade | Limite de confiança |
|---|---|---|
| Publicador de configurações | Recebe proposta de revisão, valida schema, referências e matriz de versões; aprova e grava revisão imutável | Acesso administrativo com revisão humana para telas críticas |
| Registro de revisões | Guarda payloads versionados, revisões anteriores e trilha de auditoria | Somente publicador escreve; BFF lê |
| Controle de rollout | Escolhe revisão por tela, segmento, plataforma, versão e coorte estável; expõe kill switch | Política operacional, separada do conteúdo da tela |
| BFF SDUI | Recebe versão/catálogo/capacidades do app, escolhe configuração compatível, valida e assina a resposta exata | Não concede permissão de negócio |
| Serviço de chaves | Guarda chave privada de assinatura e publica identificadores de chave autorizados | Chave privada nunca chega ao app |
| Cliente SDUI no app | Busca, verifica assinatura, prazo, schema, capacidades e limites; renderiza catálogo fechado ou aplica fallback | Trata toda resposta remota como não confiável até validar |
| Cache local | Guarda última definição validada por tela e contexto compatível | Nunca transforma payload inválido em válido |
| API de negócio/pagamento | Revalida dados, autentica e autoriza a operação; devolve resultado da contratação | Não confia na UI para autorização |
| Observabilidade | Recebe eventos de publicação, busca, validação, fallback, ação recusada e resultado de jornada | Sem dados pessoais ou payload integral em logs |

## Caminho de publicação e consumo

1. O publicador valida uma revisão candidata contra schema, catálogo, ações e versões de app ainda suportadas. Configuração de tela crítica exige revisão adicional e teste da jornada de pagamento.
2. O registro guarda a revisão de forma imutável. O controle de rollout aponta para ela somente após aprovação.
3. O app informa plataforma, versão, catálogo, ações e capacidades. O BFF seleciona uma revisão compatível com esse conjunto e com o segmento elegível.
4. O BFF valida a resposta montada, inclui `revision`, `issuedAt` e `expiresAt`, e assina o JSON canônico. A assinatura cobre envelope sem `signature`, `screen` e `fallback`; o algoritmo e o `kid` ficam no objeto `signature`. A canonização segue RFC 8785. Os exemplos do repositório usam valor de assinatura fictício e não são payloads executáveis.
5. O app verifica a assinatura com chave pública embutida, prazo, versão de schema, limites, árvore de componentes e ações antes de renderizar. O app só salva no cache depois dessa validação.
6. Ao tocar em uma ação, o cliente valida novamente tipo e parâmetros. A API de negócio autentica e autoriza a operação sensível com seus próprios dados, inclusive o pagamento.

TLS protege o transporte; a assinatura permite detectar alteração do payload entre a assinatura no BFF e a validação no app. Ela não substitui autorização no backend nem protege contra um publicador comprometido. Para revogação imediata, o app precisa consultar a política remota; uma cópia offline não recebe kill switch.

## Compatibilidade e limites

- O servidor só escolhe `schemaVersion`, componentes e ações declarados pelo cliente. O app ainda valida tudo localmente (RF2, RF3, RF6).
- Campo opcional desconhecido é ignorado. Tipo, versão ou ação essencial desconhecida dispara o fallback definido no contrato (RF5, RF7). Um enum desconhecido só usa default se o catálogo o declarar seguro.
- `requiredCapabilities` do desafio é representado por `requires.capabilities` no contrato. Uma capacidade nativa ausente exige alternativa de componente, omissão opcional ou tela embutida. Não há execução de código enviado pelo servidor.
- O publicador impõe limites de profundidade, número de nós e bytes; o cliente repete os mesmos limites. Valores propostos, ainda não medidos: 12 níveis, 200 nós e 256 KiB por tela (H5).
- O cache usa chave por `screenId`, major do schema, plataforma, versão do app e conjunto de capacidades. Uma configuração personalizada não pode ser reaproveitada entre contas.

## Segurança e fallback

| Falha | Resposta do app |
|---|---|
| Assinatura ausente/inválida, schema major desconhecida, payload fora de limite ou ação não autorizada | Rejeita toda a resposta, registra evento sem dados sensíveis e usa última definição válida elegível; se não houver, tela embutida |
| Componente desconhecido ou capacidade ausente | Usa fallback do componente; se ele for opcional e sem fallback, omite; se essencial, abandona a tela remota |
| Rede indisponível | Para tela não crítica, usa última definição validada dentro de janela offline; para pagamento, abre tela embutida e exige conexão para confirmar |
| API de pagamento recusa operação | Exibe erro de negócio; nunca interpreta recusa como sucesso de UI |

Hipóteses operacionais: revalidação da configuração ao abrir tela crítica e janela de frescor online de 60 s para telas comuns. A janela offline máxima proposta é 24 h somente para tela não crítica, com assinatura previamente validada e identificação da conta preservada. Uma revisão vencida não serve para nova operação de pagamento. O limite de rollback de 5 min (RNF3) se refere a usuários online que voltam a buscar a configuração, e depende de medição; não cobre dispositivos offline.

## Rastreabilidade

| Requisito | Responsável principal |
|---|---|
| RF1 | BFF SDUI e cliente SDUI |
| RF2, RF7, RNF1 | Catálogo do app, negociação e fallback |
| RF3 | Controle de rollout e BFF |
| RF4, RNF2 | Validador de ações no app e API de negócio |
| RF5, RF8, RNF3 | Cache, tela embutida e controle de rollout |
| RF6 | Validador no app |
| RNF4, RNF5 | Observabilidade e medição ponta a ponta |

## Referências técnicas

- [RFC 8785, JSON Canonicalization Scheme](https://www.rfc-editor.org/rfc/rfc8785.html), base para assinatura determinística do JSON.
- [OWASP MASVS-AUTH](https://mas.owasp.org/MASVS/07-MASVS-AUTH/), referência para autenticação e autorização no aplicativo móvel.
