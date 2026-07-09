# KRONOS CORE — Telemetry Dataset

> **English summary:** operational metrics dataset from [KRONOS CORE](https://github.com/carmipa/traducao_animes_llm_local_quarkus),
> an industrial anime-subtitle translation pipeline running a 100% local LLM (LM Studio).
> Metrics only — no subtitle texts, no personal data, no machine paths. Updated via one-click
> publish from the running system; the Git history is the snapshot timeline.

Dataset de **métricas operacionais** do [KRONOS CORE](https://github.com/carmipa/traducao_animes_llm_local_quarkus) —
pipeline industrial de tradução de legendas de anime com **LLM 100% local** (LM Studio, sem nuvem).
Cada commit é um snapshot; o histórico Git é a linha do tempo do dataset.

## Estrutura

```
├── README.md
├── LICENSE
└── metrics/
    └── kronos-telemetria-dataset.json
```

## Formato dos dados

JSON próprio (documentado abaixo), UTF-8, com `versaoFormato` para evolução do schema.

### `resumo` (agregado)

| Campo | Significado |
|-------|-------------|
| `totalEpisodiosTraduzidos` | Episódios processados pelo pipeline de tradução LLM |
| `totalLinhasTraduzidas` | Falas de legenda traduzidas |
| `tempoMedioPorLinhaMs` | Latência média de tradução por fala (LLM local) |
| `totalFalasReaproveitadasDoCache` | Falas resolvidas pelo cache persistente (sem chamada de LLM) |
| `alucinacoesLlmPrevenidas` | Respostas de LLM rejeitadas pelas guardas anti-alucinação |
| `arquivosRenomeados` | Arquivos padronizados pelo módulo de renomeação |
| `totalOperacoesRegistradas` | Operações de pipeline registradas (todos os módulos) |

### `traducoesLlm[]` (por episódio)

| Campo | Significado |
|-------|-------------|
| `episodio` | Nome do arquivo de legenda (sem diretórios) |
| `anime` / `temporada` | Obra e temporada |
| `modeloLlm` | Modelo local usado (id reportado pelo LM Studio) |
| `totalLinhas` / `falasTraduzidas` / `falasDoCache` | Volume e origem das traduções |
| `tempoTotalMs` | Duração total da tradução do episódio |
| `quantidadeAvisos` | Quantidade de avisos de qualidade (falas mantidas sem tradução, suspeitas etc.) |
| `registradoEm` | Timestamp UTC (ISO-8601) |

### `operacoes[]` (por operação de pipeline)

`tipo`, `tempoTotalMs`, `arquivosProcessados`, `itensDetectados`, `itensCorrigidos`, `registradoEm` —
cobre remux, extração, revisões (lore/concordância), karaokê, renomeação e auditorias.

## Anonimização (LGPD/GDPR)

Este dataset **não contém PII**: sem textos de legenda (conteúdo protegido vira apenas contagem
de avisos), sem caminhos de disco/usuário, sem IPs, tokens ou credenciais. Os únicos
identificadores são nomes públicos de obras/arquivos de release e ids de modelos LLM.

## Licença

[MIT](LICENSE) — uso livre com atribuição.

## Como é gerado

Botão **"Publicar Dataset"** no painel de Telemetria do KRONOS CORE: o sistema sanitiza a
telemetria acumulada, escreve `metrics/kronos-telemetria-dataset.json`, commita e faz push.
