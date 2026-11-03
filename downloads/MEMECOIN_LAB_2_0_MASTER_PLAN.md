# MEMECOIN LAB 2.0 — MASTER PLAN

**Especificação de produto, arquitetura, pesquisa quantitativa, agentes autônomos, integração social e novo dashboard**  
**Data de referência:** 03/11/2026  
**Destino:** Claude Code, trabalhando sobre o projeto existente em `C:\Users\gebt\Downloads\memecoins`  
**Modo autorizado:** dados reais + operações **100% fictícias** (paper trading).  
**Status de lucratividade:** **NÃO COMPROVADA**. Este documento é uma especificação a ser implementada/testada, não uma estratégia comprovadamente rentável.

> **MISSÃO:** sair de um laboratório complexo que vê moedas dispararem, mas perde dinheiro em operações simuladas, e construir um sistema de descoberta, análise multiagente, seleção, votação, execução fictícia e gestão de saídas. A métrica central é **lucro líquido da carteira**, depois de custos, e não número de tokens descobertos, agentes, modelos ou gráficos.
>
> **Importante:** nenhum projeto sério consegue prometer “comprar todas as moedas certas” nem garantir lucro. Um token 1000% acima de certo preço pode ter sido impossível de comprar/vender ao preço anunciado. Queremos reduzir perdas evitáveis, encontrar oportunidades elegíveis mais cedo, demonstrar ou refutar uma vantagem econômica e registrar cada chance perdida sem retrospectiva enganosa.

---

## 0. INSTRUÇÃO IMEDIATA AO CLAUDE CODE

Você **não está construindo um projeto novo** nem deve destruir o Memecoin Lab atual. Deve **inspecionar o repositório e os bancos primeiro**, confirmar o que já funciona e executar a migração incremental descrita neste arquivo. Não declare nada “concluído” com base apenas no texto de planejamento. Cada requisito entregue precisa ter código, teste e evidência observável.

**Prioridade absoluta, nesta ordem:**

1. Corrigir o caminho crítico do trading fictício e a contabilidade de lucro/prejuízo; jamais inventar fills ou cotações.
2. Separar *detectar uma moeda* de *aprovar sua compra*, e monitorar 100% das oportunidades detectadas mesmo quando rejeitadas.
3. Adicionar análise on-chain, social, de notícias e riscos **com decisões reais e auditáveis**.
4. Criar consenso dos agentes: a carteira `CONSENSUS_2_0` só abre posição se **todos os verificadores obrigatórios elegíveis aprovarem** e nenhum bloquear. `ABSTAIN`, erro e dado crítico ausente significam **não entrar**, mas a oportunidade continua acompanhada.
5. Ter um motor de pesquisa/replay *contrafactual* independente que mede o que teria acontecido nos rejeitados (sem fingir que eram operações da carteira).
6. Testar estratégias por **resultado líquido, frequência de oportunidades, recall de moonshots, risco, execução e custos**, em novos períodos.
7. Construir um terminal belíssimo, rápido, com **personagens pixel art realmente conectados aos eventos** e uma página mostrando por que cada agente aprovou/rejeitou.
8. Entregar serviço 24/7 enquanto a máquina estiver ligada e com rede, reinício controlado, custos de API limitados e modo degradado explícito.

**NUNCA:** conectar wallet real, usar chave privada, transmitir swap on-chain, assinar transação, inserir dados futuros na tomada de decisão, apagar perdas antigas, reiniciar carteira perdedora fingindo lucro, mascarar falhas de feeds, criar animação que simula análise inexistente ou afirmar retorno garantido.

---

## 1. DIAGNÓSTICO: O QUE ESTÁ ERRADO E O QUE MUDA

### 1.1. Situação existente a preservar

Conforme histórico fornecido pelo usuário (não auditado diretamente neste documento): Python 3.11 + `uv`, SQLite, Starlette, React/TypeScript/Vite/Tailwind, Lightweight Charts; descoberta PumpPortal/Pump.fun/PumpSwap, Solana RPC, DexScreener e GeckoTerminal; serviços `collector`, `engine`, `trainer`, `enricher`, `shadow`, `lab`, `highfreq`, `moonshot`, `early`, `paper`, `dashboard`; dados em `data/memelab.db`, `data/pump_lab.db`, `data/historical_research.db`, `data/lab/`.

Resultados conhecidos **históricos**: experiência 0002 de US$10 para ~US$8,72; carteiras anteriores próximas de US$6,64 e US$6,62; executor `paper` ligado a `e_signals` em 03/11; S1 fechou ~US$8,09, e S2 foi vista com equity aproximado de US$9,34; uma vitória OJ ~US$0,72, mas perdas superiores. HighFreq observou gaps; Early encontrou baixa taxa de movimentos 5x e sem edge econômico comprovado; ML e saídas parciais não superaram consistentemente controles.

**Não reescrever os resultados históricos.** Para cada afirmação sobre estado atual, buscar evidência em código, logs e bancos antes de assumir que o sistema permanece assim.

### 1.2. Falhas arquiteturais principais

| Problema | Consequência | Correção 2.0 |
|---|---|---|
| `Early` emitia sinais sem executor consumidor | Oportunidades sem operações | Pipeline event-driven com contrato de decisões e fills |
| Seleção baseada em variáveis frágeis | Compras de moedas que colapsam | Vetos de segurança, análise de fluxo/risco/social e validação temporal |
| Critério de lucro = acertar altíssimas | Picos teóricos sem venda executável | Simulação de ida e volta, preço impactado, custos e exit liquidity |
| Modelos em caminho crítico / demorados | Sinal fica obsoleto | Hot path determinístico rápido; modelos já carregados; LLM em paralelo |
| Pouco ou nenhum contexto narrativo/social | Falta de sinais de atenção emergente | X API, notícias, narrativas, reputação e deduplicação de campanhas |
| Conservadorismo que impede paper trades | Zero aprendizado operacional | Carteira consensual + replay contrafactual e experiments separadas |
| Overfitting em dezenas de variantes | Resultados históricos ilusórios | Registro antecipado, teste cronológico e holdout intocado |
| Exibição de agents sem dados verificáveis | “teatro de IA” | Evento real + decisão estruturada + trace + painel pixel art |
| Windows em suspensão / worker travado | Lacunas silenciosas | Supervisor, health checks, reconnect e registro explícito de gaps |

### 1.3. O que a pesquisa externa realmente sugere

