# Análise Multimodal de Biosinais para Detecção de Freezing of Gait (FoG)

**Equipe:**
* Danilo Vieira Bezerra
* Nataniel Marques Viana Neto
* Thiago Siqueira de Sousa

---

## Descrição do Projeto
Este projeto tem como objetivo analisar sinais biomédicos multimodais para detectar episódios de Freezing of Gait (FoG), um sintoma da Doença de Parkinson em que o paciente "trava" durante a caminhada. A proposta envolve trabalhar com sinais cerebrais, musculares e de movimento para identificar padrões fisiológicos associados ao FoG.

---

## Objetivos
* **Análise descritiva:** Exploração dos sinais e dos dados brutos.
* **Investigação Fisiológica:** Identificação de padrões associados ao FoG.
* **Extração de Características:** Seleção de variáveis relevantes para análise.
* **Dashboard:** Criação de ambiente para comparações de sinais.
* **Objetivo Futuro:** Classificação automática com Machine Learning.

---

## Dataset
O dataset contém dados de **11 pacientes** submetidos a experimentos controlados com indução de FoG. Os sinais foram coletados simultaneamente para garantir a multimodalidade.

### Tipos de Sinais
| Tipo | Descrição |
| :--- | :--- |
| EEG | Atividade elétrica cerebral |
| EMG | Atividade muscular |
| ECG | Atividade cardíaca |
| EOG | Movimento ocular |
| ACC/Gyro | Movimento corporal (aceleração e rotação) |
| SC | Condutância da pele |

---

## Sinais Raw e Filtered

### Dados Raw (Brutos)
Os dados Raw são armazenados em arquivos separados com formatos específicos:
* **.eeg:** Contém os dados brutos dos sinais (EEG, EMG, ECG e EOG).
* **.vhdr:** Arquivo de cabeçalho (manual) que explica a estrutura dos dados.
* **.vmrk:** Marcadores de eventos ocorridos durante o experimento.
* **.csv:** Dados de movimento dos pacientes.

**Especificações Técnicas:**
* **Sistema MOVE:** Utilizado para sinais EEG/EMG/ECG/EOG com frequência de amostragem de **1000 Hz**.
* **Sensores MPU6050 e LM324:** Utilizados para movimentos com frequência de amostragem de **500 Hz**.
* **Observação:** Os sinais Raw contêm ruídos e frequências distintas, o que dificulta o uso direto sem tratamento.

### Dados Filtered (Filtrados)
São sinais sincronizados, limpos e organizados em uma tabela única por tarefa (*task*). 
* Exemplo: O arquivo `task_n.txt` contém as linhas representando o tempo e as colunas representando as medições.
* Colunas com valor zero indicam que aquele sensor específico não foi utilizado naquela medição.

---

## Detalhamento das Colunas (Filtered)

| Colunas | Conteúdo |
| :--- | :--- |
| 1 | Tempo |
| 2 – 26 | EEG (25 canais) |
| 27 – 31 | EMG / ECG / EOG |
| 32 – 59 | Movimento (ACC + Gyro + SC) |
| 60 | Label (FoG ou não) |

### Descrição Detalhada:
* **Coluna 1 (Tempo):** Representa o tempo decorrido desde o início. Exemplo: se a taxa de amostragem for de 500 Hz e o experimento durar 10 segundos, a tabela terá cerca de 5000 linhas de amostras.
* **Colunas 2-26:** Correspondem aos 25 canais de sinais cerebrais (EEG).
* **Colunas 27-31 (Sinais do Corpo):** Variável por paciente, contendo:
    * **EMG:** Cada canal representa um músculo específico. **TA** (Tibialis anterior - frente da perna) e **GS** (Gastrocnemius - panturrilha), precedidos por **R** (direita) ou **L** (esquerda).
    * **ECG:** Sinais cardíacos.
    * **EOG:** Movimento ocular (**IO - Electrooculogram**), fixado sempre na coluna 29.
* **Colunas 32-59 (Movimento):** Dados obtidos por sensores na perna esquerda, perna direita, cintura e braço. Cada parte possui 7 colunas (acc_x, acc_y, acc_z, gyro_x, gyro_y, gyro_z e SC). A coluna **SC** (condutância de pele) é considerada apenas para o braço e ignorada nas demais partes.
* **Coluna 60 (Label):** Classificação binária onde **1** indica presença de FoG e **0** indica ausência.

---

## Pré-processamento

* **EEG:** Remoção de artefatos com ICA. Referência nos canais TP9 e TP10 (removidos do dataset final).
* **EMG:** Filtro de banda entre 10–500 Hz.
* **ACC:** Filtro passa-baixa até 16 Hz (limite de frequência do sinal).
* **Notch Filter:** Remoção de ruído elétrico em 50 Hz.

---

## Objetivo de engenharia: mimetizar o pré-processamento oficial

Além de organizar e analisar os dados, este repositório implementa um **pipeline próprio** (notebooks de **pré-processamento 1 a 4**) que parte de `data/raw/original/` e tenta **aproximar-se do pré-processamento oficial** já realizado pelos autores do dataset.

No material que acompanha os dados filtrados, o estudo original disponibiliza **código** sob `data/raw/filtered_txt/Code/` (por exemplo o script `Data Processing.py`), que descreve as regras e etapas usadas para gerar as tabelas limpas em `data/raw/filtered_txt/Filtered Data/` a partir dos registros em `data/raw/original/`. Em outras palavras: **original → (código oficial) → Filtered Data**.

