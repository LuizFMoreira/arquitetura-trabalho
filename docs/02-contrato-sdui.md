# 02 — Contrato SDUI: telas, componentes, ações e versões

Atende ao item obrigatório 1 e a RF1, RF2, RF4, RF6, RF7, RNF1 e RNF2. Cenário: jornada de contratação de serviço (`01`, §6).

## 1. Princípios do contrato
1. **Catálogo fechado.** O payload só *compõe* componentes e ações que o binário já possui. Não carrega código, script nem expressões.
2. **Declarativo e sem lógica.** Sem condicionais, laços ou fórmulas no payload. A decisão de *qual* tela enviar é do servidor (RF3).
3. **Tolerante a desconhecido.** O cliente ignora campos que não conhece e nunca falha por causa deles (RNF1).
4. **Seguro por padrão.** O que o cliente não reconhece é descartado ou substituído por fallback, nunca executado.
5. **Negociado.** O cliente declara o que suporta; o servidor entrega só o que o cliente aguenta (H1).

## 2. Negociação (requisição do cliente)
O cliente envia em toda busca de tela:

| Campo | Exemplo | Uso |
|---|---|---|
| `platform` | `android`, `ios` | Segmentação (RF3) |
| `appVersion` | `5.12.0` | Segmentação e rollout |
| `osVersion` | `14` | Capacidades do dispositivo |
| `schemaVersions` | `1.0–1.2` | Versões do envelope que o app entende |
| `catalog` | `{"Text":1,"Button":1,"TextField":2,"DatePicker":1}` | Versão máxima de cada componente conhecido |
| `actions` | `{"navigate":1,"submit_form":1,"open_url":1}` | Versão máxima de cada ação conhecida |
| `capabilities` | `["camera.capture.v1","biometrics.v1"]` | Recursos nativos disponíveis (RF7) |
| `profile` | `"cliente-pf"` | Segmentação por perfil (RF3) |

Com isso, o servidor não envia `PhotoUpload` a quem não o conhece. O fallback do cliente (§6) é a segunda barreira.

## 3. Envelope (a tela e seus metadados)

| Campo | Obrigatório | Descrição |
|---|---|---|
| `schemaVersion` | sim | `MAJOR.MINOR` do contrato |
| `screenId` | sim | Identificador estável da tela (`contratacao.dados`) |
| `revision` | sim | Identificador único da configuração publicada; permite rollback, auditoria e correlação em métricas |
| `critical` | sim | `true` para telas críticas (ex.: pagamento): política mais restrita (D3) |
| `issuedAt` / `expiresAt` | sim | Validade; o cliente rejeita payload expirado e usa o cache conforme RF8 |
| `requires.capabilities` | não | Capacidades exigidas pela tela inteira |
| `signature` | sim | Assinatura (`alg`, `kid`, `value`) do JSON canônico com o campo `signature` removido; cobre envelope, `screen` e `fallback` |
| `fallback.screenId` | sim para telas críticas | Tela de fallback embutida no app a usar se nada funcionar |

Exemplo completo: [`exemplos/tela-contratacao-valida.json`](../exemplos/tela-contratacao-valida.json).

## 4. Componente

```
{ "id": "...", "type": "TextField", "v": 2,
  "optional": false,
  "requires": { "capabilities": [] },
  "props": { ... },
  "children": [ ... ],
  "fallback": { <componente conhecido> } }
```

| Campo | Regra |
|---|---|
| `id` | Único na tela; usado em métricas e erros |
| `type` + `v` | Identificam o componente e sua versão no catálogo |
| `props` | Validadas contra o schema do componente; props desconhecidas são ignoradas |
| `optional` | Se `true` e o componente não puder ser renderizado, é omitido sem afetar a tela |
| `requires.capabilities` | Capacidades necessárias a este componente |
| `fallback` | Componente alternativo, de tipo já conhecido, usado se este não puder ser renderizado |

### Catálogo (versão inicial, ilustrativo)

| Componente | Versões | Função | Ação permitida |
|---|---|---|---|
| `Column`, `Row` | 1 | Layout | — |
| `Text` | 1 | Texto e estilos do design system | — |
| `Image` | 1 | Imagem de URL em domínio permitido | — |
| `TextField` | 1, 2 | Campo de texto (v2 adiciona máscara) | — |
| `DatePicker` | 1 | Escolha de data | — |
| `Button` | 1 | Dispara uma ação | `onTap` |
| `PhotoUpload` | 1 | Captura de foto (exige `camera.capture.v2`) | `onChange` |
| `PaymentSummary` | 1 | Resumo e confirmação de pagamento (crítico) | `submit_form` |

O catálogo é **definido e versionado no repositório do app**; cada release declara o que suporta (`catalog` da §2). Os componentes são limitados em *quantidade e tipo*, e novos tipos exigem release do app.

Limites propostos para a árvore, ainda não medidos (H5): 12 níveis, 200 nós e 256 KiB por tela. Publicador e app aplicam os mesmos limites.

## 5. Ações
Ações são **tipadas**. O payload só referencia uma ação do catálogo e passa parâmetros validados; nunca descreve *como* executar.