- Um estudo de setembro de 2026 sobre **15 milhões de lançamentos Pump.fun** identificou wash trading, ocultação da carteira criadora, vendas coordenadas, cópias de moedas e manipulação social. Portanto, **volume, compradores e buzz podem ser fabricados**; é obrigatório estudar qualidade e independência dos sinais. [Pesquisa: *Meme Coin Factories*](https://arxiv.org/abs/2609.10246).
- Trabalho apresentado na **USENIX Security 2026** mostrou manipulações frequentes entre memecoins de alta rentabilidade observada; não confundir alta no gráfico com demanda genuína. [Pesquisa](https://www.usenix.org/conference/usenixsecurity26/presentation/mongardini).
- Estudo de 2026 sobre detecção precoce de rug pulls na Solana usa os **primeiros cinco minutos** de negociação e destaca a necessidade de avaliar generalização entre plataformas. [*Catching the Rug*](https://arxiv.org/abs/2608.20271).
- Um artigo revisado sobre Twitter e pump-and-dump encontrou que promoções atraem interesse e que seguidores de campanhas podem vender tarde, acumulando perdas. **Menções são hipótese a avaliar, não sinal de compra automático.** [Artigo](https://doi.org/10.1016/j.irfa.2024.103479).
- Pesquisa de overfitting financeiro demonstra que o melhor backtest entre centenas de variantes costuma decepcionar fora da amostra. [Bailey et al.](https://papers.ssrn.com/sol3/Papers.cfm?abstract_id=2326253).

**Conclusão operacional:** nossa hipótese nova é a **confluência temporal** de atenção orgânica nascente, fluxo de compradores independentes, segurança verificável e preço ainda executável — não “comprar a moeda com mais likes” e não “comprar toda moeda nova”. Essa hipótese pode falhar e precisa ser comprovada.

---

## 2. NOVA ARQUITETURA: FAST PATH + DEEP INTELLIGENCE

```mermaid
flowchart LR
    A[PumpPortal + Solana] --> B[Event Bus persistente]
    C[DexScreener + GeckoTerminal] --> B
    D[X API + social licenciado] --> B
    E[CryptoPanic + RSS + notícias] --> B
    B --> F[Opportunity Registry / Watch Forever]
    F --> G[Fast Path: Flow + Chain + Vet]
    F --> H[Parallel Intelligence: Social + Narrative + News]
    G --> I[Council: votos independentes]
    H --> I
    I --> J[Risk / Chief: gates + decisão]
    J -->|todos APPROVE| K[Paper Execution Simulator]
    J -->|REJECT / ABSTAIN| L[Counterfactual Replay]
    K --> M[Portfolio + Realized PnL]
    L --> N[Research + Missed Moonshots]
    M --> O[Dashboard WebSocket]
    N --> O
    I --> O
```

### 2.1. Doze componentes lógicos, **não necessariamente doze processos**

A ideia de multiagentes é distribuir responsabilidades e contratos, **não lançar doze LLMs pesados ao mesmo tempo**. Um worker assíncrono pode hospedar vários agentes determinísticos, e apenas tarefas relevantes chamam inferência NLP. Cada módulo precisa ter entrada real, estado e saída assinada por `run_id/model_version`.

| Agente | Papel e informação independente | Saída obrigatória | Pode vetar compra? |
|---|---|---|---|
| **SCOUT** 🔭 | Descobertas novas, pools e migrações; rastreia todos os mints | `NEW`, `MONITOR`, `CANDIDATE` | Não; fornece candidatos |
| **FLOW** 🌊 | Compras/vendas, fluxo líquido SOL, compradores independentes, impulso, aceleração | `APPROVE/REJECT/ABSTAIN` + métricas | Sim |
| **ONCHAIN** ⛓️ | Reservas, stage bonding/pool, preço, holder/creator concentration, cluster de carteiras | Voto + evidências | Sim |
| **VET** 🛡️ | Autoridades, extensões SPL, token imitador, creator history, wash/sybil, saída possível | Veto de segurança | **Sim, veto duro** |
| **SOCIAL** 🐦 | X/Twitter, links oficiais, narrativa, autores únicos, velocidade de menções, bots | Voto + posts e intervalo | Sim |
| **NEWS** 📰 | Notícias/RSS/X News, plausibilidade e relevância da narrativa | Voto / `NEUTRAL_VALIDATED` segundo regra congelada | Sim onde obrigatória |
| **QUANT** 📈 | Projeção probabilística, edge líquido e comparação contra baseline | Voto + intervalo e versão | Sim |
| **RISK** 🧯 | Exposição, saldo, drawdown, stale feeds, execução ida/volta, limites de risco | Veto operacional | **Sim, veto duro** |
| **CHIEF** 🧠 | Reúne votos por `decision_id`, checa independência, prazo e quórum | `BUY_PAPER`, `WATCH`, `REJECT` | Nunca ignora veto |
| **EXECUTOR** ⚡ | Fill fictício, lotes, preço de execução e ordens de saída | Fills/ordens ou erro | Bloqueia se ilíquido |
| **PORTFOLIO** 💼 | Contabilidade, mark-to-market, PnL, reconciliação | Ledger e equity | Não cria oportunidades |
| **RESEARCHER** 🔬 | Replays, falsos negativos, experimentos, labels e retrain offline | Relatórios reproduzíveis | Não opera na carteira principal |

**Regra contra “personagens vazios”:** personagens sem implementação ativa devem ficar identificados `OFFLINE`/`NOT_CONFIGURED`. A aparência não torna um processo um agente inteligente. O CHIEF não “pensa” em voz alta; mostra decisões curtas fundadas em evidências.

### 2.2. Caminho rápido vs. profundo

**Fast path (eventos):** descoberta → identidade mint → validação de preço/reservas → critérios objetivos → encaminhar enriquecimento → alimentar votações. Sem carregar modelo a cada token; inferência curta e process pool warm. O sistema precisa conseguir observar todas as moedas do feed **dentro do alcance real do provedor**, mas só aprofundar nas elegíveis.

**Deep path (assíncrono):** leitura de posts/notícias, investigação de criadores, análise semântica, comparação de narrativas, features de ML, melhor estimativa de capacidade de saída e monitoramento após migração. Não bloquear o WebSocket porque o X não respondeu.

**Plano de decisões:** checkpoints configuráveis `T+5s`, `T+15s`, `T+30s`, `T+60s`, `T+3m`, `T+5m`, `T+15m`, além de eventos (surto social, migração, liquidez perdida). *Estes tempos são hipóteses de projeto, não latências prometidas*. Para entrar na carteira consensual é obrigatório que **todos os votos requeridos da estratégia estejam presentes e positivos** no momento da decisão. Uma moeda não aprovada pode ser reavaliada em outro checkpoint se as evidências surgirem; nunca abrir uma compra fictícia retroativamente no timestamp de um sinal anterior.

### 2.3. Contrato de observabilidade de todos os eventos

```json
{
  "event_id": "uuid",
  "schema_version": "2.0",
  "mint": "solana_mint_address",
  "event_type": "SOCIAL_SIGNAL|AGENT_VOTE|PAPER_FILL|...",
  "event_time_utc": "timestamp da origem (se conhecido)",
  "observed_at_utc": "timestamp em que nosso sistema realmente soube",
  "processed_at_utc": "timestamp local",
  "source": "pumpportal|solana_rpc|x_api|...",
  "source_event_id": "id da transação/post/registro",
  "data_age_ms": 0,
  "quality": "VERIFIED|ESTIMATED|STALE|MISSING",
  "trace_id": "uuid",
  "idempotency_key": "fonte:id:versao",
  "payload": {}
}
```

Não aceitar `event_time` futuro para decisões nem reordenar artificialmente a ordem em que dados foram recebidos. Guardar raw event quando permitido, hash da versão normalizada, origem, taxa de atraso, redelivery e períodos offline. Realizar rededup por `(source, source_event_id)`.

**Armazenamento:** manter SQLite/WAL para legado e baixo volume, porém introduzir **fila persistida/durável** (ex.: tabelas append-only com cursor/acks, ou SQLite dedicado para event bus) com idempotência, não somente queue na RAM. Considerar PostgreSQL apenas após benchmark de contenção; não migrar tudo sem necessidade.

---

## 3. FONTES: VERIFICAÇÃO E LIMITAÇÕES REAIS EM NOVEMBRO/2026

| Fonte | Utilização correta | Atenção/limite | Prioridade |
|---|---|---|---|
| [PumpPortal data API](https://pumpportal.fun/data-api/bonk-fun-data-api/) | `subscribeNewToken`, `subscribeMigration` e, se habilitado, trade stream para tokens selecionados | Desde **01/05/2026** trade stream exige chave e é tarifado **0,01 SOL/10.000 trades**; ligação associada com wallet de cobrança, diferente de carteira de trading; **não habilitar cobrança sem autorização** | Alta |
| [PumpPortal fees](https://pumpportal.fun/fees/) | Auditoria das taxas do provedor, negociação e rede | Preferir apenas data subscription; nada de endpoints de transação real | Alta |
| [Pump.fun public docs](https://github.com/pump-fun/pump-public-docs) | Especificação atual de curva, PumpSwap, taxas, migrações, Token-2022 | Protocolos evoluem; fixar versão verificada em código e testes | Alta |
| [Solana RPC WS](https://solana.com/docs/rpc/websocket) | Logs, accounts e acompanhamento de transações/estado | `logsSubscribe` tem filtros e `blockSubscribe` pode exigir servidor habilitado; medir limites | Alta |
| [DexScreener API](https://docs.dexscreener.com/api/reference) | Pares, preço, liquidez, boosts como contexto de promoção | Pair/token batching até 30 endereços; endpoints com limites de 60 ou 300 rpm; boosts **não são demanda orgânica** | Alta |
| [GeckoTerminal](https://api.geckoterminal.com/docs/index.html) | Novos pools, OHLCV real pós-pool e corroborar preços | API pública gratuita com **30 chamadas/min** ([fonte, out/2026](https://support.coingecko.com/articles/6867086195-does-geckoterminal-have-an-api)); trending frequentemente atualiza em ~60s | Alta |
| [Jupiter Swap API v2](https://developers.jup.ag/docs/swap) | **Somente quotes**, para avaliar possibilidade e custo de compra/venda | `/order` sem `taker` faz quote; não usar `/execute`, `/submit` ou chave de transação; não cobre necessariamente curvas recém-lançadas | Média |
| [Helius](https://www.helius.dev/blog/laserstream-websockets) | Opção de WSS avançado/gRPC com melhor resolução após necessidade comprovada | Benefícios e acesso variam por plano, **não pressupor gratuito** | Opcional |
| [X API](https://docs.x.com/x-api/getting-started/pricing) | Posts recentes, listas, filtros/stream, menções a mints e narrativas | **US$0,005 por post lido** no preço consultado; orçamento/credenciais necessários | Alta em modo opt-in |
| [LunarCrush](https://lunarcrush.com/en/developers/api) | Alternative social analytics: tópicos, criadores, posts, sentimento | Requer chave, cobertura/planos variam; comparar valor incremental, não duplicar custos | Opcional |
| [CryptoPanic API](https://cryptopanic.com/developers/api/) | Feed de notícias cripto autenticado | Plano e acesso por API; checar termos e cobertura do tema | Média |
| RSS oficiais licenciados | Manchetes de cripto, Solana e plataformas; tópicos de memes | RSS não é feed on-chain; notícias frequentemente chegam depois | Média |

**Nunca inventar uma integração.** Implementar adaptação `SourceAdapter` com `enabled`, `credential_status`, `budget_remaining`, `health`, `coverage`, `last_event`, `last_error`, `lag`, `unit_cost`. Na interface separar **CONNECTED**, **DISABLED_NO_KEY**, **BUDGET_EXHAUSTED**, **RATE_LIMITED**, **DEGRADED**, **OFFLINE**.

### 3.1. X/Twitter: integração exigida (mas não gratuita por padrão)

**Objetivo:** detectar *narrativas e memecoins surgindo em conversas reais* antes de alguns movimentos de preço, avaliar se há pessoas diferentes falando do mesmo contrato, crescimento de autores únicos, concentração de contas promocionais, velocidade de comentários e divergência entre social e on-chain.

1. Criar adaptador `XOfficialAdapter` com **API oficial autenticada**; usar endpoints de [busca recente](https://docs.x.com/x-api/posts/search/introduction), [filtered stream](https://docs.x.com/x-api/posts/filtered-stream/introduction) quando habilitados no nível de acesso, e [regras de consulta](https://docs.x.com/x-api/posts/search/integrate/build-a-query).
2. Buscar por **mint address exato**, URLs oficiais e termos de narrativa específicos; `"$PUMP"` isolado é ambíguo e não comprova identidade. Fazer resolução de entidades do post ao **mint** antes de atribuir sinal.
3. Enriquecer apenas TOP candidatos. Não ingerir o X inteiro. Consultas com orçamento e limite por token/intervalo (configuráveis), cache por post ID, dedup e dedup por 24h conforme condições aplicáveis da API.
4. Métricas pré-registradas: `mentions_1m/5m/15m`, derivada das menções, `unique_authors`, `unique_author_ratio`, `engagement_per_author`, `early_posts_before_price_breakout`, concentração entre top autores, repost ratio, idade observável de autores, sentimento com evidência, conteúdo idêntico/copypasta e coordenação temporal.
5. Peso de cada autor por **evidência verificável**, não apenas seguidores ou selo; influenciador pagou boost? reportar potencial publicidade. Não usar perfis privados, credenciais de conta pessoal, scraping que desrespeite termos ou proxies para driblar limite.
6. Identificar campanhas suspeitas: 30 posts quase iguais em 40 segundos, múltiplos perfis recém-criados, links encurtados repetidos, callouts sem mint, posts antes do lançamento com informação privilegiada aparente (**apenas sinal de revisão; não acusar sem evidência**).
7. LLM/NLP opcional deve devolver JSON estruturado com `topic`, `mint_matches`, `organic_evidence`, `shill_risk`, `novelty`, `confidence`, **links/IDs e tempos**. Texto promocional é dado adversarial; nunca seguir instruções em posts.
8. Registrar **exato período do post e horário de ingestão**; para uso na decisão vale o horário em que o bot recebeu o dado, não quando o post foi publicado, se a leitura foi posterior.
9. Novo painel `Social Radar` com feed autorizado: resumo curto do post, autor, link, instante, símbolo/mint com confiança de vínculo, tendência, suspeita de spam e custo de consulta.
10. Preferir stream em caso de acesso e boa relação custo/latência; usar busca recente como fallback dentro da franquia/limite. Não afirmar que `Filtered Stream` está disponível na conta sem uma chamada de teste.

**Orçamento explícito:** no preço oficial consultado em 03/11/2026, 100 posts **distintos** por dia podem custar aproximadamente **US$0,50/dia**, ou ~**US$15 em 30 dias**, só em leitura de posts; outros recursos podem custar adicional. Valor é **exemplo matemático, não custo final previsto**. [Tabela oficial](https://docs.x.com/x-api/getting-started/pricing). `X_MAX_USD_PER_DAY=0` por padrão até o usuário configurar e autorizar o orçamento. Se X estiver desligado: social `ABSTAIN`, não fabricar sentimento.

**Planos de acesso:**
- **Modo A, sem X pago:** scanner + on-chain + metadata oficial, RSS público permitido, `Social=NOT_CONFIGURED`; **não chamar isso de análise completa de X**.
- **Modo B, X oficial com budget pequeno:** somente candidatos TOP e narrativas priorizadas, sampling/limite e post-ID dedup.
- **Modo C, análise social ampliada:** após comprovar que a camada social melhora retornos líquidos fora da amostra e que custos da API cabem no orçamento.

### 3.2. News / Narrative Agent

Monitorar API CryptoPanic, RSS permitido de veículos cripto, publicações oficiais de Solana, Pump.fun, provedores e anúncios relevantes; registrar URL, título, publicação, **observed_at**, autor, fonte e categoria. `NEWS` deve separar:

- Notícia de evento verificável (anúncio oficial, produto, protocolo).
- Hype sem confirmação (influenciador, rumor, meme da semana).
- Spam/notícia replicada por republicação.
- Notícia de macro/mercado geral (pode afetar regime, não o token diretamente).

**Nada de manchete genérica virar ordem.** Vincular ao **token certo**, não ao símbolo genérico. Incluir classificação `POSITIVE / NEGATIVE / NEUTRAL / UNKNOWN`, intensidade, confiabilidade e a idade do evento. Na estratégia “narrativa” notícias podem ser obrigatórias; na estratégia “early organic” notícias diretas podem ser dispensadas mediante regra **previamente congelada**, mas o agente continua executando e emitindo voto `NEUTRAL_VALIDATED`. Não marcar `APPROVE` sem ter analisado a fonte prevista.

### 3.3. Segurança de tokens e criadores

Validar com dados oficiais:

- Autoridade de **mint** e **freeze** quando presentes; extensões Token-2022 (como *transfer hook* que altera regras das transferências). [Docs Solana token](https://solana.com/docs/tokens) e [Transfer Hook](https://solana.com/docs/tokens/extensions/transfer-hook).
- Distinção Pump curve/AMM (tokens 2022, versões, migração, fees e flags conforme programa; não pressupor que todos os lançamentos seguem uma única versão antiga do protocolo).
- Drenagem de reservas, liquidez concentrada, grandes vendas dos top holders, transferências para carteiras relacionadas, histórico temporal do criador e clusters suspeitos (heurísticas probabilísticas, com erro conhecido).
- Rotas de compra **e de venda**; se não consegue determinar que pode vender, **vetar a compra na carteira consensual**.
- Token *copycat* (nome, ticker, imagem) não prova relação com original. Identidade = mint e links verificáveis.
- Diferenciar `UNKNOWN` de `SAFE`; ausência de alerta de um scanner não garante ausência de rug.

**Regra absoluta:** nenhuma pontuação social ou previsão ML pode superar um veto estrutural de segurança/exequibilidade.

---

## 4. O CONSELHO: COMO OS AGENTES REALMENTE DECIDEM

### 4.1. Contrato dos votos

Cada decisão de um agente deve ser persistida com o **mesmo `decision_id`** e um snapshot de evidências disponível no momento. Sem `decision_id` compartilhado, não há votação real.

```json
{
  "decision_id": "uuid",
  "opportunity_id": "uuid",
  "mint": "endereco_mint",
  "strategy_id": "NARRATIVE_FLOW_V1",
  "checkpoint": "T+30s",
  "agent_id": "SOCIAL",
  "agent_version": "social-v1.0.0",
  "vote": "APPROVE",
  "confidence_calibrated": 0.63,
  "observed_at_utc": "2026-11-03T18:00:30Z",
  "data_cutoff_utc": "2026-11-03T18:00:28Z",
  "evidence_refs": ["x:post_id", "events:hash"],
  "features": {"unique_authors_5m": 9, "spam_ratio": 0.11},
  "reasons": ["Autores independentes em crescimento", "Contrato verificado nos posts"],
  "warnings": ["Amostra social pequena"],
  "runtime_ms": 81,
  "state": "COMPLETED"
}
```

`confidence_calibrated` só pode existir se houver calibração empírica; caso contrário, usar `null` + `heuristic_score` e deixar isso claro. Não inventar percentuais do tipo “94% de certeza de subir”. O texto é um resumo dos dados, **não raciocínio interno fabricado**.

### 4.2. Votação em paralelo e sem veto escondido

Para cada estratégia, `RequiredCouncil` é uma lista versionada de agentes obrigatórios. **COMPRA PAPER apenas se `all(vote == APPROVE)` e `RISK == APPROVE` e `VET == APPROVE` e `EXECUTION_PRECHECK == APPROVE`**; `CHIEF` apenas agrega e audita. Cada voto deve ter timestamp dentro do TTL e evidências distintas; agentes que analisam a mesma fonte não devem ser contabilizados como votos independentes por aparência.

`REJECT`: evidência negativa. `ABSTAIN`: dados insuficientes. `TIMEOUT`: não respondeu no prazo. `STALE`: evidência fora do TTL. Em todos os casos, **sem compra consensual**; registrar motivos e agendar reavaliação baseada em novos eventos.

**Importante para não zerar oportunidades:** a regra “todos devem dizer sim” é a política *dessa carteira*, não uma verdade científica. Se isso produzir 0 operações ou pior retorno, mostrar o resultado. Não burlar aprovação para forçar compras. Separadamente, criar replays comparando conselho unânime, voto por maioria e apenas regras, todos **simulados e historicamente separados**, para medir o preço de cada veto.

### 4.3. Fluxo de estados por oportunidade

```text
DISCOVERED
  -> TRACKING
  -> ELIGIBILITY_CHECK
  -> COUNCIL_RUNNING
     -> BUY_APPROVED -> PAPER_ORDER_PENDING -> OPEN -> MANAGING -> CLOSED
     -> REJECTED -> WATCHING_REJECTED -> REASSESSMENT / MISSED_MOONSHOT_ANALYSIS
     -> ABSTAINED -> WAITING_FOR_EVIDENCE -> REASSESSMENT / EXPIRED
     -> SYSTEM_ERROR -> RETRY_WITH_BACKOFF / MARK_COVERAGE_GAP
```

O banco precisa guardar **todas** as transições, inclusive zero operações, decisões anteriores e rejeições. O watcher não para se a carteira estiver sem saldo, em drawdown ou com circuit breaker acionado.

### 4.4. Exemplo de diálogo visível, exclusivamente orientado por eventos reais

```text
[13:00:04] 🔭 SCOUT  | Descobri token 9x...pump; evento PumpPortal confirmado.
[13:00:05] 🌊 FLOW   | Acompanhando compras e vendas; ainda sem amostra suficiente.
[13:00:06] ⛓ ONCHAIN | Curva e reservas verificadas em slot 123...; dados frescos.
[13:00:14] 🐦 SOCIAL | 7 autores distintos, 2 posts repetidos; aguardando vínculo com mint.
[13:00:16] 🛡 VET    | Alerta: clusters possivelmente relacionados; rejeito nesta versão.
[13:00:17] 🧠 CHIEF  | REJECTED: veto VET. Token continua na watchlist e no replay.
```

**Estes são exemplos de formato, NÃO fatos de mercado.** A aplicação só pode mostrar linhas desse tipo se os respectivos eventos existirem. Os agentes podem “digitar” no terminal em animações breves, mas o texto tem que vir do backend. Nunca colocar diálogos inventados passando por atividade ao vivo.

---

## 5. QUATRO FAMÍLIAS DE ESTRATÉGIAS 2.0 (HIPÓTESES A SEREM TESTADAS)

Não existe uma regra mágica universal para pegar 10x. A pesquisa deve comparar estratégias *regime-specific* e um baseline honesto. Todas as regras são congeladas **antes** da rodada prospectiva.

### Estratégia A — `EARLY_ORGANIC_FLOW_V1`

**Tese:** em certos lançamentos, crescimento orgânico de compradores independentes e reservas saudáveis pode preceder movimentos de alta.

**Elegibilidade:** mint recém-criado; curva com leitura fresca; fluxo de trades se a fonte estiver habilitada; wallet activity com diversidade; token passando `VET` e ida/volta simulável; ausência de anomalia grave; social ativo quando requerido pelo conselho da carteira principal.

**Features:** compradores únicos por janela, delta de compradores independentes, quantidade de SELL, fluxo líquido normalizado pela liquidez, transações por wallet, vendas do criador, participação dos maiores compradores, medidas simples de aceleração, preço obtido pela fórmula correta da curva, `age_seconds` e `lag_seconds`.

**Não comprar** só porque há muitas compras. Investigar se são self-trades, clusters ou compradores repetidos. Provar se features separadas de volume têm valor incremental.

**Horizontes:** 30s, 1m, 3m, 5m. Decisões reavaliadas apenas com dados novos. Se não houver confirmação social no prazo, registrar *miss*, e não falsa compra.

### Estratégia B — `NARRATIVE_FLOW_V1`

**Tese:** um meme/narrativa pode aparecer primeiro em comunidades, cruzar o X e depois alcançar alta atividade no token.

**Elegibilidade:** vínculo de posts com **mint exato** ou prova forte de identidade; aumento de autores únicos ao longo do tempo; diferentes comunidades/fontes, baixo risco de spam; fluxo financeiro on-chain já aumentando sem estar no estágio extremo de preço; aprovação de todos os agentes.

**Features:** `unique_author_acceleration`, diversidade das fontes, novelty do meme, `link_to_mint_confidence`, social-to-volume lead, relação entre post timestamp e momento real de observação, conteúdo repetido/automatizado, engajamento por autor, concentração e comportamento de venda.

**Falso positivo típico:** disparo organizado com 50 contas publicando o mesmo contrato enquanto insiders descarregam. Isso é um alerta, não uma oportunidade segura.

### Estratégia C — `POST_MIGRATION_CONTINUATION_V1`

**Tese:** depois de migrar para pool, a transparência da liquidez, dados de trades e possibilidade de quote podem melhorar a medição de execução; alguns tokens mantêm narrativa e interesse.

**Elegibilidade:** migração confirmada; pool identificada; liquidez/rota venda em quantidade compatível com a ordem; fluxo não dominado por pouquíssimos holders; spread e impacto aceitáveis; consenso completo.

**Features:** liquidez efetiva, desequilíbrio de negociações, retenção de compradores após migração, intensidade de vendas, evolução dos 1m/5m, persistência do social. Não confundir cobertura de candles pós-migração com cotação disponível no instante do lançamento.

### Estratégia D — `SOCIAL_REVIVAL_V1`

**Tese:** moeda já existente pode ganhar atenção por um meme ou evento novo, ainda que o bot não tenha comprado cedo.

**Elegibilidade:** token previamente descoberto ou localizado em nova narrativa, curva/pool íntegra, aumento demonstrável de autores distintos e volume novo, preço ainda executável, sem concentração/drenagem anormal.

**Vantagem do escopo:** permite capturar *parte* de um movimento que começou depois do lançamento, em vez de insistir apenas nos primeiros 5 segundos. Riscos de chase/fomo devem ser medidos.

### Controles obrigatórios

- `RANDOM_SAME_LIQUIDITY`: seleção aleatória de tokens dentro da mesma idade/faixa de liquidez/horário.
- `FLOW_ONLY`: compra com regra de fluxo isolada sem social.
- `SOCIAL_ONLY_REPLAY`: pesquisa de sinal social isolado, nunca produção.
- `BUY_ALL_ELIGIBLE_REPLAY`: compra teórica de todos os elegíveis com os mesmos custos de execução.
- `NO_TRADE`: caixa puro, referência de risco/perda zero em termos nominais.

**O sistema não pode selecionar a estratégia vencedora com dados que usará posteriormente como “prova” de desempenho.** Testar em segmentos novos, regimes diferentes e períodos de lançamento variados.

---

## 6. RISK ENGINE: EVITAR DERROTAS EVITÁVEIS

### 6.1. Hard blocks sem pesos arbitrários

O `RISK` e o `VET` devem vetar quando:

- Cotação está stale ou indisponível para a ordem/saída.
- Não foi possível verificar regras de transferência/token ou há condição de venda não atendida.
- Liquidez efetiva de saída não comporta tamanho da posição dentro do limite de impacto.
- Há feed inconsistente, gap grande, reorg/rollback relevante, slot ausente ou pool não confirmada.
- Risco do criador/holder, mint/freeze authority ou extensão perigosa excede limite pré-configurado **e tecnicamente verificável**.
- Orçamento da carteira, número de posições ou drawdown atingiu limite.
- Token idêntico já está aberto na mesma carteira; nenhuma ordem duplicada.
- A compra só seria possível antes de a informação existir para o bot.

As regras exatas e thresholds são **parâmetros de experimento** a calibrar, não “números certeiros” fornecidos pelo texto. No dashboard, cada `REJECT` deve citar a condição e o dado que a acionou.

### 6.2. Microcapital: US$10 exige atenção especial

Uma carteira fictícia de US$10 comprando US$2 pode pagar custos proporcionais muito altos, e pequenas taxas fixas podem inviabilizar a operação. **Antes** de testar estratégias mirabolantes:

- Calcular o custo de compra **e** venda por venue e estado do protocolo; taxa da curva, maker/LP/protocolo/criador, taxa de provedores, eventual taxa de roteamento, rede, prioridade, slippage e price impact.
- Em dados com apenas snapshots, não assumir que stop -15% vende a -15%. Simular execução na próxima oportunidade observável e sinalizar gaps.
- Identificar ordens minúsculas cuja economia líquida é inviável, mesmo se houver um candle com +50%.
- Mostrar por trade o ponto de equilíbrio em preço (%), já com custos, de maneira transparente.
- Medir sensibilidade a capital por ordem: ex. US$1, US$2, US$5, US$10, sempre em **replays independentes** e respeitando saldo.

**Não definir taxa universal do PumpSwap**: depende de produto, versão, capitalização e programas. Usar as informações da [documentação Pump](https://pump.fun/docs/fees) e contas on-chain, com testes contra transações conhecidas. Modelar também as alterações de maio/2026 de outros provedores.

### 6.3. Três carteiras (não misturar livros)

1. **`LEGACY_*`**: todas as carteiras antigas preservadas, read-only para auditoria; o sistema não “conserta” prejuízo histórico apagando operações.
2. **`CONSENSUS_2_0`**: carteira totalmente fictícia, iniciada em US$10, **única** com execução da estratégia multiagente aprovada na nova sessão. Gestão do risco formalmente congelada.
3. **`REPLAY_COUNTERFACTUAL_2_0`**: universo de oportunidades, inclusive rejeitadas, com custos realistas, execução condicionada à observabilidade e patrimônio **separado/não comercial**; não somar seu PnL ao da carteira principal.

Pode existir carteira de pesquisa para cada estratégia; cada uma com ID, início, regras versionadas, custo e resultado isolado. Não substituir silenciosamente o ledger da S1/S2.

### 6.4. Gestão de posições como política experimental

Testar lado a lado:

- `EXIT_FULL_TIME`: sair após horizonte fixo ou evento risco.
- `EXIT_DYNAMIC_RISK`: trailing com gap e spread realistas, sem stop impossível.
- `EXIT_LADDER`: venda por tranches se preço **executável** atingir condições definidas, incluindo mínimo econômico por tranche.
- `EXIT_REGIME`: decisão reavaliada quando fluxo social/on-chain começa a inverter.

O executor pode **administrar saídas por regras determinísticas de emergência** mesmo quando um agente não responde (proteção da carteira). Isso não autoriza novas compras sem consenso. **Saídas não dependem de unanimidade**, porque esperar um LLM pode aumentar perdas. Medir qualidade de saída com MFE/MAE, lucro devolvido, fill-rate e fees, mas **otimizar patrimônio líquido final**, não taxa de acerto de take-profit.

---

## 7. SIMULADOR DE EXECUÇÃO: PRECISAMOS ACREDITAR NOS FILLs

### 7.1. Níveis de fidelidade

`F0` = observação incompleta → apenas hipótese, **sem fill**.  
`F1` = simulação sobre snapshot posterior, com custo, delay e intervalos conservadores.  
`F2` = trade events + reservas + modelo da curva/protocolo conhecido e validado.  
`F3` = quotes de compra/venda e pool pós-migração, sem execução real.

Cada trade registra `fidelity_level` e `estimated_uncertainty`. **Nunca elevar F1 a F2 apenas porque o gráfico ficou bonito**. `F3` não significa swap executado; quote pode expirar antes da ordem hipotética.

### 7.2. Regras técnicas não negociáveis

- `signal_time <= decision_time < earliest_fill_possible_time`, respeitando a origem real dos dados e a latência medida. Para ordens abertas após evento, usar próxima cotação realmente observada ou lógica de curva atual; nunca candle OHLCV que contém informações posteriores ao timestamp da decisão.
- Executar **na quantidade de tokens** resultante de uma compra de valor fixo de entrada, **não** assumir retorno igual a `preço_futuro/preço_atual` sem impacto.
- Para saída, calcular **valor que o mercado/curva/pool entregaria para a quantidade inteira**, não `quantity * last_trade_price` como caixa realizado.
- Modelar cap de liquidez, mínima negociação, spread, custo de cada tranche, taxas fixas, prioridade e reorg/cancelamento quando relevantes.
- Se não há rota/venda, a posição permanece ilíquida e recebe valuation conservador ou intervalo explícito. Não marcar lucro como realizado.
- Fills têm `order_id`, `fill_id`, `decision_id`, `quote_snapshot_id`, versão da fórmula, fee breakdown e assertiva de idempotência.
- Ledger com lógica de double-entry ou invariantes equivalentes: caixa + posições marcadas = patrimônio; `realized_pnl` calculado por fluxos e lotes; `unrealized_pnl` separado; total return consistente.
- Cotação da entrada pode ser usada provisoriamente para equity inicial se não houver quote posterior — corrigindo o bug histórico de equity=0 — mas deve indicar `MARK_AT_ENTRY_STALE` e ser substituída quando houver dado fresco.
- Reconciliação contínua entre trades, fees, posições, caixa e equity. Falha leva a `TRADING_PAUSED_ACCOUNTING_ERROR` e alerta claro.

### 7.3. Métricas verdadeiras de execução

`fill_rate`, `estimated_round_trip_cost_pct`, `exit_capacity_usd`, `bid_ask_spread`, `observed_quote_lag`, `slippage_distribution`, `mean_execution_delay`, `stale_quote_rejections`, `unrealized_exposure`, `cost_to_gross_profit_ratio`, `unwind_time`, `stuck_positions`, `missing_tick_count`.

**Se a estratégia teórica tem resultado positivo, mas a simulação com custos conservadores fica negativa, não chamar de lucrativa.**

---

## 8. MACHINE LEARNING 2.0: PREVER A DECISÃO ECONÔMICA, NÃO SÓ O X

### 8.1. Troca do target

Não otimizar apenas `token_faz_5x` ou `retorno_5m > 0`. Criar alvos independentes e causais:

1. `P(executable_profit_gt_costs_by_horizon)` — chance de a política de saída gerar resultado líquido positivo para o tamanho da posição.
2. `E[net_pnl_usd|eligible_entry]` — retorno esperado estimado depois das perdas, custos e oportunidades sem liquidez.
3. `P(large_gain_before_large_drawdown)` — risco/recompensa por janelas concorrentes, sem vazamento.
4. `P(exit_unavailable_or_drain)` — risco de uma saída não estar disponível.
5. `P(breakout_5x|features_at_t)` — **apenas auxiliar** para recall de oportunidades de alta.

O classificador de moonshots pode ser útil para priorizar observação sem ser automaticamente um classificador de compras rentáveis.

### 8.2. Features e avaliação

**Grupos de features:** idade/slot, fluxo de compradores/vendedores, mudança de reservas, concentração/cluster, propriedades do token, fase curva/AMM, lag e gaps, custo da rota, novidade/atividade de autores X, divergência entre buzz e volume, regime de mercado, histórico de criadores (somente eventos conhecidos até o tempo de decisão).

**Treinamento:** primeiro baselines simples (logit/árvores/gradient boosting), depois pesquisas de LightGBM/XGBoost/HistGradientBoosting se instalados e justificados; não introduzir redes neurais gigantes por estética. Ajustar pesos de classe cuidadosamente sem mudar base rate real na calibração. Medir **precision@k**, recall de 5x/10x, calibração/Brier, PR-AUC e, acima de tudo, EV e PnL *on-policy* depois dos custos.

**Prevenção de vazamento:** split cronológico, embargo/purga por token e por creator/wallet quando útil, features exatamente disponíveis no timestamp, valores ausentes preservados, imutabilidade de data cutoffs, regimes de lançamento heterogêneos, seleção prévia de família de hipóteses, holdout final intocado. Evitar milhares de testes sem correção para múltiplas tentativas.

**Ciclo de vida:** `candidate -> offline_validated -> shadow -> prospective_verified -> eligible_for_vote -> champion` com critérios pré-definidos, e nunca promover por PR-AUC isolada. Criar registry com artefato/hash, treino, amostra, versão de custo, período, features, calibrador e motivo da promoção/rejeição. Treinar em jobs isolados com limite de CPU/RAM; não bloquear scanner nem reiniciar modelos a cada token.

### 8.3. LLMs não devem operar em milissegundos

LLM pode analisar narrativa, textos e evidências sociais, criar resumo de risco e investigar anomalias. **Ele não deve ficar no caminho de cada tick**, nem receber “todas as moedas lançadas”, nem decidir apenas lendo gráfico. Pré-computar embeddings/resumos opcionais por tópico, cache por contrato/URL, deadlines e fila de prioridade.

Se **nenhum provedor LLM ou modelo local estiver configurado**, mostrar `NO_LLM_PROVIDER`; regras determinísticas continuam operando, mas **não mentir que SOCIAL/NEWS foram analisados semanticamente por IA**. Não presumir que assinatura comum do Claude Code autoriza uso ilimitado do modelo como backend 24/7; consultar termos/custos aplicáveis e configurar provedor autorizado.

### 8.4. Experimentos causais para descobrir *se social vale o custo*

Criar um quadro com grupos de tokens pareados por **idade, liquidez, horário de lançamento, reservas e fase**, comparando:

- on-chain apenas;
- on-chain + filtro de risco;
- on-chain + risco + social;
- on-chain + risco + social + notícias;
- conselho unânime vs regra simples.

Métricas **prospectivas**, intervalos de confiança por bloco/cluster temporal e token/creator, retorno por custo da API, número de oportunidades, missed moonshots e perda evitada. O social só entra como requisito obrigatório de determinada estratégia após medir cobertura/benefício; a carteira consensual pode manter requisito social por preferência de desenho, mas o relatório deve mostrar se esse requisito prejudicou ou ajudou.

---

## 9. MOONSHOT WATCHER: NUNCA MAIS IGNORAR UMA ALTA SEM SABER POR QUÊ

**Objetivo verificável:** toda moeda recebida por uma fonte durante um período com conexão estável deve entrar em um registro durável com identidade e `first_seen`. Registrar desde cedo o que aconteceu com ela, mesmo se não houver dinheiro ou autorização de entrada.

### 9.1. Dois relógios de valorização

- `market_multiple`: múltiplo observável entre preços cotados (referências com qualidade e intervalo), útil para pesquisa.
- `opportunity_multiple`: múltiplo **potencialmente capturável a partir do primeiro instante elegível e com fill possível**, incluindo entrada depois do sinal, impacto e custo. Pode ser muito menor.

### 9.2. Painel de oportunidades perdidas

Para cada moeda que atingiu 3x, 5x, 10x, 20x, 50x, 100x (em qualquer horizonte definido), responder:

1. O bot **detectou**? Em que momento? Antes ou depois da alta?
2. Qual era o primeiro preço/volume/reserva realmente observados?
3. Passou pelo VET? Quais flags o barraram?
4. SOCIAL/NEWS estavam ativos? Havia posts anteriores **e disponíveis ao bot**?
5. Que votos recebeu em cada checkpoint?
6. A moeda **teria sido comprável e vendável** na simulação conservadora?
7. Qual teria sido o máximo favorável executável (MFE), drawdown e resultado de cada política de saída?
8. Qual cenário de compra hipotética satisfaz a simultaneidade temporal? Se nenhum, marcar `UNTRADABLE_EX_POST`, **não** culpar o bot por ter perdido a previsão impossível.
9. Qual agente foi o primeiro a detectar ou rejeitar? Qual veto foi falso positivo e qual evitou rug?
10. Qual mudança poderia ser uma hipótese para o próximo período, sem ajustar dados históricos da carteira ao resultado conhecido?

**KPI novo:** `coverage_of_launches`, `detection_delay_p50/p95/p99`, `watchlist_continuity`, `moonshot_recall_eligible`, `missed_due_to_feed_gap`, `missed_due_to_veto`, `missed_due_to_budget`, `missed_due_to_lag`, `missed_due_to_capacity`, `missed_untradable`. Exibir denominador e população observável. Um bot não consegue afirmar “viu todas as moedas do mundo” se suas fontes têm limites e lacunas.

### 9.3. Replay útil, não máquina de dinheiro fictícia

Toda moeda rejeitada gera uma **trajetória contrafactual** (onde houver ticks/quotes suficientes): o simulador busca o primeiro instante observável elegível, simula uma entrada sob uma política pré-fixada, seguida por saídas. Esses resultados aparecem como `HYPOTHETICAL_REPLAY` e não entram no PnL principal. Registrar a incerteza quando faltarem ticks.

---

## 10. OPERAÇÃO 24/7, DE VERDADE

**Limite físico:** no Windows local, 24/7 só funciona se o computador estiver ligado, não suspenso, com energia, rede e serviços saudáveis. Sem servidor externo, não é operação remota permanente. Monitoramento pode ser contínuo; não exigir que o usuário mantenha o dashboard aberto.

### 10.1. Serviços e confiabilidade

- Supervisor com Windows Task Scheduler/Service configurado conforme permissões, startup automático, políticas de restart, logs rotativos; sem depender de abrir aba web.
- Um único loop PumpPortal por instância onde recomendado; reconectar com exponential backoff e jitter; resubscribe dos mints prioritários; backfill quando disponível e marcar `GAP_UNRECOVERABLE` quando não.
- Health `HEALTHY/DEGRADED/FAILED`, heartbeat por serviço, backlog, idade do último evento, latência, rate limit, erro, consumer lag e contador de eventos perdidos.
- Circuit breakers distintos: **mercado/risco**, **custo da API**, **integridade de dados** e **falha operacional**. Um circuit breaker de carteira **não desliga coleta/research**.
- Usar UTC em banco, timezone local somente para exibição; NTP/clock drift testado.
- Dead-letter queue para eventos inválidos; replay de eventos para recuperação; lock de instância única; SQLite WAL + busy timeout + transações curtas; backups consistentes antes de migrations; sem apagar arquivos de dados antigos.
- Retenção: eventos de decisão e ledger permanentes; raw de alta frequência por janelas com compressão/arquivamento e política que **não destrua auditabilidade**; custos de disco medidos.
- Segredos em `.env.local`/Windows Credential Manager com ACLs; nunca git/log/frontend, tokens de API expostos ou chave de wallet; uso da chave do PumpPortal para **dados pagos** não equivale a conectar wallet de investimento.

### 10.2. Política de prioridade (evitar lag e contas enormes)

`TIER0`: todos os novos tokens de discovery (evento leve, dedup, persistência).  
`TIER1`: candidatos com primeira checagem básica e liquidez/atividade elegíveis.  
`TIER2`: Top candidatos com trade stream pago, on-chain enrichment e social (até budget).  
`TIER3`: posições abertas e oportunidades críticas; monitoramento de saída tem precedência sobre novas compras.

Quando o sistema estiver sobrecarregado, reduzir enriquecimento dos tokens de baixa prioridade, **não** deixar posições sem monitoramento de saída; registrar o que deixou de coletar.

### 10.3. SLOs iniciais, metas a testar e ajustar

- **Saúde:** nenhuma perda silenciosa de serviço; alarme/estado em até 30 s de heartbeat faltante (meta operacional).
- **Descoberta:** medir p50/p95/p99 da criação à primeira observação; **não prometer** 100ms em PC doméstico.
- **Caminho de decisão:** após todos os dados necessários estarem disponíveis, política determinística p95 alvo <= 1 segundo em workload representativo, sem depender de LLM remoto.
- **Social:** mostrar atraso real, cobertura e orçamento, não anunciar social “realtime” quando consulta for periódica.
- **Estado do dashboard:** updates com sequência/offset; após refresh retomar sem duplicar trades ou “chat” de agentes.

---

## 11. METAS E CRITÉRIOS DE SUCESSO

**Não usar “lucro garantido” ou “100% de acertos”.** Critérios de *qualidade* e *possível edge* devem ser distintos.

### Produto e dados (necessários para declarar v2 implementado)

- Pipeline `event -> opportunity -> votes -> decision -> paper_fill -> position -> exit -> ledger` passando em teste ponta a ponta.
- Novos tokens persistidos enquanto feed ativo; métricas de cobertura/lag e gaps auditáveis.
- Todos os agentes previstos executam função verificável ou ficam explicitamente desabilitados; nenhum falso consenso.
- Compras da carteira consensual com **todos os votos obrigatórios positivos no mesmo ciclo**.
- Saúde dos serviços visível no dashboard; sem false-live; persistência reinicializável.
- Compra e venda fictícias calculadas com rotas/reservas/custos verificados, quando possíveis; casos indeterminados não viram lucro fictício.
- Social API integrada em modo opt-in, com teste de autenticação, data real, orçamento, política de cache e fallback transparente.
- UI responsiva com escritório pixel, agent logs reais, gráficos, trades, notícias e missed moonshots.
- Legado preservado e backups testados.

### Edge financeiro (não presumir que será atingido)

- Definir antes da coleta uma janela prospectiva e número mínimo de trades **compatível com a variância observada**, documentando análise de poder; não inventar que “100 trades provam edge”.
- Medir PnL líquido, retorno, equity, max drawdown, pior perda, win rate, profit factor, EV por trade, custo por trade, taxa de ordem executável, variância, sensibilidade à latência, dependência de poucos moonshots e intervalos de confiança.
- Comparar com os baselines congelados, aleatório pareado, sem trade e regras atuais, no mesmo período e mesmos custos.
- Proibir prometer rentabilidade após uma noite positiva. Exigir replicação fora da amostra e por mais de um regime de mercado antes de declarar sinal econômico robusto.
- Se todas as estratégias forem negativas, **mostrar claramente o resultado e a melhor informação extraída**; não falsificar lucro.

---

## 12. O NOVO SITE — TERMINAL PROFISSIONAL + PEQUENO ESCRITÓRIO DE AGENTES

### 12.1. Direção criativa: **“MEMECOIN LAB / MISSION CONTROL”**

**Queremos uma mistura de trading terminal premium, sala de comando e personagens fofos do tipo pixel art que trabalham em miniestações.** Não quero uma landing page genérica de IA, cartões gigantes nem gráficos sem contexto. Quero abrir o sistema de manhã e entender o que aconteceu enquanto dormia: quais tokens foram vistos, quais passaram no conselho, quais perderam dinheiro, o que foi comprado/vendido no papel e que oportunidades 10x/20x/100x foram perdidas.

**Identidade:** desktop-first, profissional e editorial, com detalhes charmosos. Sem degradê roxo gratuito, glow excessivo, centenas de bordas ou estética “casino de criptomoedas”. Inspiração em produto real de analytics e jogos pixel art, **sem copiar layouts, logotipos ou personagens protegidos**.

### 12.2. Links de imagens reais para o Claude usar como **referência visual**

Estas URLs foram identificadas em pesquisa na web em 03/11/2026. Use **como referência e inspiração**, não como assets a redistribuir sem verificar licença.

**REFERÊNCIA 1 — melhor referência de terminal: Dribbble / Crypto Trading Terminal**

- **Link direto da imagem:** https://cdn.dribbble.com/userupload/44112968/file/original-80378b545226636dc471e9a43a8b329b.png
- **Página do projeto:** https://dribbble.com/shots/26278242-Crypto-Trading-Terminal-Dashboard
- **Usar para:** densidade visual controlada, gráficos, painéis pequenos de indicadores, tabela de negociações, hierarquia tipográfica e uso de espaço.

**REFERÊNCIA 2 — melhor referência de personagens em escritório: Claude Office Visualizer**

- **Link direto da imagem:** https://raw.githubusercontent.com/paulrobello/claude-office/main/screenshots/sc_floor_office.png
- **Página/projeto open source:** https://github.com/paulrobello/claude-office
- **Usar para:** personagens pixel art nos computadores, mesas, pequenas animações, líder central e estações de trabalho. **Checar licença do código e principalmente dos assets**; inspiração não significa autorização de cópia.

**REFERÊNCIA 3 — escritório com dados: AgentGUI**

- **Link direto da imagem:** https://agent-gui-project.github.io/assets/screenshots/intro_photo.png
- **Site:** https://agent-gui-project.github.io/
- **Usar para:** visualização de agentes com terminal e detalhes de atividade, mas fazer versão mais limpa e legível.

**REFERÊNCIA 4 — área social/news de terminal de criptos: RX.AI**

- **Link direto da imagem:** https://cdn.dribbble.com/userupload/15603483/file/original-9c87b4a00eec594ea95f212e121aabcc.png?resize=%7Bwidth%7Dx%7Bheight%7D&vertical=center
- **Página do projeto:** https://dribbble.com/shots/24523134-RX-AI-Cryptocurrency-terminal-Web-application
- **Usar para:** integração editorial de notícias, influencers, tabelas de tokens e hierarquia visual. Não reaproveitar nome, identidade nem screenshot como UI final.

**REFERÊNCIA 5 — implementação de arquitetura visual observável (opcional):**

- https://github.com/glglak/agent-mission-control
- https://github.com/JoelHutchinson/agent-visualiser
- Inspirar-se no **mapeamento de telemetria para ação do personagem**, e não nas narrativas ou ícones particulares. Confirmar direitos dos sprites. Uma implementação verdadeira recebe traces reais de tarefas/agents, não GIF em loop sem fonte.

### 12.3. Paleta e tipografia propostas

```css
:root {
  --bg: #090E14;           /* fundo petróleo-quase-preto */
  --surface: #101821;      /* placas/escritório */
  --surface-raised: #16212B;
  --border: #263540;
  --text: #E9F2F2;
  --muted: #8CA0AC;
  --accent: #6FE2C2;       /* verde-mint, ações e foco */
  --accent-warm: #FFBE76;  /* alertas e research */
  --positive: #62D6A2;
  --negative: #F17D83;
  --info: #91B9FF;
  --neutral: #A3AABA;
}
```

Fonte primária: `Inter`, `Geist` ou semelhante (confirmar licença e disponibilidade). Números: `JetBrains Mono`/`IBM Plex Mono` com tabular numerals. Pixel art: tipografia pixel apenas em pequenos letreiros, **nunca** em longas tabelas. Preferir bordas discretas, alinhamento preciso e densidade de informação ajustável. Tema claro opcional depois; dark mode polido é prioridade.

### 12.4. Layout desktop ideal (1440px ou mais)

```text
┌────────────────────────────────────────────────────────────────────────────────────────┐
│ ◈ MEMECOIN LAB 2.0   • PAPER ONLY    SOLANA LIVE    X: CONNECTED   COST $0.00   ⚙    │
├───────────────┬──────────────────────────────────────────────────────────┬─────────────┤
│ NAV           │ PORTFOLIO / EQUITY / REALIZED PnL / FEES / HEALTH       │ ACTIVITY    │
│              ├──────────────────────────────────────────────────────────┤             │
│ Overview      │                                                          │ Live        │
│ Agent HQ      │               GRÁFICO PRINCIPAL                          │ decisions   │
│ Scout         │          Lightweight Charts + markers                    │ Votes       │
│ Social Radar  │                                                          │ Alerts      │
│ Trades        ├──────────────────────────────────────────────────────────┤             │
│ Moonshots     │                   AGENTS OFFICE                           │             │
│ Research      │   🔭    🌊    🛡️    🐦    📰    🧠    ⚡ (pixel sprites)    │             │
│ Replay        │   [PC]  [PC]  [PC]  [PC]  [PC]  [Desk] [Terminal]        │             │
│ Settings      │     pequenas falas reais + status + missões              │             │
├───────────────┼──────────────────────────────────────────────────────────┴─────────────┤
│               │ LIVE OPPORTUNITIES  /  OPEN POSITIONS / MISSED MOONSHOTS              │
│               │ token  idade  consenso  risco  social  entrada  PnL  origem          │
└───────────────┴────────────────────────────────────────────────────────────────────────┘
```

Na versão final, `AGENTS OFFICE` não precisa estar espremido; permitir expandir para tela inteira em `Agent HQ`. O gráfico deve continuar utilizável enquanto vê os bonequinhos. Painel lateral pode ser recolhido; nenhuma seção deve estourar a viewport e nenhum vídeo/canvas ultrapassar containers.

### 12.5. Os bonequinhos: visuais lindos, atividade **real**

Criar **personagens originais** pixel art 16x16 / 24x24 / 32x32 ampliados sem blur (`image-rendering: pixelated`), cada um com cor e objeto discreto:

- SCOUT com binóculo e radar no monitor; quando detecta token, aparece um pequeno pulso verde.
- FLOW com gráfico/ondas e calculadora; digita ao atualizar métricas on-chain.
- ONCHAIN com cubos/elos e visor de blockchain; observa reservas e holders.
- VET com escudo/pequena lupa; ao rejeitar, aparece luz âmbar/vermelha.
- SOCIAL com passarinho genérico original ou celular (não copiar a marca X como personagem); pesquisa postagens.
- NEWS com jornal/notícia; vira páginas ao processar fonte.
- QUANT com planilha/mini gráfico; apenas trabalha quando rodando cálculo/versão de modelo.
- RISK com controle de segurança; ativa trava em evento real.
- CHIEF numa mesa central com telinha de votação e carimbo `APPROVED` / `REJECTED`.
- EXECUTOR numa estação de compra/venda simulada, confirmando fills e posições.
- RESEARCHER com livros e laboratório, mostrando análises off-line.

**Estados renderizados a partir de eventos do backend:** `IDLE`, `RECEIVING`, `ANALYZING`, `WAITING_DATA`, `VOTING`, `APPROVED`, `REJECTED`, `BLOCKED`, `ERROR`, `OFFLINE`. Cada evento tem `started_at`, `completed_at`/`expired_at`, trace e ID da tarefa. Personagem fica ocupado **somente durante atividade real**; ao terminar, volta a idle. Animações de respiração/olhar podem existir como *decoração explicitamente não operacional*.

**Microinterações:** digitar (quando tarefa em andamento), virar para o Chief após voto, enviar envelope luminosa só quando evento de mensagem estiver registrado; pausas simpáticas, pequenas faíscas no radar, hover com últimos eventos. Respeitar `prefers-reduced-motion` e não ficar distraindo durante leitura de gráfico.

### 12.6. “Chat do Conselho”: o que o usuário quer ver

Dentro de `Agent HQ`, colocar duas colunas: **mini-escritório** à esquerda e **terminal do conselho** à direita. Um evento do backend gera uma linha cronológica com tempo, agent name, tarefa, resultado e link para evidência. Uma aprovação gera linha animada ligando a mesa do agente ao Chief; a seta só avança quando o voto existe no banco. Clique em agente abre:

- Tarefa atual, fonte e instante da última análise.
- Últimos 10 eventos com links de origem.
- Sinais extraídos e versão do modelo/regra.
- Votos `APPROVE / REJECT / ABSTAIN` com motivo.
- Runtime, taxa de erro, custo de API, filas e health.
- Histórico de decisões de que participou.

**NÃO** simular stream de pensamento interno, frases aleatórias como “estou ficando rico”, prompts secretos, hallucinations ou percentuais de confiança inventados. Frases curtas podem ser templates determinísticos vinculados a campos reais; texto original de notícias deve respeitar direitos/limites de uso da fonte.

### 12.7. Páginas exigidas

**1. OVERVIEW / COMMAND CENTER**  
Resumo 24h e últimas 7d: equity (US$), caixa, PnL realizado/não realizado, taxas, ordens, win rate, drawdown, exposição, número de mints vistos, número de candidatos, aprovados/rejeitados, status das 11+ funções, gastos X/trades, último tick. Gráfico de equity **verdadeira** + marcador de trades e notícias. Visão em tempo real com relógio UTC/local.

**2. AGENT HQ / PIXEL OFFICE**  
Escritório animado em tela ampliada; terminal de voto, decisão e trabalho; clique em personagem, replay de sessão, filtros por agente e token. Alternar `Live`, `Session Replay`, `Diagnostic` sem false-live.

**3. SCOUT / LIVE RADAR**  
Tabela virtualizada pesquisável por mint/símbolo; idade, tipo Pump curve/pool, últimas leituras, fluxo, liquidez, risco, social, comunidade e estágio de votos. Ordenar por `newest`, `candidate_rank`, `social_acceleration`, `liquidity_quality`. Todos os tokens detectados ficam registrados, mas a UI carrega por página.

**4. SOCIAL RADAR / NEWSROOM**  
Feed X + news com timestamp, origem, autores distintos, mint verificado, trend narrativa, intensidade, spam/duplicatas e custo. Painel “Narrativas emergentes”, “Postagens com contrato”, “Fontes indisponíveis”. Exibir links para post original e cada extração, sem confundir publicação tardia com antecipação.

**5. TRADES & POSITIONS**  
Separar `OPEN`, `CLOSED`, `REJECTED ORDERS`, `REPLAYS`. Ao clicar: timeline de entrada/sinal, candles/quotes válidos, parcial, slippage, taxas por perna, preço médio, valor realizável, lucro realizado, patrimônio e `fidelity_level`. Mostrar verdades negativas e posições presas sem esconder.

**6. MOONSHOT FORENSICS**  
Painel “Moedas que fizeram 3x/5x/10x/20x/50x/100x”; filtros por primeiro preço válido, trajetória, qualidade do pico e exequibilidade. Clique numa moeda abre **por que perdemos ou rejeitamos**, quais agentes atrasaram, qual foi a primeira evidência, se custos e liquidez permitiam operar, e um replay visual comparando carteira principal vs contrafactual.

**7. RESEARCH / EXPERIMENTS**  
Comparar carteiras, estratégia e versão; benchmarks em mesmo período; cronologia, drawdown, EV, custos, validação out-of-sample, modelos shadow, drift, alertas de overfitting. Botão **“Exportar relatório Markdown/CSV”** deve realmente gerar arquivo.

**8. REPLAY / SESSIONS**  
Selecionar uma data/sessão com linhas do tempo e reconstituir estados dos personagens + votos + trades. Marca d'água `REPLAY` para não parecer ao vivo. Velocidade 1x/2x/4x apenas na visualização, sem alterar dados.

**9. SETTINGS / SYSTEM STATUS**  
Provedores conectados; credenciais **nunca mostradas**; orçamento X e de market data, cotas e gastos, políticas congeladas e suas versões, latência, limites de papéis, logs, backups, pausa de novas compras fictícias, exportação dos eventos.

### 12.8. Responsividade, acessibilidade e desempenho

- Desktop: grid 12 colunas; nav fixa estreita; central com gráfico + office + lista; painel lateral redimensionável.
- Laptop menor: nav ícones; terminal do conselho colapsável; oficina com scroll interno sem overflow da página.
- Mobile: cards só com dados indispensáveis; office miniatura clicável e full-screen separado, gráfico responsivo, tabela como linhas condensadas; **nada deve ultrapassar largura**.
- Virtualização de tabelas, memoização, WebSocket com buffer/backpressure, atualização de candles agrupada, refresh seletivo; FPS do escritório não deve atrapalhar dados.
- Skeletons e empty states honestos; `NO DATA` em vez de gráfico falso.
- Contraste WCAG quando possível, navegação por teclado, tooltips acessíveis, `aria-live` com parcimônia, `prefers-reduced-motion`, color coding + texto (nunca só verde/vermelho).
- Teste visual automático em 375, 768, 1280, 1440 e 1920 px. Capturas reais pós-build e avaliação manual de overflow/layout.

### 12.9. Stack recomendada (preservar o que já existe)

- **Manter:** React, TypeScript, Vite, Tailwind, Lightweight Charts, Starlette/Python.
- Animações: CSS pixel sprites simples e **PixiJS** (opcional) se performance/complexidade justificarem; usar assets originais ou permissivamente licenciados com atribuição/documentação. Evitar renderizar 3D pesado.
- Estilo: componentes acessíveis (Radix/shadcn se já disponíveis, não exigir refazer tudo); ícones Lucide; tipografia eficiente; SVG simples para alertas e fluxo; canvas para sprites apenas.
- Backend API: eventos WebSocket ou SSE com offset, REST para snapshots/histórico, filtros server-side, paginação.
- Criar uma camada `ui/events` que mapeia cada `trace.event_type` para status/anim do personagem; *toda tela ao vivo deriva do mesmo event store*.

---

## 13. API E ESQUEMA DE DADOS MÍNIMOS

Antes de adicionar tabelas, inspecionar nomes e constraints do banco atual. Se já existirem entidades equivalentes, **ampliar em migration reversível** e evitar duplicação.

```text
source_health(id, provider, state, last_seen_at, lag_ms, cost_spent, budget, errors)
raw_events(event_id PK, source, source_id, mint, event_at, observed_at, payload_hash, quality)
opportunities(id PK, mint, first_seen_at, current_status, strategy_id, last_observed)
opportunity_snapshots(id, opportunity_id, observed_at, quote_id, features_json, data_quality)
agent_tasks(id PK, opportunity_id, agent_id, started_at, ended_at, status, trace_id)
agent_votes(id PK, decision_id, opportunity_id, agent_id, vote, evidence_json, observed_at, expires_at, version)
council_decisions(id PK, opportunity_id, strategy_id, checkpoint, outcome, votes_hash, reason, created_at)
social_posts(id PK, provider, provider_post_id UNIQUE, author_id, created_at, observed_at, mint_confidence, metrics_json)
narrative_events(id PK, topic_key, first_seen_at, linked_mint, evidence_json, source_quality)
paper_orders(id PK, portfolio_id, decision_id, status, created_at, earliest_executable_at)
paper_fills(id PK, order_id, quote_snapshot_id, amount_in, amount_out, fee_breakdown_json, executed_at)
positions(id PK, portfolio_id, mint, quantity, cost_basis, opened_at, status)
portfolio_ledger(id PK, portfolio_id, event_id UNIQUE, cash_delta, quantity_delta, fees, event_at)
moonshot_milestones(id PK, mint, milestone, reference_price_id, observed_at, robust_peak_flag)
counterfactual_runs(id PK, opportunity_id, policy_version, result_json, valid_until, quality)
experiment_registry(id PK, config_hash, period_start, period_end, frozen_at, evaluation_status)
coverage_gaps(id PK, source, started_at, ended_at, reason, recoverable)
```

**Invariantes:** `UNIQUE(portfolio_id, decision_id, order_side, leg_id)` ou idempotency key equivalente; sem posição aberta duplicada; atomicidade dos fills; cap de saldo; timestamps observados; toda decisão com seus votos e `required_agent_versions` persistidos; campos numéricos monetários com `Decimal` ou unidades inteiras e arredondamento explícito, **não float binário em caixa/custo**.

### Endpoints conceituais (ajustar às rotas atuais)

```http
GET  /api/v2/overview
GET  /api/v2/health
GET  /api/v2/opportunities?status=...&cursor=...
GET  /api/v2/opportunities/{id}/timeline
GET  /api/v2/agents
GET  /api/v2/agents/{id}/tasks
GET  /api/v2/council/{decision_id}
GET  /api/v2/social?mint=...&cursor=...
GET  /api/v2/news?cursor=...
GET  /api/v2/portfolios/{id}/equity
GET  /api/v2/portfolios/{id}/trades
GET  /api/v2/moonshots?min_multiple=10
GET  /api/v2/research/experiments
GET  /api/v2/replay/{session_id}
GET  /api/v2/api-usage
WS   /api/v2/events?after_seq=...
```

`WS` deve ter `sequence_number`, heartbeats, re-sync de snapshots, autenticação local adequada, limite de payload e uma política de reconexão. Um observador web não deve ser capaz de habilitar ordem real, alterar configs congeladas ou executar shell arbitrário.

---

## 14. PLANO DE IMPLEMENTAÇÃO, COM ORDEM DE ENTREGA

### FASE 0 — Auditoria e backup, **antes de qualquer edição**

- Abrir `CLAUDE.md`, código, testes, árvore de pastas, serviços ativos, `uv`, migrations, UI e modelos.
- Listar integrações realmente existentes e seus custos atuais; avaliar se stream PumpPortal de trades está disponível/legalmente configurado e qual tipo de fonte alimenta cada decisão.
- Verificar integridade SQLite, WAL, contagem de trades/candidatos, situação de `e_signals`, filas, replay, estado de S1/S2, versão do frontend e bugs conhecidos; produzir relatório conciso.
- Backups consistentes dos bancos com SQLite backup API, com checks de restauração. Não copiar DB travado ingenuamente; não apagar sessões antigas.
- Confirmar process tree, startup no Windows, política de suspensão, health, timeouts e consumo de recursos.

**Entrega:** `docs/2.0/00_AUDIT.md`, lista de riscos e `data/backups/<timestamp>/` comprovadamente restaurável.

### FASE 1 — O coração do bot: consenso e fills

- Implementar event store + oportunidade única por mint/estratégia/checkpoint.
- Definir votos estruturados e `RequiredCouncil`, com idempotência, ABSTAIN, timeouts e versionamento.
- Conectar decisão aprovada ao executor `paper` existente sem quebrar Early/legacy.
- Revalidar `paper` contra erros de equity, taxas, preço stale, posição ilíquida e corridas simultâneas.
- Criar `CONSENSUS_2_0` e watcher que recebe *todos* os candidatos, sem alterar S1/S2.
- Criar/continuar `COUNTERFACTUAL` mesmo quando carteira travada, com rótulo explícito `REPLAY`.

**Entrega:** uma sessão ponta-a-ponta determinística com eventos sintéticos de teste e teste live de dados reais **sem compra verdadeira**. Mostrar pelo menos um cenário `all approve -> paper buy`, um `vet reject -> no buy`, um `social abstain -> no buy`, um `paper buy -> sell -> ledger`, e um `watch after circuit breaker`.

### FASE 2 — Vet + fluxo e análise de custos

- Inspecionar curva Pump.fun e fees atuais do protocol, simular volume de compra/venda com precisão adequada.
- Implementar `FLOW`, `ONCHAIN`, `VET`, `RISK` e execução precheck; ligar aos dados reais atuais.
- Implementar capacidades de compra/venda e marcadores de incerteza `F0..F3`.
- Comparar seleção com replay de universo elegível, medir falsos negativos e falsos positivos por criador/cluster.

**Entrega:** cartões de voto auditáveis; relatório de PnL de custos por perna; warnings claros quando dados insuficientes.

### FASE 3 — Social e News: integração REAL, com orçamento

- Criar `XOfficialAdapter`, `SocialAgent`, `NarrativeResolver`, `NewsAgent` e abstração alternativa LunarCrush/CryptoPanic.
- Implementar UI de credencial configurada ou ausente (sem expor segredo); `X_MAX_USD_PER_DAY=0` por padrão e variável/controle explícito de ativação, sem tentar cobrança automática.
- Após chave autorizada: teste de conectividade, consulta de poucos posts por endereço de contrato/candidatos, persistir IDs, pontuar evidências e orçamentos; provar que nenhum dado posterior retroage ao checkpoint.
- Testar casos de ticker duplicado, spam, influenciador com muitos reposts, notícias em atraso, source sem resposta e orçamento esgotado.
- Comparar performance *sem social* e *com social* em shadow prospectivo, antes de promover peso/threshold.

**Entrega:** `Social Radar` vivo com fontes e limites mostrados; agentes sociais não inventam conteúdo sem chave.

### FASE 4 — Machine Learning 2.0, sem bloquear execução

- Congelar datasets e targets econômicos; construir splits temporais/embargo.
- Benchmark regras e modelos leves; registry; shadow; drift e custo de sinal.
- Medir custo/latência de modelos; sempre rodar em worker separado; rollback automático para heurística pré-definida caso erro.
- Tratar `model_not_loaded` como estado operacional, não parar toda descoberta.

**Entrega:** relatório fora da amostra mostrando retorno líquido, perdas e limites, inclusive se todos forem negativos.

### FASE 5 — UI redesign (pode iniciar em paralelo depois dos contratos API)

- Produzir design tokens, wireframe, componentes, escritório pixel art **original** e páginas v2.
- Implementar Overview + Agent HQ + Council Console + Trades + Radar + Social + Moonshot Forensics + Research.
- Conectar **eventos reais**, permitir session replay marcado, proteger visualização de sobrecarga e testar telas responsivas.
- Capturar screenshots das páginas para inspeção visual, revisar problemas de overflow/legibilidade, acessibilidade e performance.

**Entrega:** frontend rodando no serviço local atual, usando dados reais de paper trading, sem alterar backend financeiro apenas para “ficar bonito”.

### FASE 6 — Validação prospectiva e operação

- Congelar `strategy_id`, filtros de aprovação, portfolio policy, budgets, custos e horizonte, registrar `experiment_registry`.
- Rodar experiência por janela pré-declarada em tempo de execução real; acompanhar saúde/intervalos válidos, sem dobrar amostra após olhar resultados.
- Comparar contra controle e publicar resumo de todos os trades, inclusive perdas, recall de moonshots e custos das APIs.
- Só promova estratégia após evidência suficiente e repetição robusta; não existe data certa em que será lucrativa.

**Entrega:** `docs/2.0/VALIDATION.md` + `RUNBOOK.md` + `CHANGELOG_2_0.md`, testes e demonstração no navegador.

---

## 15. TESTES QUE NÃO PODEM FALTAR

### Funcionais

1. Descoberta de 100 eventos de mint com 5 duplicados => 100 eventos brutos, 95 tokens únicos, nenhuma oportunidade duplicada por chave.
2. Todos aprovam => **uma** ordem paper; dois agentes votam duas vezes => no máximo uma ordem.
3. Qualquer veto/ABSTAIN/TIMEOUT => **nenhuma** compra principal; oportunidade acompanhada pelo replay.
4. Carteira entra em circuit breaker => posições continuam monitoradas para saídas e novas oportunidades continuam registradas.
5. Reiniciar worker no meio do processamento => nenhum fill duplicado; decisões duráveis; last_seq respeitado.
6. X retorna 429/401 ou zero budget => estado visível; custo não ultrapassado; nenhuma afirmação de sentimento.
7. Post sobre ticker ambíguo => não atribuir automaticamente ao mint de mesmo símbolo.
8. Post atrasado em 20 minutos => não é prova antecipada de compra anterior.
9. Stop cruzado entre snapshots => fill na primeira cotação disponível, não stop idealizado.
10. Compra sem quote posterior => marcação conservadora provisória, sem acionar falso breaker por valuation 0.
11. Token sem rota de saída => veto ou estado ilíquido explícito, sem criar lucro.
12. Taxa extra cobrada => PnL realizado e equity batem com ledger.
13. Candle pós-migração => não pode ser usado em trade pré-migração.
14. Modelo offline indisponível => serviços de discovery e monitoramento seguem operando; decisão reconhece falta de voto.
15. Replay de sessão => cada linha com `REPLAY`, nunca confundir com live.
16. Health OFFLINE => office mostra agente offline, sem fala fictícia.
17. Telas 375/768/1280/1440/1920: sem scroll horizontal indevido, texto legível.
18. Pagamentos externos: nenhum provider com `auto_purchase`/gasto habilitado sem autorização explícita.
19. `rg` por comandos de swap real/keys => nenhuma rota de execução de cripto real ativa na aplicação.
20. Verificação dos backups do banco e migrations; resultado da carteira antiga inalterado.

### Quantitativos

- Backtest *lookahead trap*: evento social recebido **depois** da entrada não pode melhorá-la.
- Leakage por creator: train e test com mecanismos de agrupamento e avaliação temporal.
- Sensibilidade a 1s, 3s, 10s, 30s de atraso e slippage maior que baseline.
- Liquidity shock: vendedor grande retira reservas antes de saída e a política admite prejuízo superior ao stop ideal.
- Custos por tranche de US$2: comparador parcial vs saída integral preservando capital/fees.
- Random control matched e holdout; bootstrap por bloco/creator, sem tratar 40 checkpoints de um token como 40 exemplos independentes.
- Criação de token copycat com ticker idêntico a moeda famosa não herda o sentimento de original.
- Reprodutibilidade: mesmo raw dataset, config hash e código => mesmo resultado em replay.

---

## 16. PROMPT EXECUTIVO PRONTO PARA COLAR NO CLAUDE CODE

> **Cole a instrução a seguir no Claude Code com este arquivo disponível no projeto.** Ele deve implementar e executar tarefas reais, não apenas gerar outra proposta.

```text
VOCÊ É O ENGENHEIRO PRINCIPAL DO MEMECOIN LAB 2.0.

Repositório local: C:\Users\gebt\Downloads\memecoins
Documento mestre: MEMECOIN_LAB_2_0_MASTER_PLAN.md

LEIA ESTE DOCUMENTO INTEIRO, LEIA CLAUDE.md E AUDITE CÓDIGO/BANCOS/SERVIÇOS ANTES DE EDITAR.

Meu bot atual já coleta Pump.fun/PumpSwap, tem Early, Moonshot, HighFreq, ML, Shadow,
Paper e UI React. Ele ainda perde dinheiro fictício e às vezes apenas observa
moedas valorizarem sem que a operação ou a análise sejam feitas. NÃO QUEBRE
DADOS ANTIGOS NEM TRANSFORME ISSO EM UM PROJETO NOVO VAZIO.

EXECUTE O PLANO DE IMPLEMENTAÇÃO em ordem. Comece pelo backup verificado,
conselho com votos reais, caminho crítico de Paper/ledger e monitor de todas
as oportunidades. Depois implemente agentes on-chain, social/news, research e
UI, respeitando dependências e orçamentos. Ao final de cada fase, deixe o
software funcional e reporte arquivos editados, comandos rodados, testes e
resultados realmente observados.

QUERO AGENTES REAIS: SCOUT, FLOW, ONCHAIN, VET, SOCIAL, NEWS, QUANT, RISK,
CHIEF, EXECUTOR, PORTFOLIO, RESEARCHER. Uma carteira de consenso somente
compra ficticiamente após votos APPROVE de TODOS os agentes obrigatórios
para aquela estratégia (sem veto e com dados frescos). Se falta fonte, o agente
ABSTAIN e o bot monitora/reavalia; não invente aprovação. Compare resultados
com replay independente de moedas rejeitadas para não ficar cego aos 10x.

X/TWITTER: adicione integração via API OFICIAL e limites de custo por padrão
0; você deve criar conector, health, adapters, source mapping e UI, mas NÃO
comprar créditos nem afirmar conexão sem chave válida. Faça monitoramento de
narrativas, posts por mint, autores únicos, spam, timing e relação on-chain;
crie News Agent e comparações quantitativas com e sem redes sociais.

NÃO CARREGUE MODELO PESADO PARA DECIDIR EM CADA TICK. Fast path leve,
async, workers warm, LLM somente para narrativa e tarefas oportunas;
modelos pesados fora do caminho de trading fictício e com timeouts.

INTERFACE: reconstruir visual com referências de imagens e links do item 12,
com terminal premium preto/petróleo, gráfico lindo, PnL verdadeiro,
feed de notícias, radares, trades, missed moonshots e ESCRITÓRIO PIXEL ART
ORIGINAL dos agentes, com bonequinhos digitando, votando e conversando.
Mas personagens e terminal devem derivar de AGENT_TASKS, AGENT_VOTES e
EVENTOS reais. Sem chat decorativo ou lucro fabricado. Seguir responsividade.

REGRAS INEGOCIÁVEIS:
- 100% PAPER TRADING, sem ordem real, assinatura ou uso de chave privada.
- Não ligar API paga ou custear X/PumpPortal sem autorização explícita.
- Não apagar/resetar ledgers existentes para mostrar lucro.
- Não prometer rentabilidade nem manipular holdout.
- Não usar informação futura em decisões/fills.
- Não parar após gerar planos: implementar, testar e demonstrar o que for possível.
- Não adicionar frameworks pesados sem necessidade; reutilizar o stack.
- Se uma fase não puder ser concluída (sem credencial, rede, etc.), entregar
  o máximo funcional sem inventar integração, marcar bloqueio claramente
  e continuar as partes independentes.

PRIMEIRO OUTPUT ESPERADO: diagnóstico real do código/serviços/bancos e backup
restaurável. EM SEGUIDA faça a FASE 1, com teste end-to-end comprovado e
um dashboard que mostre as decisões reais. Continue fases seguintes em ordem.
Me dê sempre evidência verificável, não slogans.
```

---

## 17. MATRIZ DE FONTES DA PESQUISA — CONSULTADAS EM 03/11/2026

**Documentação primária de market data e execução hipotética**

1. Pump.fun public protocol SDK/docs — https://github.com/pump-fun/pump-public-docs
2. Pump.fun fees — https://pump.fun/docs/fees
3. PumpPortal WebSocket Data API — https://pumpportal.fun/data-api/bonk-fun-data-api/
4. PumpPortal Fees — https://pumpportal.fun/fees/
5. Solana RPC overview — https://solana.com/docs/rpc
6. Solana RPC WebSocket methods — https://solana.com/docs/rpc/websocket
7. Solana Token/Token-2022 — https://solana.com/docs/tokens
8. Solana Transfer Hook — https://solana.com/docs/tokens/extensions/transfer-hook
9. DEX Screener API reference — https://docs.dexscreener.com/api/reference
10. GeckoTerminal API — https://api.geckoterminal.com/docs/index.html
11. GeckoTerminal free rate limit (outubro 2026) — https://support.coingecko.com/articles/6867086195-does-geckoterminal-have-an-api
12. Jupiter Swap API v2 (somente quote para paper) — https://developers.jup.ag/docs/swap
13. Helius streaming infra — https://www.helius.dev/blog/laserstream-websockets

**Social, notícias e observabilidade**

14. X official pay-per-use pricing — https://docs.x.com/x-api/getting-started/pricing
15. X official post search docs — https://docs.x.com/x-api/posts/search/introduction
16. X official filtered stream — https://docs.x.com/x-api/posts/filtered-stream/introduction
17. X official query syntax — https://docs.x.com/x-api/posts/search/integrate/build-a-query
18. LunarCrush official social API — https://lunarcrush.com/en/developers/api
19. CryptoPanic official API — https://cryptopanic.com/developers/api/
20. OpenTelemetry conventions for GenAI agents — https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-agent-spans.md

**Artigos científicos e risco de inferência**

21. Szwajcok et al., *Meme Coin Factories* (2026) — https://arxiv.org/abs/2609.10246
22. Mongardini & Mei, *A Midsummer Meme's Dream* (USENIX 2026) — https://www.usenix.org/conference/usenixsecurity26/presentation/mongardini
23. Li et al., *Catching the Rug* (2026) — https://arxiv.org/abs/2608.20271
24. Chen et al., *SolRugDetector* (2026) — https://arxiv.org/abs/2603.24625
25. Ardia & Bluteau, *Twitter and cryptocurrency pump-and-dumps* (2024) — https://doi.org/10.1016/j.irfa.2024.103479
26. Bailey et al., *The Probability of Backtest Overfitting* — https://papers.ssrn.com/sol3/Papers.cfm?abstract_id=2326253

**Referências de layout e personagens**

27. Trading Terminal — https://dribbble.com/shots/26278242-Crypto-Trading-Terminal-Dashboard
28. Claude Office — https://github.com/paulrobello/claude-office
29. AgentGUI — https://agent-gui-project.github.io/
30. RX.AI UI — https://dribbble.com/shots/24523134-RX-AI-Cryptocurrency-terminal-Web-application
31. Agent Mission Control — https://github.com/glglak/agent-mission-control
32. Agent Visualiser — https://github.com/JoelHutchinson/agent-visualiser

**Limite da pesquisa:** documentação muda, APIs podem exigir credenciais, planos e quotas dependem da conta, e artigos científicos não comprovam lucratividade de qualquer estratégia proposta aqui. Versões, limites e taxas devem ser validados na implementação. Links de imagem servem para **inspiração e avaliação**, não licença automática de uso comercial. Os vídeos do Grok Bot/Jev inspiram a ideia do conselho; os resultados financeiros mostrados neles não foram verificados independentemente e não constituem prova de que este projeto será lucrativo.

---

## 18. CRITÉRIO FINAL: O QUE PRECISA ESTAR VERDADEIRO PARA DIZER “V2.0 PRONTA”

**A V2 não está pronta por ter 12 avatars, 50 testes ou um site bonito.** Está pronta tecnicamente quando a cadeia inteira funciona, com fontes e agentes auditáveis, risco e custos conservadores, operações em paper trading, replays honestos e dashboard visual belo. Estará **economicamente validada** apenas se, depois de uma quantidade adequada de operações prospectivas, existir vantagem líquida replicável contra os controles e sem depender de manipulação retrospectiva.

> **Quero ver o bot procurar, investigar, votar, comprar ficticiamente, administrar a posição, vender, contabilizar e aprender com os erros — tudo com transparência. Prefiro um sistema que registra corretamente perdas e oportunidades perdidas a um que mostra lucro impossível. Mas ele não deve ficar passivo: o watcher, as estratégias e a pesquisa continuam trabalhando mesmo quando não é seguro abrir uma posição.**

**NOME DA FASE:** `MEMECOIN LAB 2.0 — CONSENSUS / SOCIAL INTELLIGENCE / MISSION CONTROL`  
**INÍCIO AUTORIZADO:** arquitetura e paper trading exclusivamente.  
**NENHUMA GARANTIA DE LUCRO. NENHUMA TRANSAÇÃO REAL.**