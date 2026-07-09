# KRONOS CORE — Telemetry Dataset

[English](#english) | [Português](#portugues)

<a id="english"></a>

## English

Operational telemetry dataset from [KRONOS CORE](https://github.com/carmipa/traducao_animes_llm_local_quarkus), an anime subtitle translation pipeline that runs local LLM inference through LM Studio.

This repository is meant to expose reproducible performance and pipeline metrics, not subtitle content. Each commit is a dataset snapshot; the Git history is the public timeline.

### Repository Layout

```text
├── README.md
├── LICENSE
└── metrics/
    └── kronos-telemetria-dataset.json
```

### Data Format

The dataset uses a custom UTF-8 JSON format with `versaoFormato` for schema evolution.

#### `ambienteExecucao`

Safe execution-environment metadata for benchmark context.

| Field | Meaning |
|-------|---------|
| `fabricante` / `modeloMaquina` | Generic manufacturer and machine model reported by the OS |
| `cpu` | Public CPU name |
| `gpuPrincipal` | GPU name published for benchmark comparison |
| `gpuDetectadaSistema` | GPU name reported by the OS/driver when it differs from the configured public name |
| `ramTotalGb` | Rounded total physical RAM in GB |
| `sistemaOperacional` / `arquitetura` | Runtime platform, without username, hostname, paths, IPs or device IDs |
| `hardwareColetadoAutomaticamente` | Whether the values were collected automatically from the local system |
| `gpuPublicaConfigurada` | Whether a public GPU override was configured |

#### `resumo`

Aggregate metrics.

| Field | Meaning |
|-------|---------|
| `totalEpisodiosTraduzidos` | Episodes processed by the LLM translation pipeline |
| `totalLinhasTraduzidas` | Subtitle dialogue lines translated |
| `tempoMedioPorLinhaMs` | Average translation latency per dialogue line |
| `totalFalasReaproveitadasDoCache` | Dialogue lines resolved from persistent cache without another LLM call |
| `alucinacoesLlmPrevenidas` | LLM responses rejected by anti-hallucination guards |
| `arquivosRenomeados` | Files normalized by the rename module |
| `totalOperacoesRegistradas` | Recorded pipeline operations across modules |

#### `traducoesLlm[]`

Per-episode LLM translation metrics.

| Field | Meaning |
|-------|---------|
| `episodio` | Subtitle filename only, without directories |
| `anime` / `temporada` | Work and season |
| `modeloLlm` | Local model id reported by LM Studio |
| `totalLinhas` / `falasTraduzidas` / `falasDoCache` | Workload and translation source |
| `tempoTotalMs` | Total episode translation duration |
| `quantidadeAvisos` | Count of quality warnings, without warning text |
| `registradoEm` | UTC ISO-8601 timestamp |

#### `operacoes[]`

Generic pipeline-operation metrics: `tipo`, `tempoTotalMs`, `arquivosProcessados`, `itensDetectados`, `itensCorrigidos`, `registradoEm`.

This covers remuxing, subtitle extraction, lore/review steps, karaoke processing, file renaming and audits.

### Privacy And Anonymization

This dataset does not publish subtitle text, local machine paths, usernames, hostnames, IP addresses, MAC addresses, serial numbers, device identifiers, credentials, tokens or API keys.

The only public identifiers are release/work names, local LLM model ids and generic hardware metadata useful for benchmark interpretation.

### Generation

The dataset is generated from the KRONOS CORE Telemetry panel through the **Publicar Dataset** button. KRONOS sanitizes the accumulated telemetry, writes `metrics/kronos-telemetria-dataset.json`, commits the snapshot and pushes it to this repository.

### License

[MIT](LICENSE) — free use with attribution.

<a id="portugues"></a>

## Português

Dataset de telemetria operacional do [KRONOS CORE](https://github.com/carmipa/traducao_animes_llm_local_quarkus), uma esteira de tradução de legendas de anime que executa inferência LLM local via LM Studio.

Este repositório existe para expor métricas reprodutíveis de performance e pipeline, não conteúdo de legendas. Cada commit é um snapshot do dataset; o histórico Git é a linha do tempo pública.

### Estrutura Do Repositório

```text
├── README.md
├── LICENSE
└── metrics/
    └── kronos-telemetria-dataset.json
```

### Formato Dos Dados

O dataset usa JSON próprio em UTF-8, com `versaoFormato` para evolução do schema.

#### `ambienteExecucao`

Metadados seguros do ambiente de execução para contextualizar benchmarks.

| Campo | Significado |
|-------|-------------|
| `fabricante` / `modeloMaquina` | Fabricante e modelo genérico reportados pelo sistema operacional |
| `cpu` | Nome público do processador |
| `gpuPrincipal` | GPU publicada para comparação de benchmark |
| `gpuDetectadaSistema` | GPU detectada pelo sistema/driver quando difere do nome público configurado |
| `ramTotalGb` | RAM física total arredondada em GB |
| `sistemaOperacional` / `arquitetura` | Plataforma de execução, sem usuário, hostname, caminhos, IPs ou IDs de dispositivo |
| `hardwareColetadoAutomaticamente` | Indica se os valores foram coletados automaticamente do sistema local |
| `gpuPublicaConfigurada` | Indica se houve override público da GPU |

#### `resumo`

Métricas agregadas.

| Campo | Significado |
|-------|-------------|
| `totalEpisodiosTraduzidos` | Episódios processados pelo pipeline de tradução LLM |
| `totalLinhasTraduzidas` | Falas de legenda traduzidas |
| `tempoMedioPorLinhaMs` | Latência média de tradução por fala |
| `totalFalasReaproveitadasDoCache` | Falas resolvidas pelo cache persistente sem nova chamada ao LLM |
| `alucinacoesLlmPrevenidas` | Respostas de LLM rejeitadas pelas guardas anti-alucinação |
| `arquivosRenomeados` | Arquivos padronizados pelo módulo de renomeação |
| `totalOperacoesRegistradas` | Operações de pipeline registradas entre os módulos |

#### `traducoesLlm[]`

Métricas de tradução LLM por episódio.

| Campo | Significado |
|-------|-------------|
| `episodio` | Nome do arquivo de legenda, sem diretórios |
| `anime` / `temporada` | Obra e temporada |
| `modeloLlm` | Modelo local usado, conforme id reportado pelo LM Studio |
| `totalLinhas` / `falasTraduzidas` / `falasDoCache` | Volume e origem das traduções |
| `tempoTotalMs` | Duração total da tradução do episódio |
| `quantidadeAvisos` | Contagem de avisos de qualidade, sem texto dos avisos |
| `registradoEm` | Timestamp UTC em ISO-8601 |

#### `operacoes[]`

Métricas genéricas por operação de pipeline: `tipo`, `tempoTotalMs`, `arquivosProcessados`, `itensDetectados`, `itensCorrigidos`, `registradoEm`.

Cobre remux, extração de legendas, revisões de lore/concordância, karaokê, renomeação de arquivos e auditorias.

### Privacidade E Anonimização

Este dataset não publica texto de legenda, caminhos locais da máquina, nomes de usuário, hostnames, endereços IP, endereços MAC, números de série, identificadores de dispositivo, credenciais, tokens ou chaves de API.

Os únicos identificadores públicos são nomes de obras/releases, ids de modelos LLM locais e metadados genéricos de hardware úteis para interpretar benchmarks.

### Geração

O dataset é gerado pelo painel de Telemetria do KRONOS CORE através do botão **Publicar Dataset**. O KRONOS sanitiza a telemetria acumulada, escreve `metrics/kronos-telemetria-dataset.json`, commita o snapshot e faz push para este repositório.

### Licença

[MIT](LICENSE) — uso livre com atribuição.