O que fazemos aqui é **reimplementar e documentar em Jupyter** um caminho **original → (nosso pipeline) → silver**, com escolhas explícitas (reamostragem a 500 Hz, filtros Butterworth, Z-score por paciente, *clip* de outliers, etc.), e usamos os arquivos já exportados em **Filtered Data** (e a bronze `task_*.parquet` derivada deles) como **referência de validação** — em especial no `pre_processamento_4.ipynb`, que compara nosso resultado com os `task_*.parquet` da camada bronze filtrada. **Igualdade byte a byte não é obrigatória** (ambiente numérico, ordem de operações e ICA podem divergir), mas a **proximidade** dos sinais e a **coerência temporal** indicam se estamos alinhados à intenção metodológica do pré-processamento oficial.

---

## Os oito notebooks (`notebooks/`)

Todos os cadernos estão na pasta `notebooks/`. A tabela resume a **função** de cada um; abaixo há um parágrafo por arquivo.

| # | Arquivo | Função principal |
|:-:|---------|------------------|
| 1 | `bronze_bruta.ipynb` | Ingestão dos dados **raw** (BrainVision + CSV) → tabela única `full_table.parquet`. |
| 2 | `bronze_filtrada.ipynb` | Ingestão dos `.txt` de **Filtered Data** → `task_*.parquet` por paciente/tarefa (referência tabular). |
| 3 | `analise_brutos.ipynb` | Análise exploratória e QC sobre `full_table.parquet`. |
| 4 | `analise_filtrados.ipynb` | Análise exploratória e QC sobre os `task_*.parquet` da bronze filtrada. |
| 5 | `pre_processamento_1.ipynb` | Parte 1: caminhos, constantes, funções compartilhadas e **validação estrutural** do raw em bronze. |
| 6 | `pre_processamento_2.ipynb` | Parte 2: **reamostragem e alinhamento temporal** (ex.: 500 Hz) → `resampled.parquet`. |
| 7 | `pre_processamento_3.ipynb` | Parte 3: **filtragem** (bandas, notch 50 Hz) → `filtered.parquet`. |
| 8 | `pre_processamento_4.ipynb` | Parte 4: **flags de sensores**, NaNs, **normalização** (Z-score), *clip*, saída `normalized.parquet` e **relatório de comparação** com a referência oficial. |

### 1. `bronze_bruta.ipynb`

Lê `data/raw/original/`, integra EEG/bio (MNE) com IMU/CSV, padroniza nomes de colunas, trata sensores ausentes e grava **`data/bronze/full_table.parquet`**. É a base tabular única a partir da qual roda o pipeline silver.

### 2. `bronze_filtrada.ipynb`

Lê os `.txt` produzidos pelo fluxo oficial (pasta **Filtered Data**), aplica o dicionário de colunas e particiona em **`data/bronze/filtered_parquet/<paciente>/task_*.parquet`**. Esses arquivos materializam o que o código em `filtered_txt/Code` pretendia como saída “oficial” para comparação.

### 3. `analise_brutos.ipynb`

Não altera o pipeline: **explora** `full_table.parquet` (dimensões, faltantes, distribuições, gráficos) para entender qualidade e heterogeneidade antes e depois de decisões de modelagem.

### 4. `analise_filtrados.ipynb`

**Explora** os Parquets da bronze filtrada (tarefas por paciente), visualiza canais, detecta colunas vazias e examina o rótulo FoG — útil para calibrar expectativas do que a referência contém.

### 5. `pre_processamento_1.ipynb`

Centraliza **imports, paths, constantes e funções** usadas nas partes 2–4 e executa **checagens de sanidade** em `full_table.parquet` (e listagem dos `task_*.parquet` de referência). Deve rodar antes das demais partes do pré-processamento.

### 6. `pre_processamento_2.ipynb`

Corrige o problema de **taxas de amostragem diferentes** na mesma tabela bronze: gera **`data/silver/resampled.parquet`** com grade temporal uniforme (alvo típico 500 Hz), condição necessária para filtragem estável.

### 7. `pre_processamento_3.ipynb`

Aplica filtros digitais (ex.: Butterworth, notch 50 Hz, bandas por modalidade) sobre o resampled e grava **`data/silver/filtered.parquet`**, alinhado às bandas descritas na documentação do projeto e à lógica do pré-processamento de referência.

### 8. `pre_processamento_4.ipynb`

Etapa final: documenta sensores faltantes (`sensor_flags.parquet`), trata NaNs, normaliza (Z-score por paciente, com *clip* de outliers), salva **`data/silver/normalized.parquet`** e **`normalization_params.parquet`**, e gera **`comparacao_report.txt`** confrontando o resultado com os `task_*.parquet` da referência — o fechamento do objetivo de **mimetizar** o fluxo oficial.

**Ordem sugerida:** `bronze_bruta` e/ou `bronze_filtrada` → opcionalmente `analise_*` → `pre_processamento_1` → `2` → `3` → `4`.

---

## Organização do Projeto

```text
TI093-biosignals-acquisition/
├── data/
│   └── raw/
│       ├── README.md              # Links Mendeley e onde colocar original/ e filtered_txt/
│       ├── original/              # Dados BrainVision + CSV (não versionados)
│       └── filtered_txt/
│           ├── Code/              # Código oficial (ex.: Data Processing.py)
│           └── Filtered Data/     # .txt por tarefa gerados a partir de original/
├── notebooks/                     # Os oito notebooks descritos acima
├── .gitignore
├── README.md
└── requirements.txt
```

Pastas `data/bronze/`, `data/silver/` e arquivos `.parquet` são gerados localmente ao executar os notebooks; em geral estão no `.gitignore`.

---

## Dependências

Arquivo `requirements.txt` na raiz (numpy, pandas, matplotlib, mne, jupyter, pyarrow). Recomenda-se ambiente virtual e `pip install -r requirements.txt` antes de abrir os notebooks.
