# Bioinformática & Ômicas na Periodontite — Pipeline de Análise

Repositório para análise e visualização dos estudos sobre **bioinformática e ômicas aplicadas à periodontite** (genômica, proteômica, transcriptômica, metabolômica, resistômica, filogenia/fenética etc.), com base na planilha consolidada do Elicit.

## Estrutura sugerida

```
.
├─ data/
│  └─ periodontitis_bioinfo_omics_elicit_filled.xlsx   # coloque aqui seu .xlsx
├─ outputs/
│  ├─ figures/                                         # figuras (.png)
│  └─ tables/                                          # tabelas .csv + workbook .xlsx
└─ scripts/
   └─ analyze_periodontitis_bioinfo.R                  # script principal
```

> **Importante:** o script cria as pastas `outputs/figures` e `outputs/tables` automaticamente.

## Pré-requisitos

- **R ≥ 4.1**  
- Pacotes (instalados automaticamente, se faltarem):
  - `tidyverse`, `readxl`, `janitor`, `stringr`, `forcats`, `tidyr`,
    `ggpubr`, `ggalluvial`, `openxlsx`
  - **Opcional:** `ggbeeswarm` (se ausente, o script usa `geom_jitter()` como fallback)

### Dependências de SO (se necessário)

- Linux: `libxml2`, `libcurl`, `libssl`, `fontconfig` (dev)
- macOS: Xcode Command Line Tools
- Windows: RTools para compilações

## Dados de entrada

Coloque o arquivo **`periodontitis_bioinfo_omics_elicit_filled.xlsx`** em `data/`.  
O script espera colunas como (nomes já normalizados por `janitor::clean_names()`):

- Identificação: `study_id`, `year`, `first_author`, `title`, `focus`
- Ômicas/métodos: `omics_bioinfo_approaches`, `tool_type`, `platform`
- Amostras: `sample_types`
- Desempenho: `auc_min`, `auc_max`, `accuracy_pct`, `sensitivity_pct`, `specificity_pct`
- Validação/prontidão: `validation_status`, `clinical_readiness`
- Aplicações/novidades/limitações: `application`, `key_innovation`, `limitations`
- Meta: `full_text_retrieved`

### Variáveis derivadas criadas pelo script

- `auc_best` = `coalesce(auc_max, auc_min)`
- `accuracy_best` = `accuracy_pct`
- `omics_class` = classe mapeada por regex a partir de `omics_bioinfo_approaches`
- `uses_ml_dl` = flag (regex para ML/DL)
- `readiness` = fator ordenado a partir de `clinical_readiness`

## Como rodar

### Opção A — Padrão (pastas `data/` e `outputs/`)
```bash
Rscript scripts/analyze_periodontitis_bioinfo.R
```

### Opção B — Com parâmetros
```bash
Rscript scripts/analyze_periodontitis_bioinfo.R   --in  data/periodontitis_bioinfo_omics_elicit_filled.xlsx   --out outputs
```

## Saídas

### Figuras (`outputs/figures/`)

- `01_count_by_omics_class.png` — barras: nº de estudos por classe ômica  
- `02_count_by_sample_type.png` — barras: nº de estudos por tipo de amostra  
- `03_auc_heatmap_omics_by_sample.png` — heatmap: **AUC mediana** por ômica × amostra (com **N**)  
- `04_auc_beeswarm_by_omics.png` *ou* `04_auc_jitter_by_omics.png` — beeswarm/jitter da **AUC por ômica** (mediana marcada)  
- `05_accuracy_lollipop.png` — gráfico **lollipop** da **acurácia (%)** por estudo  
- `06_alluvial_omics_application_readiness.png` — **alluvial (Sankey)**: Ômica → Aplicação → Prontidão

### Tabelas CSV (`outputs/tables/`)

- `count_by_omics_class.csv`  
- `count_by_sample_type.csv`  
- `performance_by_omics_class.csv` (n, AUC n/mediana/mín/máx, ACC n/mediana/mín/máx)  
- `top10_auc_studies.csv`  
- `top10_accuracy_studies.csv`

### Workbook XLSX (`outputs/tables/periodontitis_bioinfo_outputs.xlsx`)

Abas: `raw`, `studies_curated`, `count_by_omics`, `count_by_sample`,  
`perf_by_omics`, `top10_auc`, `top10_accuracy`.

## Dicionário (aba `studies_curated`)

- **`omics_class`**:  
  `Metagenomics (shotgun)`, `Metagenomics (16S)`, `Metatranscriptomics`,  
  `Transcriptomics (single-cell)`, `Transcriptomics (bulk)`, `Proteomics`,  
  `Metabolomics`, `Genomics/Functional arrays`, `Other/Methodological`
- **`uses_ml_dl`**: `TRUE/FALSE` (busca por *machine learning*, *SVM*, *RF*, *CNN*, *deep learning*, *xgboost*, etc.)
- **`auc_best`**: melhor AUC disponível
- **`accuracy_best`**: alias de `accuracy_pct`
- **`readiness`**: `Conceptual` → `Potential` → `Feasible` → `Promising` → `High accuracy` → `Pilot` → `Unspecified`

## Reprodutibilidade

- O pipeline é determinístico.  
- Para registrar o ambiente:
  ```r
  sessionInfo()
  ```
  Salve a saída em `outputs/sessionInfo.txt` (opcional).

## Solução de problemas

- **Arquivo não encontrado**: verifique o caminho passado no `--in` e se o `.xlsx` está em `data/`.  
- **Erro de pacote `ggbeeswarm`**: instale com `install.packages("ggbeeswarm")` ou use o fallback (já automático).  
- **Rótulos do alluvial**: o script detecta `ggplot2 >= 3.4` e usa `after_stat(stratum)`; em versões antigas, usa `..stratum..`.  
- **AUC mediana = NA**: quando todos os valores na célula ômica×amostra são `NA`. O tile mostra apenas `n=` e as funções “safe” evitam `Inf`.

## Customização

- Ajuste `classify_omics()` para granularidades extras (ex.: separar *WGS* vs *shotgun*).  
- Edite o *binning* das aplicações em `app_group` (bloco do alluvial).  
- Personalize o tema em `theme_pub`.  
- Adicione novas figuras após a seção **05) FIGURAS**.

## Citação sugerida

> *Periodontitis bioinformatics & multi-omics analysis pipeline (R). GitHub: <seu-usuário>/<seu-repo>. Version: <tag/commit>. Ano.*

Ajuste ao estilo exigido (ABNT/Vancouver/APA).

## Licença

Sugestão: **MIT**. Inclua um arquivo `LICENSE` com o texto da licença.
