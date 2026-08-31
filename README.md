# KRONOS CORE — Telemetry Dataset

[English](#english) | [Português](#portugues)

<a id="english"></a>

## English

Operational telemetry dataset from [KRONOS CORE](https://github.com/carmipa/traducao_animes_llm_local_quarkus), an anime subtitle translation pipeline that runs local LLM inference through LM Studio.

Source translator project: [carmipa/traducao_animes_llm_local_quarkus](https://github.com/carmipa/traducao_animes_llm_local_quarkus)

This repository is meant to expose reproducible performance and pipeline metrics, not subtitle content. Each commit is a dataset snapshot; the Git history is the public timeline.

### Repository Layout

```text
├── README.md
├── LICENSE
└── metrics/
    ├── kronos-telemetria-dataset.json     # snapshot (latest state per episode)
    ├── kronos-telemetria-execucoes.jsonl  # append-only archive, one line per run
    ├── kronos-karaoke-execucoes.jsonl     # append-only, one line per karaoke subtitle file
    ├── fatias/                            # per-module consolidated metrics
    └── csv/                               # same data, tabular
        ├── kronos-resumo.csv
        ├── kronos-ambiente-execucao.csv
        ├── kronos-traducoes-llm.csv
        ├── kronos-operacoes.csv
        ├── kronos-execucoes.csv
        ├── kronos-avisos.csv
        ├── kronos-karaoke.csv
        └── kronos-karaoke-avisos.csv
```

### CSV Files

Every metric published as JSON is also published as CSV, generated from the **same
in-memory node** — the two formats cannot drift apart.

- **Encoding** UTF-8, no BOM · **separator** `,` · **quoting** RFC 4180 (embedded quotes
  doubled) · **line ending** LF.
- Real line breaks inside a field are written as the two-character text `
`, so one
  physical line is always one record.
- `kronos-avisos.csv` is tidy data: **one row per warning**, joinable back to a run by
  `registradoEm` + `nomeEpisodio`.
- `kronos-karaoke.csv` is the **song-lyric pipeline**, one row per subtitle FILE — a
  different unit from the dialogue tables, because one karaoke run sweeps a whole folder.
  `desfechoArquivo` has three values on purpose: `TRADUZIDO`, `FALHOU` and `NAO_ALCANCADO`
  (the run died before reaching any file). Its own counters — `preservadasOriginalJapones`,
  `efeitosKfxPreservados`, `acentosRepostos`, `entradasCacheDescartadas` — do not exist in
  the dialogue schema. `kronos-karaoke-avisos.csv` is its tidy warning table, joinable by
  `registradoEm` + `arquivo`.
- `origemDoRegistro` diz **quais campos de uma linha de karaokê significam alguma coisa**.
  Linha `EXECUCAO` foi medida por uma execução ao vivo e tem todos os campos preenchidos.
  Linha `MANIFESTO_HISTORICO` foi reconstruída de manifestos de auditoria escritos antes
  desses campos existirem, então `statusExecucao`, `estadoDicionario`, `acentosRepostos` e
  `entradasCacheDescartadas` voltam vazios **porque nada os mediu** — não porque o valor
  era zero. Filtre por esta coluna em vez de adivinhar.
- `origemDoRegistro` tells you **which fields of a karaoke row mean anything**.
  `EXECUCAO` rows were measured by a live run and every field is populated.
  `MANIFESTO_HISTORICO` rows were rebuilt from audit manifests written before those
  fields existed, so `statusExecucao`, `estadoDicionario`, `acentosRepostos` and
  `entradasCacheDescartadas` come back empty **because nothing measured them** — not
  because the value was zero. Filter on this column instead of guessing.

Opening in Excel: import as UTF-8 / comma-separated instead of double-clicking, otherwise
accented characters and comma-bearing titles are misread.

### Data Format

The dataset uses a custom UTF-8 JSON format with `versaoFormato` for schema evolution.

#### `ambienteExecucao`

Safe execution-environment metadata for benchmark context.

| Field | Meaning |
|-------|---------|
| `fabricante` / `modeloMaquina` | Generic manufacturer and machine model reported by the OS |
| `cpu` | Public CPU name |
| `gpuPrincipal` | Dedicated GPU selected automatically from the current machine |
| `gpusDetectadas` | All GPUs reported by the current operating system/driver |
| `ramTotalGb` | Rounded total physical RAM in GB |
| `sistemaOperacional` / `arquitetura` | Runtime platform, without username, hostname, paths, IPs or device IDs |
| `hardwareColetadoAutomaticamente` | Whether the values were collected automatically from the local system |

#### `resumo`

Aggregate metrics.

| Field | Meaning |
|-------|---------|
| `totalEpisodiosTraduzidos` | Episodes processed by the LLM translation pipeline |
| `totalLinhasTraduzidas` | Subtitle dialogue lines translated |
| `tempoMedioPorLinhaMs` | Average translation latency per dialogue line |
| `totalFalasReaproveitadasDoCache` | Dialogue lines resolved from persistent cache without another LLM call |
| `alucinacoesLlmPrevenidas` | LLM responses rejected by anti-hallucination guards |
| `respostasTraducaoRejeitadas` | Invalid model attempts rejected before persistence |
| `falhasTraducaoRecuperadas` | Lines recovered by a later validated retry |
| `fallbacksTraducaoMantidos` | Distinct lines still pending after retry exhaustion |
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
| `falasItalicoRemovido` | Lines whose italic was stripped by the italic rule (null = not measured) |
| `falasItalicoPreservado` | Lines the rule ABSTAINED from: italic inherited from the `Style:` header |
| `quantidadeAvisos` | Count of quality warnings, without warning text |
| `registradoEm` | UTC ISO-8601 timestamp |

#### `operacoes[]`

Generic pipeline-operation metrics: `tipo`, `tempoTotalMs`, `arquivosProcessados`, `itensDetectados`, `itensCorrigidos`, `registradoEm`.

This covers remuxing, subtitle extraction, lore/review steps, karaoke processing, file renaming and audits.

### Privacy And Anonymization

This dataset does not publish local machine paths, usernames, hostnames, IP addresses, MAC addresses, serial numbers, device identifiers, credentials, tokens or API keys.

**Subtitle excerpts are published, deliberately and only in the warning tables.** Pipeline warnings in `kronos-telemetria-execucoes.jsonl` / `metrics/csv/kronos-avisos.csv` (dialogue) and `kronos-karaoke-execucoes.jsonl` / `metrics/csv/kronos-karaoke-avisos.csv` (song lyrics) quote the subtitle line that triggered the failure — for example a line kept untranslated because the model corrupted its ASS tags. Without the line itself, the failure cannot be studied or reproduced, which is the point of publishing translation telemetry at all. The karaoke tables quote **song lyric lines** for the same reason and under the same rule: only the line that failed, never the full lyric.

These are short diagnostic excerpts from fansub subtitle files, published for research into machine-translation failure modes. They are not a translated corpus and no complete subtitle file is redistributed. If you hold rights over a quoted line and want it removed, open an issue.

The aggregate metrics (`kronos-telemetria-dataset.json` and the other CSVs) contain **no** subtitle text: warnings there are reduced to `quantidadeAvisos`.

The only other public identifiers are release/work names, local LLM model ids and generic hardware metadata useful for benchmark interpretation.

### Generation

The dataset is generated from the KRONOS CORE Telemetry panel through the **Publicar Dataset** button. KRONOS sanitizes the accumulated telemetry, writes `metrics/kronos-telemetria-dataset.json`, commits the snapshot and pushes it to this repository.

### License

[MIT](LICENSE) — free use with attribution.

<a id="portugues"></a>

## Português

Dataset de telemetria operacional do [KRONOS CORE](https://github.com/carmipa/traducao_animes_llm_local_quarkus), uma esteira de tradução de legendas de anime que executa inferência LLM local via LM Studio.

Projeto do tradutor: [carmipa/traducao_animes_llm_local_quarkus](https://github.com/carmipa/traducao_animes_llm_local_quarkus)

Este repositório existe para expor métricas reprodutíveis de performance e pipeline, não conteúdo de legendas. Cada commit é um snapshot do dataset; o histórico Git é a linha do tempo pública.

### Estrutura Do Repositório

```text
├── README.md
├── LICENSE
└── metrics/
    ├── kronos-telemetria-dataset.json     # foto (último estado por episódio)
    ├── kronos-telemetria-execucoes.jsonl  # acervo append-only, uma linha por execução
    ├── kronos-karaoke-execucoes.jsonl     # append-only, uma linha por arquivo de karaokê
    ├── fatias/                            # consolidado por módulo
    └── csv/                               # os mesmos dados, em tabela
        ├── kronos-resumo.csv
        ├── kronos-ambiente-execucao.csv
        ├── kronos-traducoes-llm.csv
        ├── kronos-operacoes.csv
        ├── kronos-execucoes.csv
        ├── kronos-avisos.csv
        ├── kronos-karaoke.csv
        └── kronos-karaoke-avisos.csv
```

### Arquivos CSV

Toda métrica publicada em JSON é publicada também em CSV, gerada a partir do **mesmo nó em
memória** — os dois formatos não têm como divergir.

- **Codificação** UTF-8 sem BOM · **separador** `,` · **aspas** RFC 4180 (aspas internas
  duplicadas) · **fim de linha** LF.
- Quebra de linha real dentro de campo é gravada como o texto `
` de dois caracteres, para
  uma linha física ser sempre um registro.
- `kronos-avisos.csv` é tidy data: **uma linha por aviso**, ligada à execução por
  `registradoEm` + `nomeEpisodio`.
- `kronos-karaoke.csv` é o pipeline de **letra de música**, uma linha por ARQUIVO de
  legenda — unidade diferente das tabelas de diálogo, porque uma execução de karaokê varre
  a pasta inteira. `desfechoArquivo` tem três valores de propósito: `TRADUZIDO`, `FALHOU` e
  `NAO_ALCANCADO` (a execução morreu antes de alcançar arquivo nenhum). Os contadores
  próprios dela — `preservadasOriginalJapones`, `efeitosKfxPreservados`, `acentosRepostos`,
  `entradasCacheDescartadas` — não existem no schema do diálogo.
  `kronos-karaoke-avisos.csv` é a tabela tidy de avisos dela, ligada por
  `registradoEm` + `arquivo`.

Abrindo no Excel: importe como UTF-8 / separado por vírgula em vez de dar duplo clique,
senão acento e título com vírgula saem errados.

### Formato Dos Dados

O dataset usa JSON próprio em UTF-8, com `versaoFormato` para evolução do schema.

#### `ambienteExecucao`

Metadados seguros do ambiente de execução para contextualizar benchmarks.

| Campo | Significado |
|-------|-------------|
| `fabricante` / `modeloMaquina` | Fabricante e modelo genérico reportados pelo sistema operacional |
| `cpu` | Nome público do processador |
| `gpuPrincipal` | GPU dedicada selecionada automaticamente na máquina atual |
| `gpusDetectadas` | Todas as GPUs reportadas pelo sistema operacional/driver atual |
| `ramTotalGb` | RAM física total arredondada em GB |
| `sistemaOperacional` / `arquitetura` | Plataforma de execução, sem usuário, hostname, caminhos, IPs ou IDs de dispositivo |
| `hardwareColetadoAutomaticamente` | Indica se os valores foram coletados automaticamente do sistema local |

#### `resumo`

Métricas agregadas.

| Campo | Significado |
|-------|-------------|
| `totalEpisodiosTraduzidos` | Episódios processados pelo pipeline de tradução LLM |
| `totalLinhasTraduzidas` | Falas de legenda traduzidas |
| `tempoMedioPorLinhaMs` | Latência média de tradução por fala |
| `totalFalasReaproveitadasDoCache` | Falas resolvidas pelo cache persistente sem nova chamada ao LLM |
| `alucinacoesLlmPrevenidas` | Respostas de LLM rejeitadas pelas guardas anti-alucinação |
| `respostasTraducaoRejeitadas` | Tentativas inválidas rejeitadas antes da persistência |
| `falhasTraducaoRecuperadas` | Falas recuperadas por tentativa posterior validada |
| `fallbacksTraducaoMantidos` | Falas distintas ainda pendentes após esgotar tentativas |
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
| `falasItalicoRemovido` | Falas de que a regra do itálico tirou a tag (null = não medido) |
| `falasItalicoPreservado` | Falas de que a regra SE ABSTEVE: o itálico vem do `Style:` do cabeçalho |
| `quantidadeAvisos` | Contagem de avisos de qualidade, sem texto dos avisos |
| `registradoEm` | Timestamp UTC em ISO-8601 |

#### `operacoes[]`

Métricas genéricas por operação de pipeline: `tipo`, `tempoTotalMs`, `arquivosProcessados`, `itensDetectados`, `itensCorrigidos`, `registradoEm`.

Cobre remux, extração de legendas, revisões de lore/concordância, karaokê, renomeação de arquivos e auditorias.

### Privacidade E Anonimização

Este dataset não publica caminhos locais da máquina, nomes de usuário, hostnames, endereços IP, endereços MAC, números de série, identificadores de dispositivo, credenciais, tokens ou chaves de API.

**Trechos de legenda SÃO publicados, deliberadamente e só nas tabelas de aviso.** Os avisos do pipeline, em `kronos-telemetria-execucoes.jsonl` / `metrics/csv/kronos-avisos.csv` (diálogo) e em `kronos-karaoke-execucoes.jsonl` / `metrics/csv/kronos-karaoke-avisos.csv` (letra de música), citam a fala que provocou a falha — por exemplo uma linha mantida sem tradução porque o modelo corrompeu as tags ASS. Sem a fala, a falha não pode ser estudada nem reproduzida, que é a razão de publicar telemetria de tradução. As tabelas de karaokê citam **linha de letra de música** pelo mesmo motivo e sob a mesma regra: só a linha que falhou, nunca a letra inteira.

São trechos curtos de diagnóstico, vindos de legendas de fansub, publicados para pesquisa de modos de falha em tradução automática. Não constituem corpus traduzido e nenhum arquivo de legenda completo é redistribuído. Se você detém direitos sobre uma fala citada e quer removê-la, abra uma issue.

As métricas agregadas (`kronos-telemetria-dataset.json` e os demais CSVs) **não** contêm texto de legenda: ali os avisos viram apenas `quantidadeAvisos`.

Fora isso, os únicos identificadores públicos são nomes de obras/releases, ids de modelos LLM locais e metadados genéricos de hardware úteis para interpretar benchmarks.

### Geração

O dataset é gerado pelo painel de Telemetria do KRONOS CORE através do botão **Publicar Dataset**. O KRONOS sanitiza a telemetria acumulada, escreve `metrics/kronos-telemetria-dataset.json`, commita o snapshot e faz push para este repositório.

### Licença

[MIT](LICENSE) — uso livre com atribuição.