| Ação | Parâmetros | Restrições |
|---|---|---|
| `navigate` | `screenId` | Só para telas registradas no app |
| `submit_form` | `formId`, `fields[]` | `formId` registrado; o backend valida e autoriza (RNF2) |
| `open_url` | `url` | `https` apenas; domínio na allowlist do app |
| `show_dialog` | `dialogId` | Diálogos de um conjunto fixo |
| `dismiss` | — | Fecha a tela atual |
| `track_event` | `name`, `props` | Nomes de evento em allowlist; sem dados sensíveis |

Regras de segurança (camada de cliente, RF4 e RNF2):
1. Ação com `type` fora da allowlist → **rejeitada**, tela substituída por fallback e evento de segurança registrado.
2. Parâmetros fora do schema da ação → rejeitados.
3. A ação **não carrega autoridade**. Uma operação sensível (ex.: confirmar pagamento) é autorizada pelo backend com credencial do usuário e verificações próprias, ainda que o payload esteja legítimo.
4. Nenhum tipo de ação executa código, avalia expressão ou carrega módulo.

Exemplo de rejeição: [`exemplos/payload-invalido-acao-nao-autorizada.json`](../exemplos/payload-invalido-acao-nao-autorizada.json).

## 6. Componente novo em app antigo (resposta à pergunta orientadora)
Ordem de resolução no cliente para cada componente:

1. **Conhece `type` e `v`?** Renderiza.
2. **Conhece `type`, mas `v` é maior?** Se o componente tem `fallback`, usa-o; senão, tenta a maior versão conhecida *somente se* o catálogo marcar a versão como retrocompatível.
3. **Não conhece `type`, ou falta capacidade** (`requires`)?
   - Tem `fallback` → renderiza o fallback.
   - `optional: true` → omite.
   - Nenhum dos dois → o componente é **essencial e não renderizável**: a **tela inteira** vai para o fallback (`fallback.screenId`, ou a última definição válida em cache, RF8).
4. Em todos os casos, registra o evento (`unknown_component`, `fallback_used`) para métrica (RNF4).

O servidor reduz a ocorrência disso pela negociação (§2), mas **nunca depende só dela**. Exemplo: [`exemplos/componente-novo-app-antigo.json`](../exemplos/componente-novo-app-antigo.json).

## 7. Versionamento e evolução (RNF1)

| Mudança | Classificação | Regra |
|---|---|---|
| Adicionar prop opcional, componente ou ação | Compatível (**MINOR**) | Permitida; apps antigos ignoram |
| Adicionar valor de enum | Compatível com ressalva | Cliente trata valor desconhecido com default seguro |
| Tornar prop opcional em obrigatória | **Incompatível** | Proibida na mesma `v`; criar nova versão do componente |
| Mudar o significado de prop existente | **Incompatível** | Proibida; criar nova `v` ou novo `type` |
| Remover prop, componente ou ação | **Incompatível** | Só após descontinuação e depois que a versão do app deixa de ser suportada |
| Mudar estrutura do envelope | **MAJOR** | Exige negociação; servidor serve a major antiga enquanto houver apps nela |

Regras de governança:
- Toda mudança de contrato passa por **testes de contrato** contra as versões de app ainda suportadas.
- Política de suporte: o time define a janela de versões suportadas e a data de descontinuação (hipótese: acompanhar a cauda de instalações, H3).
- O servidor mantém serializadores por `schemaVersion` suportada.

## 8. Validação no cliente (RF6)
Antes de renderizar, nesta ordem:
1. Assinatura válida e `kid` conhecido. O BFF assina a resposta final após segmentação, usando canonização RFC 8785; a chave pública correspondente já está no app.
2. `expiresAt` não vencido.
3. `schemaVersion` suportada.
4. Estrutura válida contra o schema do envelope e dos componentes.
5. Limites: profundidade, número de nós e tamanho total do payload (12 níveis, 200 nós e 256 KiB propostos como hipótese).
6. Ações na allowlist e com parâmetros válidos.
7. Capacidades exigidas disponíveis.

Qualquer falha em 1–3 → descarta o payload e usa o cache (RF8) ou o fallback. Falhas em 4–7 podem ser resolvidas por componente (§6) ou levar à tela de fallback.

O cache só é elegível se a definição foi previamente validada, pertence à mesma tela e contexto, ainda não passou de `expiresAt` e não está revogada. Para tela de pagamento, a verificação de revogação exige rede; se falhar, usa-se a tela embutida. Os exemplos JSON têm assinatura fictícia e datas ilustrativas, portanto demonstram a forma do contrato, não uma resposta verificável em produção.

## 9. O que o contrato propositalmente **não** permite
Para responder "qual o limite entre flexibilidade e complexidade":

- Lógica condicional, laços, variáveis ou expressões no payload.
- Novos tipos de componente ou de ação sem release do app.
- Scripts, WebAssembly, módulos nativos ou URLs de código.
- Mudar permissões, credenciais ou regras de negócio via payload.

O servidor controla **composição, conteúdo, ordem e regras de exibição por segmento**. O que exige capacidade nativa nova exige atualização do app. A decisão formal fica em `06-decisoes`.

## 10. Pendências
- [ ] Diagrama de componentes mostrando quem produz, valida e consome o contrato (`03`).
- [ ] Confirmar com a equipe o catálogo inicial e os limites de payload (hipóteses).
- [ ] Revisão do `arquiteto-senior`.
