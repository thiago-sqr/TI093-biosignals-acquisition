# Análise Multimodal de Biosinais para Detecção de Freezing of Gait (FoG)

**Equipe:**
- Danilo Vieira Bezerra
- Nataniel Marques Viana Neto
- Thiago Siqueira de Sousa

---

## O que é este projeto e por que ele existe

Imagine uma pessoa com Doença de Parkinson caminhando por um corredor. De repente, sem aviso, os pés dela param de se mover, como se estivessem grudados no chão, enquanto o resto do corpo ainda tenta andar. Isso se chama **Freezing of Gait**, ou simplesmente **FoG** (congelamento da marcha). O episódio pode durar alguns segundos, mas nesse tempo a pessoa fica vulnerável a quedas, acidentes e situações de risco.

O objetivo deste projeto é criar um pipeline capaz de **detectar automaticamente os episódios de FoG** a partir de sinais biológicos coletados durante a caminhada de pacientes reais. Esses sinais incluem atividade cerebral (EEG), muscular (EMG), cardíaca (ECG), de movimento (acelerômetros e giroscópios) e outros sensores. A ideia central é que, antes de um episódio de FoG, o corpo e o cérebro do paciente emitem padrões fisiológicos que um sistema inteligente pode aprender a reconhecer.

O projeto é dividido em duas grandes fases:

1. **Fase A: Pré-processamento** — Os dados brutos coletados pelos sensores estão cheios de ruído, artefatos e inconsistências. Esta fase limpa, organiza e valida os sinais.
2. **Fase B: Extração e Seleção de Características** — A partir dos sinais limpos, calculam-se centenas de variáveis numéricas que descrevem o comportamento do sinal em cada momento. Depois, as variáveis mais úteis são selecionadas para alimentar um classificador automático.

---

## O dataset

O dataset contém dados de **12 pacientes** com Doença de Parkinson submetidos a experimentos controlados com indução de FoG. Os sinais foram coletados simultaneamente por sensores de diferentes tipos:

| Tipo de sinal | O que mede |
| :--- | :--- |
| **EEG** (Eletroencefalograma) | Atividade elétrica do cérebro, captada por eletrodos no couro cabeludo |
| **EMG** (Eletromiograma) | Atividade elétrica dos músculos, especialmente das pernas |
| **ECG** (Eletrocardiograma) | Atividade elétrica do coração |
| **EOG** (Eletrooculograma) | Movimentos dos olhos |
| **ACC** (Acelerômetro) | Aceleração linear de partes do corpo: pernas, cintura, braço |
| **GYRO** (Giroscópio) | Velocidade angular de partes do corpo |
| **SC/NC** (Condutância da pele) | Nível de suor na pele, ligado ao sistema nervoso autônomo |

Os sensores EEG, EMG, ECG e EOG foram coletados com frequência de **1000 Hz** (1000 medições por segundo). Os sensores de movimento foram coletados a **500 Hz**. Isso significa que, a cada segundo, o sistema estava registrando dezenas de milhares de valores numéricos ao mesmo tempo.

Cada amostra tem um **rótulo binário** associado: `0` para marcha normal e `1` para episódio de FoG. Esses rótulos foram anotados manualmente por especialistas a partir de vídeos gravados durante os experimentos.

---

## Como os dados estão organizados

```text
TI093-biosignals-acquisition/
├── data/
│   ├── raw/
│   │   ├── original/              ← Dados brutos: arquivos .eeg, .vhdr, .vmrk e .csv
│   │   └── filtered_txt/
│   │       ├── Code/              ← Código oficial dos autores do dataset
│   │       └── Filtered Data/     ← Dados já processados pelos autores, em .txt
│   ├── bronze/
│   │   ├── full_table.parquet     ← Tabela única unificando todos os sinais brutos
│   │   └── filtered_parquet/      ← Versão tabular dos .txt oficiais, particionada por paciente
│   └── silver/                    ← Produtos do pipeline de pré-processamento e extração
├── notebooks/                     ← Todos os cadernos Jupyter do projeto
├── requirements.txt
└── README.md
```

As pastas `bronze/` e `silver/` não estão versionadas no Git (estão no `.gitignore`) porque são arquivos grandes gerados automaticamente ao rodar os notebooks. Para recriar tudo do zero, execute os notebooks na ordem indicada abaixo.

---

## O que é a camada Bronze e o que é a camada Silver

Pense nessas camadas como estágios de refinamento, como acontece na mineração de ouro:

- **Bronze:** Dados ainda no formato "como vieram do mundo real", apenas convertidos para uma estrutura tabular legível. Não há limpeza de sinal, mas já existe uma estrutura consistente.
- **Silver:** Dados que passaram por todo o processo de limpeza, filtragem, normalização, qualidade e extração de características. É aqui que vivem os arquivos prontos para o classificador.

---

## Arquivos da camada Silver — o que cada um significa

Esta seção existe porque a pasta `data/silver/` contém muitos arquivos e pode ser difícil entender o papel de cada um sem ter executado os notebooks. A tabela abaixo organiza os arquivos na ordem em que são gerados:

| Arquivo | Gerado por | O que contém |
| :--- | :--- | :--- |
| `resampled.parquet` | PP2 | Sinais brutos reamostrados para 500 Hz — grade temporal uniforme |
| `filtered.parquet` | PP3 | Sinais após filtragem digital (passa-banda, notch 50 Hz) |
| `sensor_flags.parquet` | PP4 | Tabela indicando quais sensores estavam ausentes em cada paciente |
| `normalized.parquet` | PP4 | Sinais normalizados por Z-score, prontos para análise |
| `normalization_params.parquet` | PP4 | Média e desvio padrão usados na normalização, por paciente e canal |
| `comparacao_report.txt` | PP4 | Relatório comparando nosso resultado com o pré-processamento oficial |
| `sqi_stats.parquet` | PP5 | Métricas de qualidade (kurtose, SNR, etc.) por janela e canal |
| `sqi_mask.parquet` | PP5 | Máscara indicando quais janelas de 5s são válidas para extração |
| `sqi_report.txt` | PP5 | Relatório textual com taxa de aprovação por paciente e canal |
| `pp6_report.txt` | PP6 | Relatório estatístico: normalidade, homocedasticidade, correlações |
| `pp4_validacao_estatistica.txt` | PP6 | Complemento da validação estatística do PP4 |
| `segments.npz` | Ext1 | Array 3D: 24.564 janelas × 500 amostras × 54 canais |
| `windows_metadata.parquet` | Ext1 | Metadados por janela: paciente, task, posição temporal, label |
| `segmentation_report.txt` | Ext1 | Resumo do janelamento: contagens por paciente e label |
| `features_raw.parquet` | Ext2 | 24.564 janelas × 1.251 features brutas (tempo, frequência, DWT, não-linear) |
| `features_engineered.parquet` | Ext3 | 24.564 × 1.370 features após enriquecimento (razões, deltas, baseline) |
| `feature_selection_rankings.parquet` | Ext5 | Ranking de todas as 1.370 features por ANOVA, MI, ReliefF, LASSO, Cohen's d |
| `features_selecionadas.parquet` | Ext5 | 24.564 × 156 features selecionadas pelo consenso dos métodos |
| `features_pca.parquet` | Ext4 | 24.564 × 447 componentes principais (PCA, 95% da variância) |
| `pca_loadings.parquet` | Ext4 | Pesos de cada feature original em cada componente principal |
| `pca_scaler.joblib` | Ext4 | Objeto de normalização do PCA, necessário para aplicar em dados novos |
| `pca_model.joblib` | Ext4 | Modelo PCA treinado, necessário para aplicar em dados novos |
| `dataset_final.parquet` | Ext6 | 24.564 × 125 features finais, após remoção de multicolinearidade (VIF) |
| `dataset_manifest.json` | Ext6 | Resumo numérico do dataset final: contagens, ratio, correlação média |

Além desses, a pasta contém imagens PNG geradas pelos notebooks para visualização: histogramas, scree plot do PCA, curvas de densidade, heatmap de correlações, gráfico de balanceamento e outros.

---

## Como rodar o projeto

Execute os notebooks nesta ordem:

```
1.  bronze_bruta.ipynb                       ← Ingestão dos dados originais
2.  bronze_filtrada.ipynb                    ← Ingestão dos dados filtrados oficiais
[Opcional: analise_brutos.ipynb e analise_filtrados.ipynb para exploração]
3.  pre_processamento_1.ipynb                ← Funções e validação estrutural
4.  pre_processamento_2.ipynb                ← Reamostragem
5.  pre_processamento_3.ipynb                ← Filtragem
6.  pre_processamento_4.ipynb                ← Normalização e comparação com referência
7.  pre_processamento_5_sqi.ipynb            ← Avaliação de qualidade do sinal
8.  pre_processamento_6_estatistica.ipynb    ← Estatística descritiva
9.  extração_1_segmentação.ipynb             ← Janelamento dos sinais
10. extração_2_features.ipynb                ← Extração das características brutas
11. extração_3_feature_engineering.ipynb     ← Enriquecimento das características
12. extração_4_reducao_dimensionalidade.ipynb  ← Redução de dimensionalidade (PCA)
13. extração_5_selecao_atributos.ipynb       ← Seleção de atributos
14. extração_6_validacao_final.ipynb         ← Validação e dataset final
```

---

# Fase A: Pré-processamento

O objetivo desta fase é transformar os dados brutos e ruidosos em sinais limpos, validados e prontos para extração de características.

---

## Notebook 1 — `bronze_bruta.ipynb`

**Objetivo:** Ler os arquivos originais dos sensores e juntá-los em uma única tabela.

**Entrada:** `data/raw/original/` — arquivos `.eeg`, `.vhdr`, `.vmrk` (sinais EEG/bio) e `.csv` (sinais de movimento).

**Saída:** `data/bronze/full_table.parquet` — uma tabela com todas as medições de todos os pacientes em um único arquivo estruturado.

O problema central aqui é que os dados vêm de dois sistemas diferentes com frequências de amostragem diferentes: os sensores cerebrais e biológicos a 1000 Hz e os sensores de movimento a 500 Hz. Antes mesmo de qualquer limpeza, é preciso juntar tudo numa estrutura comum. Este notebook usa a biblioteca **MNE** (especializada em sinais cerebrais) para ler os arquivos BrainVision e fazer o alinhamento inicial com os arquivos CSV de movimento.

**Limitação conhecida:** Os canais EEG-FZ, EEG-CZ, EEG-PZ e IO ficaram com todos os valores como NaN neste notebook. Eles existem no dataset original, mas não foram capturados corretamente durante a ingestão. Essa limitação é documentada no PP6 e se propaga por todo o pipeline da camada silver.

---

## Notebook 2 — `bronze_filtrada.ipynb`

**Objetivo:** Ler os dados já processados pelos autores originais do dataset e organizá-los no mesmo formato tabular.

**Entrada:** `data/raw/filtered_txt/Filtered Data/` — arquivos `.txt` com os sinais pré-filtrados.

**Saída:** `data/bronze/filtered_parquet/<paciente>/task_*.parquet` — um arquivo por tarefa por paciente.

Esses arquivos são usados como **referência de validação**. Ao final do pré-processamento 4, comparamos o resultado do nosso pipeline com esses arquivos para verificar se estamos próximos da intenção metodológica dos autores.

---

## Notebooks de análise exploratória — `analise_brutos.ipynb` e `analise_filtrados.ipynb`

Esses dois notebooks não modificam nenhum dado. Eles apenas exploram visualmente os arquivos bronze para entender a qualidade, as distribuições e as características dos sinais antes de qualquer processamento. São úteis para quem quer entender os dados antes de mergulhar no pipeline.

---

## Notebook 3 — `pre_processamento_1.ipynb`

**Objetivo:** Centralizar todas as funções, constantes e caminhos de arquivo que serão usados nas etapas seguintes.

**Entrada:** `data/bronze/full_table.parquet`

**Saída:** Nenhum arquivo novo — apenas funções e variáveis disponíveis para os próximos notebooks.

Este notebook também faz **checagens de sanidade**: verifica se os arquivos bronze existem, se têm o número esperado de colunas e pacientes, e se os dados têm dimensões coerentes. É o ponto de partida obrigatório antes de qualquer pré-processamento.

---

## Notebook 4 — `pre_processamento_2.ipynb` — Reamostragem

**Objetivo:** Converter todos os sinais para a mesma frequência de amostragem (500 Hz).

**Entrada:** `data/bronze/full_table.parquet`

**Saída:** `data/silver/resampled.parquet`

**Por que isso é necessário?**

Imagine dois relógios marcando tempos diferentes: um bate 1000 vezes por segundo e outro bate 500 vezes. Para comparar os sinais capturados por esses relógios, é preciso colocá-los na mesma grade temporal. Se misturarmos sinais a taxas diferentes sem ajuste, os filtros digitais vão distorcer o sinal e os algoritmos de aprendizado vão receber dados temporalmente inconsistentes.

**O que é reamostragem?**

É o processo de calcular os valores de um sinal em novos instantes de tempo, usando interpolação. Para os sinais a 1000 Hz, calculamos os valores nos instantes de 500 Hz, descartando as amostras intermediárias de forma controlada (com filtro antialiasing para evitar distorções).

**Resultado esperado:** Todos os 54 canais disponíveis passam a ter exatamente 500 amostras por segundo, com a mesma grade temporal.

---

## Notebook 5 — `pre_processamento_3.ipynb` — Filtragem

**Objetivo:** Remover ruídos e frequências irrelevantes de cada tipo de sinal.

**Entrada:** `data/silver/resampled.parquet`

**Saída:** `data/silver/filtered.parquet`

**O que é filtragem de sinal?**

Pense em um filtro de café: ele deixa passar o líquido e retém o pó. Um filtro digital faz a mesma coisa, mas para frequências: ele deixa passar apenas as frequências de interesse e bloqueia as indesejadas.

Os filtros aplicados foram:

- **EEG:** Filtro passa-banda de 0,5 a 40 Hz. O cérebro humano gera ondas elétricas nessa faixa. Abaixo de 0,5 Hz há deriva lenta dos eletrodos. Acima de 40 Hz há artefatos musculares.
- **EMG:** Filtro passa-banda de 10 a 500 Hz. A atividade muscular útil está nessa faixa.
- **ACC:** Filtro passa-baixa até 16 Hz. Movimentos do corpo humano não ultrapassam essa frequência.
- **Notch 50 Hz (todos os canais):** Remove o ruído da rede elétrica. Em qualquer ambiente com tomadas, os cabos dos sensores captam uma interferência de 50 Hz que nada tem a ver com o paciente. O filtro notch é cirúrgico: remove especificamente 50 Hz sem afetar as frequências vizinhas.

**Por que filtros Butterworth?**

O filtro Butterworth tem resposta em frequência maximamente plana na banda de passagem, sem ondulações. Isso significa que as frequências que queremos preservar chegam ao outro lado com amplitude praticamente intacta, enquanto as indesejadas são atenuadas suavemente.

---

## Notebook 6 — `pre_processamento_4.ipynb` — Normalização e Validação

**Objetivo:** Normalizar os sinais por paciente e comparar com o pré-processamento oficial.

**Entrada:** `data/silver/filtered.parquet`

**Saída:**
- `data/silver/normalized.parquet`
- `data/silver/normalization_params.parquet`
- `data/silver/sensor_flags.parquet`
- `data/silver/comparacao_report.txt`

**Por que normalizar?**

Cada paciente tem características fisiológicas únicas. O sinal EEG do paciente A pode ter amplitude natural de 50 microvolts, enquanto o do paciente B pode ser de 20 microvolts. Se misturarmos os dados sem normalizar, o classificador vai aprender a diferenciar pacientes pela amplitude absoluta, e não pelos padrões de FoG que queremos detectar.

**O que é normalização Z-score?**

Para cada canal de cada paciente, subtraímos a média e dividimos pelo desvio padrão. O resultado é um sinal com média zero e desvio padrão um:

```
valor_normalizado = (valor_original - média_do_canal) / desvio_padrão_do_canal
```

Além disso, aplicamos um **clip de outliers**: valores que ultrapassam 5 desvios padrões são limitados a esse valor máximo, para evitar que artefatos pontuais dominem a escala.

**Resultado da comparação com a referência oficial:**

O relatório `comparacao_report.txt` compara nosso pipeline canal a canal com os dados pré-processados pelos autores originais. Os resultados para EEG e EMG foram muito bons: correlação entre 0,75 e 0,99 para a maioria dos pacientes e canais, com frequência de pico coincidindo com a referência. Os canais de acelerômetro (especialmente LShank-ACCX) apresentaram maior divergência em alguns pacientes, provavelmente por diferenças na ordem de filtragem e no tratamento do sincronismo de referência.

---

## Notebook 7 — `pre_processamento_5_sqi.ipynb` — Avaliação de Qualidade do Sinal (SQI)

**Objetivo:** Identificar e descartar janelas de tempo em que o sinal estava contaminado por artefatos, mesmo após a filtragem.

**Entrada:** `data/silver/filtered.parquet`

**Saída:**
- `data/silver/sqi_stats.parquet`
- `data/silver/sqi_mask.parquet`
- `data/silver/sqi_report.txt`

**O que é SQI (Signal Quality Index)?**

Dado tratado não é necessariamente dado confiável. Um sinal pode ter passado pelos filtros e mesmo assim conter um momento em que o eletrodo soltou, o paciente se mexeu bruscamente, ou alguém tocou no cabo do sensor. Nesses momentos, o sinal não representa fisiologia: representa ruído. Se incluirmos esses trechos na extração de características, o classificador vai aprender padrões falsos.

O SQI divide o sinal em **janelas de 5 segundos** e avalia a qualidade de cada janela com quatro métricas:

**1. Kurtose**

A kurtose mede o "peso das caudas" de uma distribuição. Se você fizer um histograma da amplitude do sinal EEG em 5 segundos, ele deveria ter formato de sino (gaussiano), com a maioria dos valores próximos de zero e poucas ocorrências nas extremidades. Quando um eletrodo solta por um instante e gera um spike enorme, esse spike cria uma barra isolada muito longe do centro no histograma. A kurtose detecta isso: valores acima de 10 indicam spikes atípicos.

**2. Razão pico a pico / desvio padrão**

A amplitude máxima menos a mínima, dividida pelo desvio padrão. Um sinal limpo tem essa razão entre 4 e 6. Valores acima de 10 indicam que houve um spike extremo.

**3. SNR proxy (Signal-to-Noise Ratio)**

Mede se o espectro de frequências do sinal tem estrutura (picos em frequências específicas, como as ondas cerebrais) ou é completamente plano como ruído branco. Um sinal biológico sempre tem alguma estrutura espectral. Um eletrodo desconectado gera ruído sem estrutura. Valores abaixo de 3 dB indicam sinal suspeito.

**4. Entropia Espectral**

Complementa o SNR: alta entropia (próxima de 1) significa espectro plano, sugerindo ausência de sinal útil.

**Critérios de rejeição:**

- Kurtose > 10
- Razão pico a pico / std > 10
- SNR proxy < 3 dB

Uma janela é rejeitada se qualquer canal crítico (EEG-C3, EEG-FP1, EMG-RTA) violar qualquer um desses critérios.

**Resultados obtidos:**

Das 769 janelas avaliadas, **651 foram aprovadas (84,7%)**. Isso significa que descartamos 15,3% dos dados por qualidade insuficiente. Os canais mais problemáticos foram os giroscópios: o RShank-GYRO-X teve kurtose mediana de 10,69 e 59,4% das janelas rejeitadas. Isso é esperado: durante a marcha, os giroscópios nas pernas registram movimentos bruscos com grande variação de amplitude.

Por paciente, a taxa de aprovação variou de 53,4% (paciente 008) a 100% (pacientes 005 e 012). O que esse resultado indica: o paciente 008 apresentou mais artefatos de movimento do que os demais, possivelmente por características individuais da marcha. Os resultados por paciente são:

| Paciente | Janelas aprovadas | Taxa |
| :--- | :--- | :--- |
| 001 | 38/44 | 86,4% |
| 002 | 78/94 | 83,0% |
| 003 | 93/104 | 89,4% |
| 004 | 41/44 | 93,2% |
| 005 | 36/36 | 100,0% |
| 006 | 38/43 | 88,4% |
| 007 | 58/65 | 89,2% |
| 008 | 39/73 | 53,4% |
| 009 | 36/54 | 66,7% |
| 010 | 77/86 | 89,5% |
| 011 | 57/66 | 86,4% |
| 012 | 60/60 | 100,0% |

---

## Notebook 8 — `pre_processamento_6_estatistica.ipynb` — Análise Estatística dos Sinais

**Objetivo:** Caracterizar estatisticamente os sinais aprovados pelo SQI para documentar as propriedades do dataset antes da extração de características.

**Entrada:** `data/silver/normalized.parquet` + `data/silver/sqi_mask.parquet`

**Saída:** `data/silver/pp6_report.txt` + imagens PNG

**Análises realizadas:**

**1. Estatística descritiva**

Calcula média, desvio padrão, assimetria (skewness) e kurtose de Fisher para todos os 54 canais nas 651 janelas aprovadas (1.627.500 amostras no total). A kurtose de Fisher tem valor zero para uma distribuição gaussiana perfeita.

**2. Testes de normalidade: Shapiro-Wilk e Kolmogorov-Smirnov**

O teste de Shapiro-Wilk (SW) testa se uma amostra vem de uma distribuição normal. A hipótese nula H₀ é "os dados são normais": se o p-valor for maior que 0,05, não rejeitamos essa hipótese. O teste de Kolmogorov-Smirnov (KS) compara a função de distribuição acumulada empírica dos dados com uma gaussiana teórica.

**O que se esperava:** Seria desejável que os sinais fossem aproximadamente normais, pois isso facilitaria o uso de métodos estatísticos paramétricos. No entanto, sinais biológicos raramente seguem distribuição normal perfeita.

**O que foi obtido:** **0 de 54 canais** passaram em qualquer dos dois testes de normalidade. Todos os sinais são não-gaussianos. Isso não é um problema em si, mas informa que métodos que assumem normalidade (como Análise Discriminante Linear clássica) podem não ser os mais adequados. Classificadores como Random Forest, XGBoost e SVM com kernel RBF são mais robustos a essa realidade.

**3. Homocedasticidade — Teste de Levene/Brown-Forsythe**

Testa se a variância de cada canal é a mesma entre todos os 12 pacientes. A hipótese nula H₀ é "as variâncias são iguais entre grupos".

**O que se esperava:** Alguma heterogeneidade, já que diferentes pacientes têm características fisiológicas distintas.

**O que foi obtido:** **0 de 54 canais** apresentaram variâncias homogêneas entre pacientes. Isso confirma que os pacientes são muito diferentes entre si, o que reforça a necessidade da normalização Z-score individual (PP4) e da validação cruzada LOSO (Leave-One-Subject-Out) no módulo de classificação.

**O que isso significa para o projeto:** A conclusão principal é que o modelo de classificação não pode assumir que um paciente novo vai ter as mesmas distribuições de sinal que os pacientes de treino. Isso torna o problema mais difícil, mas também mais realista para aplicação clínica.

**4. Correlação de Pearson entre canais**

Calculada sobre as médias de cada canal por janela (651 pontos), capturando a covariância entre modalidades ao longo do experimento. Os canais EEG-FZ, EEG-CZ, EEG-PZ e IO foram excluídos por terem variância zero.

---

# Fase B: Extração de Características

A partir daqui começa o trabalho central da disciplina de Reconhecimento de Padrões. Em vez de alimentar o classificador com os sinais brutos (500 amostras por segundo por 54 canais = 27.000 valores por segundo), extraímos **características numéricas** que comprimem a informação relevante em poucas dezenas de números por janela de tempo.

---

## Notebook 9 — `extração_1_segmentação.ipynb` — Janelamento dos Sinais

**Objetivo:** Dividir os sinais contínuos em janelas de tempo fixas e associar cada janela a um rótulo (normal ou FoG).

**Entrada:** `data/bronze/filtered_parquet/<paciente>/task_*.parquet` (dados pré-filtrados dos autores originais, mais confiáveis para a fase de extração)

**Saída:**
- `data/silver/segments.npz` — array tridimensional com todas as janelas
- `data/silver/windows_metadata.parquet` — metadados por janela
- `data/silver/segmentation_report.txt` — resumo do janelamento

**O que é janelamento?**

Pense no sinal de cada canal como uma fita de áudio infinita. Para extrair características, precisamos pegar pedaços dessa fita, analisar cada pedaço separadamente e atribuir a ele um rótulo. Esse corte em pedaços é o janelamento.

**Parâmetros escolhidos:**

- **Tamanho da janela: 1 segundo (500 amostras a 500 Hz).** Por que 1 segundo? A banda delta do EEG (1 a 4 Hz) requer pelo menos 1 ciclo completo para ser mensurada. Janelas menores não capturam as ondas cerebrais mais lentas. Janelas maiores de 2 segundos ou mais diluem o onset do FoG, tornando difícil detectar o início preciso do episódio.

- **Sobreposição de 50%:** Cada nova janela começa 250 amostras (0,5 segundos) depois da anterior. Ao invés de cortar a fita em pedaços que não se tocam, os pedaços se sobrepõem na metade. Isso aumenta o número de janelas disponíveis para treinamento e garante que eventos que ocorrem na transição entre janelas sejam capturados por pelo menos uma delas.

- **Rótulo por voto majoritário:** Se dentro de uma janela de 1 segundo, 300 amostras são normais e 200 são FoG, o rótulo atribuído à janela é 0 (normal), pois a maioria das amostras é normal. Janelas de transição que cruzam a fronteira entre tarefas diferentes são descartadas.

- **Filtro SQI inline:** Os mesmos critérios de qualidade do PP5 são aplicados canal a canal durante o janelamento, nos canais críticos EEG-C3, EEG-FP1 e EMG-RTA.

**Resultados obtidos:**

- **Total de janelas aprovadas:** 24.564
- **Label 0 (marcha normal):** 14.386 janelas (58,6%)
- **Label 1 (FoG):** 10.178 janelas (41,4%)
- **Forma do array 3D:** (24.564, 500, 54) — 24.564 janelas, cada uma com 500 amostras de 54 canais
- **Desvio padrão intra-janela (mediana):** 0,79 — ligeiramente abaixo de 1,0 porque o Z-score é calculado sobre todo o paciente, e janelas com marcha muito estável têm amplitudes naturalmente menores
- **Variância inter-janela (mediana):** 0,15 — indica variabilidade moderada entre janelas, o que é bom para o aprendizado

Por paciente, o número de janelas varia bastante. O paciente 003 tem 4.133 janelas (a maioria sendo FoG), enquanto o paciente 005 tem apenas 608 (todas normais). Essa variabilidade é uma característica real do dataset: alguns pacientes tiveram mais episódios de FoG do que outros durante os experimentos.

| Paciente | Total | Normal (0) | FoG (1) |
| :--- | :--- | :--- | :--- |
| 001 | 1.502 | 1.004 | 498 |
| 002 | 1.070 | 1.065 | 5 |
| 003 | 4.133 | 672 | 3.461 |
| 004 | 1.257 | 1.004 | 253 |
| 005 | 608 | 608 | 0 |
| 006 | 2.116 | 1.426 | 690 |
| 007 | 1.368 | 732 | 636 |
| 008 | 3.375 | 1.802 | 1.573 |
| 009 | 1.578 | 1.382 | 196 |
| 010 | 3.203 | 1.710 | 1.493 |
| 011 | 2.312 | 1.419 | 893 |
| 012 | 2.042 | 1.562 | 480 |

---

## Notebook 10 — `extração_2_features.ipynb` — Extração das Características Brutas

**Objetivo:** Calcular características numéricas de cada janela em quatro domínios diferentes.

**Entrada:** `data/silver/segments.npz` (24.564 janelas × 500 amostras × 54 canais)

**Saída:** `data/silver/features_raw.parquet` — 24.564 janelas × 1.251 características

Este é o notebook central da Fase B. Para cada uma das 24.564 janelas de 1 segundo, calculamos 1.251 números que descrevem o comportamento do sinal naquela janela. Cada número é uma **característica** (ou feature). As colunas seguem a convenção `{canal}__{domínio}__{métrica}`, por exemplo: `EEG-C3__freq__band_alpha` ou `LShank-GYRO-Z__time__VAR`.

### Família 1 — Domínio do Tempo (291 características)

Calculadas diretamente sobre os 500 pontos de cada janela, sem qualquer transformação de frequência. São as mais simples e rápidas de calcular.

| Característica | O que mede | Por que é útil para FoG |
| :--- | :--- | :--- |
| **RMS** | Energia média do sinal | Para EMG: quanto mais ativo o músculo, maior o RMS; durante FoG, o padrão muda |
| **MAV** | Amplitude média dos valores absolutos | Similar ao RMS, mais robusto a outliers pontuais |
| **VAR** | Dispersão dos valores em torno da média | Sinal mais irregular tem maior variância |
| **ZCR** | Quantas vezes o sinal cruza o zero por segundo | Para EEG: frequências mais altas cruzam o zero mais vezes |
| **Hjorth Mobility** | Raiz da variância da derivada dividida pela variância do sinal | Caracteriza a "frequência dominante" sem transformada de Fourier |
| **Hjorth Complexity** | Razão entre a mobilidade da derivada e a do sinal | Mede irregularidade: alto em sinais caóticos, baixo em sinais rítmicos |

### Família 2 — Domínio da Frequência (290 características)

A análise espectral transforma o sinal do tempo para a frequência, revelando quais ritmos estão presentes. Usamos o método de **Welch**: divide a janela em sub-janelas, calcula o espectro de cada uma e faz a média. O resultado é a **densidade espectral de potência (PSD)**, que mostra quanta energia existe em cada frequência.

Para o EEG, as bandas de frequência têm nomes tradicionais e significados fisiológicos:

| Banda | Faixa (Hz) | Associação fisiológica |
| :--- | :--- | :--- |
| **Delta** | 0,5 a 4 | Sono profundo; aumenta em déficits motores |
| **Theta** | 4 a 8 | Sonolência, memória de trabalho |
| **Alpha** | 8 a 13 | Relaxamento, idling motor |
| **Beta** | 13 a 30 | Alerta, movimento ativo |
| **Gamma** | 30 a 40 | Processamento sensorial de alta ordem |

Para os acelerômetros e giroscópios: banda de movimento (0,1 a 3 Hz) e banda de tremor (3 a 10 Hz).

Além da potência em cada banda, calculamos a **frequência mediana** (metade da potência está abaixo e metade acima) e a **frequência de pico** (onde está o maior pico espectral).

### Família 3 — Tempo-Frequência: STFT e DWT (661 características)

As transformadas acima assumem que o sinal é estacionário (as mesmas frequências do começo ao fim da janela). Mas durante o FoG, o sinal muda ao longo da janela. As características tempo-frequência capturam essa dinâmica.

**STFT (Short-Time Fourier Transform):**

Divide cada janela de 1 segundo em 4 sub-janelas de 250 ms e calcula o espectro de cada sub-janela. O resultado mostra como a potência de cada banda muda ao longo da janela. Por exemplo: se a energia delta do EEG aumenta no terceiro quarto da janela, isso pode indicar o início de um episódio de FoG.

**DWT (Discrete Wavelet Transform — Transformada Wavelet Discreta):**

Enquanto a Fourier decompõe o sinal em ondas senoidais, a wavelet decompõe em "ondas com formas variadas". A wavelet de Daubechies-4 (db4) usada aqui é especialmente boa para capturar transições rápidas, como as que ocorrem no início de um episódio de FoG.

A decomposição em 4 níveis mapeia as frequências assim:

| Nível | Faixa (Hz) | Relevância |
| :--- | :--- | :--- |
| D1 | 125 a 250 | Ruído de alta frequência |
| D2 | 62,5 a 125 | Artefatos musculares |
| D3 | 31,25 a 62,5 | Alta beta / gamma |
| D4 | 15,6 a 31,25 | Beta |
| A4 | 0 a 15,6 | Delta + theta + alpha |

A característica extraída é a energia normalizada de cada nível: soma dos coeficientes ao quadrado dividida pelo número de coeficientes.

### Família 4 — Características Não-Lineares (9 características)

Aplicadas apenas aos canais mais relevantes (EEG-C3, EEG-FP1, EMG-RTA), estas características capturam padrões que as transformadas lineares não conseguem expressar.

**Sample Entropy (SampEn):**

Mede a regularidade de um sinal. Um coração batendo normalmente é bastante regular (SampEn baixo). Um coração em fibrilação é caótico (SampEn alto). Para o EEG durante FoG, o padrão de regularidade muda. O cálculo usa uma **KD-Tree com distância de Chebyshev**: uma estrutura de dados que organiza os padrões do sinal em um espaço multidimensional para encontrar rapidamente os padrões similares (vizinhos), sem comparar todos os pares um a um.

A **KD-Tree** (K-Dimensional Tree) é uma árvore de busca binária adaptada para múltiplas dimensões. Em vez de comparar um padrão contra todos os outros (O(N²)), organiza os padrões em um espaço que permite encontrar vizinhos em O(N log N).

A **distância de Chebyshev** entre dois padrões é o máximo das diferenças em cada dimensão, em vez da raiz quadrada da soma dos quadrados (distância euclidiana). Ela é mais adequada para comparar janelas de sinal porque é menos sensível a dimensões individuais com grande variação.

**Approximate Entropy (ApEn):**

Similar ao SampEn, mas inclui auto-comparação (um padrão comparado consigo mesmo), tornando-o ligeiramente mais sensível a mudanças de regularidade em sinais curtos.

**DFA α (Detrended Fluctuation Analysis):**

Calcula o **expoente fractal** do sinal: quantifica se a série temporal tem memória de longo prazo. Para o EEG saudável, α ≈ 1. Valores muito diferentes indicam alterações na dinâmica cortical.

---

## Notebook 11 — `extração_3_feature_engineering.ipynb` — Engenharia de Características

**Objetivo:** Enriquecer o conjunto de características brutas com novas variáveis derivadas, normalizar por baseline individual e remover características redundantes ou sem poder discriminativo.

**Entrada:** `data/silver/features_raw.parquet`

**Saída:** `data/silver/features_engineered.parquet` — 24.564 × 1.370 características

### Passo 1 — Razões espectrais (+90 características)

Em vez de olhar a potência de cada banda isoladamente, calculamos **razões entre bandas**, que capturam o equilíbrio relativo entre diferentes ritmos cerebrais ou musculares:

| Razão | Fórmula | Interpretação |
| :--- | :--- | :--- |
| **Theta/Alpha** | θ / α | Sonolência e carga cognitiva: aumenta quando o estado motor deteriora |
| **Relax index** | (α + θ) / β | Alto = relaxado; baixo = alerta e em movimento ativo |
| **Delta/Alpha** | δ / α | Sedação vs. ativação cortical |
| **Alpha asymmetry** | log(α_direito / α_esquerdo) | Assimetria hemisférica, conhecida por estar alterada em Parkinson |
| **EMG fatigue** | potência_baixa / potência_alta | Fadiga muscular desloca a energia para frequências mais baixas |

### Passo 2 — Derivadas temporais Δ e Δ²

Para cada característica f calculada em uma janela t, calculamos:

```
Δ f(t)  = f(t) − f(t−1)       ← velocidade de mudança (1ª derivada discreta)
Δ² f(t) = Δf(t) − Δf(t−1)    ← aceleração de mudança (2ª derivada discreta)
```

Isso é importante porque, durante o FoG, a **transição** entre o estado normal e o congelado pode ser mais discriminativa do que o valor absoluto da característica. Se o EMG-RTA de repente cai de 0,5 para 0,1 em duas janelas consecutivas, esse Δ negativo é um sinal forte de FoG, mesmo que o valor 0,1 em si seja ambíguo.

As derivadas são calculadas dentro de cada combinação de paciente e tarefa, para evitar que uma diferença entre o fim de uma tarefa e o início de outra seja interpretada como variação fisiológica.

### Passo 3 — Normalização por baseline individual

Para cada paciente, calculamos a **mediana** de cada característica nas janelas de marcha normal (label=0) e subtraímos esse valor de todas as janelas daquele paciente:

```
f_normalizada(t, paciente) = f(t, paciente) - mediana_baseline(paciente)
```

O efeito é que as características ficam centradas no nível basal do próprio paciente, removendo a variabilidade inter-sujeito irrelevante. O que importa para a classificação é o **desvio em relação ao próprio baseline** do paciente, não o valor absoluto.

### Passo 4 — Filtro de relevância e deduplicação

**Filtro Spearman:** Calculamos a correlação de Spearman de cada característica com o rótulo. A correlação de Spearman trabalha com ranks (posições), sendo robusta a distribuições não-gaussianas. Uma característica é descartada se simultaneamente tiver |ρ| < 0,05 (correlação muito fraca) E p-valor > 0,05 (não significativa estatisticamente).

**Deduplicação:** Dentro do mesmo canal, se duas características têm correlação de Pearson acima de 0,95 entre si (quase idênticas), mantemos apenas a de maior correlação com o rótulo.

---

## Notebook 12 — `extração_4_reducao_dimensionalidade.ipynb` — Redução de Dimensionalidade (PCA e ICA)

**Objetivo:** Reduzir as 1.370 características a um conjunto menor de componentes que preserva a maior parte da informação.

**Entrada:** `data/silver/features_engineered.parquet`

**Saída:**
- `data/silver/features_pca.parquet` — 447 componentes principais
- `data/silver/pca_loadings.parquet` — pesos de cada feature original
- `data/silver/pca_scaler.joblib` e `pca_model.joblib`

### PCA (Análise de Componentes Principais)

**O que é PCA?**

Com 1.370 características, é provável que muitas variem juntas. O PCA encontra essas fontes compartilhadas de variação e as representa como novas variáveis: os **componentes principais**.

Pense assim: você tem 1.370 eixos num espaço de 1.370 dimensões. O PCA encontra uma nova orientação desses eixos, organizados do que mais varia ao que menos varia. O primeiro componente captura a maior quantidade de variação possível, o segundo captura o que sobrou, e assim por diante.

**Por que é útil?**

Com 24.564 janelas e 1.370 características, muitas características são redundantes. O PCA permite trabalhar com um número menor de variáveis, sem perder muita informação. Isso reduz o risco de overfitting: o classificador decorar padrões do dataset de treino que não se repetem em dados novos.

**Implementação:**

Antes do PCA, todas as características são padronizadas com **StandardScaler** (média zero, desvio padrão 1) globalmente. Usamos **PCA randomizado** (svd_solver='randomized'), que usa uma aproximação matemática muito mais rápida do que a decomposição exata, sem perda significativa de precisão.

**Resultado:** Foram necessários **447 componentes** para acumular 95% da variância total das 1.370 características. O critério de qualidade (RMSE relativo de reconstrução < 5%) foi atendido.

### ICA (Análise de Componentes Independentes) nos canais EEG

**O que é ICA?**

Enquanto o PCA encontra direções que maximizam a variância, a ICA encontra direções que são **estatisticamente independentes** entre si, o que é diferente de apenas não-correlacionadas.

Para EEG, a ICA é uma ferramenta clássica de identificação de artefatos. O raciocínio é: artefatos de piscar de olho e movimentação muscular geram padrões com kurtose muito alta (distribuição de cauda pesada, com spikes frequentes), enquanto a atividade cerebral genuína tem distribuição mais gaussiana.

**Aplicação:** Como o dataset tem apenas 2 canais EEG (EEG-C3 e EEG-FP1), a ICA é aplicada no espaço de características derivadas desses canais para separar padrões independentes. Componentes com kurtose de Fisher (excesso) acima de 5 em módulo são marcados como possíveis artefatos.

---

## Notebook 13 — `extração_5_selecao_atributos.ipynb` — Seleção de Atributos

**Objetivo:** Selecionar as características mais relevantes e discriminativas usando quatro métodos independentes, combinando os resultados por consenso.

**Entrada:** `data/silver/features_engineered.parquet`

**Saída:**
- `data/silver/features_selecionadas.parquet` — 156 características selecionadas
- `data/silver/feature_selection_rankings.parquet` — ranking completo

**Por que selecionar em vez de usar tudo?**

Com 1.370 características e 24.564 janelas, surgem problemas: o **problema da maldição da dimensionalidade**. Em espaços de alta dimensão, todos os pontos ficam igualmente "distantes" uns dos outros, fazendo os classificadores baseados em distância (KNN) e regularização (SVM, Regressão Logística) perderem poder discriminativo.

A regra prática adotada é: número de features selecionadas < √n_janelas = √24.564 ≈ **156 features**. Esse limite evita overfitting sem subutilizar o dataset.

### Método 1 — ANOVA F-test

O teste ANOVA compara as médias de cada característica entre as duas classes (normal vs. FoG). A estatística F é a razão entre a variância entre grupos e a variância dentro dos grupos. Quanto maior o F, mais discriminativa a característica.

**Resultados — Top 5 pelo F-statistic:**

| Característica | F-statistic | Cohen's d |
| :--- | :--- | :--- |
| LShank-GYRO-Z\_\_time\_\_VAR | 13.562,8 | -1,51 |
| LShank-GYRO-Z\_\_time\_\_MAV | 13.488,3 | -1,50 |
| LShank-GYRO-Y\_\_time\_\_RMS | 9.040,5 | -1,23 |
| LShank-GYRO-Y\_\_time\_\_VAR | 8.922,4 | -1,22 |
| LShank-GYRO-Z\_\_time\_\_ZCR | 7.168,9 | 1,10 |

As características de giroscópio da perna esquerda dominam o topo do ranking. Isso faz sentido fisiologicamente: durante o FoG, a variância do giroscópio da perna esquerda cai drasticamente, pois os pés param de se mover. O Cohen's d negativo indica que a característica é menor durante FoG do que durante marcha normal.

### Método 2 — Mutual Information (Informação Mútua)

A informação mútua mede o quanto conhecer o valor de uma característica reduz a incerteza sobre o rótulo. Ao contrário da ANOVA, ela captura relações não-lineares. Se uma característica tem uma relação em forma de U com o rótulo (muito baixo e muito alto ambos indicam FoG), a ANOVA não detecta isso, mas a informação mútua detecta.

### Método 3 — ReliefF (subamostrado)

O ReliefF é especialmente poderoso para biossinais porque considera **interações entre características**. O algoritmo funciona assim:

1. Seleciona uma janela aleatoriamente.
2. Encontra as k janelas mais parecidas da mesma classe (vizinhos "acertados").
3. Encontra as k janelas mais parecidas da classe oposta (vizinhos "errados").
4. Aumenta o peso de características que diferenciam a janela dos vizinhos errados (bom discriminador).
5. Diminui o peso de características que diferenciam a janela dos vizinhos acertados (ruído).

Esse processo é repetido 200 vezes sobre 2.000 janelas subamostradas (das 24.564 totais) para controlar o custo computacional. A distância usada é a distância L1 (soma das diferenças absolutas em cada feature), que é mais robusta a outliers do que a distância euclidiana.

### Método 4 — LASSO (método embutido)

O LASSO (Least Absolute Shrinkage and Selection Operator) é um método de regularização que força alguns coeficientes do modelo a exatamente zero, funcionando simultaneamente como modelo e seletor de características.

Usamos regressão logística com penalidade L1 e variamos o parâmetro de regularização C de muito pequeno (alta regularização, pouquíssimas features ativas) até 1 (baixa regularização, quase todas as features ativas). A **ordem em que cada feature "acorda"** ao aumentar C é usada como ranking de importância LASSO. Features que aparecem com a regularização mais intensa são as mais robustamente úteis.

O gráfico `lasso_path.png` mostra esse "caminho de regularização": à medida que C cresce, novas features entram no modelo.

### Correção BH e Cohen's d

**Problema das múltiplas comparações:** Com 1.370 testes simultâneos (um por característica) ao nível p < 0,05, esperamos encontrar por acaso cerca de 68 características "significativas" que na verdade são ruído (falsos positivos).

A **correção de Benjamini-Hochberg (BH)** controla a **Taxa de Falsa Descoberta (FDR)**: a proporção esperada de falsos positivos entre todas as rejeições não pode ultrapassar 5%. É menos conservadora que a correção de Bonferroni, mas mais adequada quando o objetivo é selecionar variáveis úteis, não garantir que nenhum falso positivo exista.

O **Cohen's d** quantifica o tamanho do efeito de forma independente do tamanho amostral:

```
d = (média_FoG − média_normal) / desvio_padrão_pooled
```

Interpretação: |d| < 0,2 é trivial, 0,2 a 0,5 é pequeno, 0,5 a 0,8 é médio, > 0,8 é grande. As características de giroscópio chegam a |d| > 1,5, indicando efeito muito grande.

### Ranking de Consenso

Para cada característica, somamos suas posições nos rankings de ANOVA, MI e ReliefF. Características com **menor soma** são as mais consensualmente informativas pelos três critérios independentes. As 156 características com melhor consenso formam o conjunto `features_selecionadas.parquet`.

---

## Notebook 14 — `extração_6_validacao_final.ipynb` — Validação e Dataset Final

**Objetivo:** Verificar a qualidade estatística do conjunto selecionado, remover multicolinearidade e documentar o dataset final.

**Entrada:** `data/silver/features_selecionadas.parquet`

**Saída:**
- `data/silver/dataset_final.parquet` — 125 características finais
- `data/silver/dataset_manifest.json`

### Passo 1 — Análise de Multicolinearidade (VIF)

**O que é multicolinearidade?**

Quando duas características são muito correlacionadas entre si, é difícil saber qual delas está realmente contribuindo para a classificação. Em modelos lineares, coeficientes de características muito correlacionadas ficam instáveis, variando muito com pequenas mudanças nos dados de treino.

**O que é o VIF (Variance Inflation Factor)?**

O VIF mede o quanto a estimativa de uma característica é "inflada" pela sua correlação com as demais:

- VIF = 1: sem correlação com as outras
- VIF 1 a 5: correlação moderada, aceitável
- VIF > 5: correlação alta, problemática
- VIF > 10: multicolinearidade grave

**Cálculo eficiente via matriz inversa:**

Em vez de ajustar uma regressão separada para cada característica, calculamos o VIF de todas de uma vez: `VIF_j = [R⁻¹]_jj`, onde R é a matriz de correlação de Pearson e R⁻¹ é sua inversa. Isso é equivalente a 1/(1 − R²) da regressão de j sobre todas as demais, mas muito mais rápido computacionalmente.

**Remoção iterativa:** Em cada iteração, removemos a característica com maior VIF e recalculamos para as restantes. Repetimos até que todas tenham VIF < 5.

**Resultado:** Das 156 características selecionadas, **31 foram removidas por VIF > 5**, resultando em **125 características finais**.

### Passo 2 — Separabilidade das Classes (Índice de Fisher)

**O que é o Índice de Fisher?**

Mede o quanto as distribuições das duas classes se separam para cada característica:

```
FI_j = (média_FoG − média_normal)² / (variância_normal + variância_FoG)
```

FI próximo de zero: as distribuições se sobrepõem totalmente (característica não discriminativa). FI >> 1: as distribuições estão bem separadas (característica altamente discriminativa).

As curvas de densidade KDE (Kernel Density Estimation) para as 5 características com maior FI (imagem `density_top5_fisher.png`) mostram visualmente essa separação: quando as curvas de cada classe têm pouca sobreposição, a característica é boa discriminadora.

**Resultado:** `fi_mean_top10 = 0,2393`. O índice médio das 10 melhores características é 0,24, que está na faixa de separabilidade moderada. Isso indica que nenhuma característica individual separa perfeitamente as classes, mas a combinação de 125 características complementares pode gerar um classificador robusto. Classificadores não-lineares (Random Forest, XGBoost, SVM com kernel RBF) se beneficiam exatamente dessa situação.

### Passo 3 — Balanceamento do Dataset

O dataset tem **14.386 janelas normais** (58,6%) e **10.178 janelas de FoG** (41,4%). A razão é 10.178 / 14.386 = **0,7075**: para cada 10 janelas normais há 7 de FoG. Esse nível é considerado **balanceado** pelo critério adotado (razão > 60%).

**Importante:** Não aplicamos SMOTE (Synthetic Minority Oversampling Technique) nem undersampling nesta etapa. A decisão pertence ao módulo de classificação, onde a estratégia de balanceamento deve ser aplicada **dentro de cada fold** da validação cruzada LOSO. Se balancearmos antes da validação cruzada, estaríamos criando amostras artificiais que "vazam" informação do conjunto de treino para o de teste (data leakage), inflando artificialmente as métricas de avaliação.

### Passo 4 — Correlação Média entre Features

O heatmap `feature_correlation_heatmap.png` mostra a matriz de correlação entre as 125 features finais. A correlação média em valor absoluto é **0,0951**, bem abaixo do limite de 0,5. Isso confirma que as 125 features selecionadas são razoavelmente independentes entre si: cada uma carrega informação diferente das outras.

### Dataset Final — Resumo

O arquivo `dataset_manifest.json` resume o resultado de todo o pipeline:

```json
{
  "n_patients": 12,
  "n_windows": 24564,
  "n_windows_label0": 14386,
  "n_windows_label1": 10178,
  "class_ratio": 0.7075,
  "n_features_raw": 156,
  "n_features_final": 125,
  "n_removed_vif": 31,
  "mean_inter_corr": 0.0951,
  "fi_mean_top10": 0.2393
}
```

O `dataset_final.parquet` está pronto para o módulo de classificação.

---

## Resumo do Pipeline Completo

```
data/raw/original/
│
├── bronze_bruta.ipynb
│     → data/bronze/full_table.parquet
│
├── bronze_filtrada.ipynb
│     → data/bronze/filtered_parquet/<paciente>/task_*.parquet
│
│  [Fase A — Pré-processamento]
│
├── PP1 → funções e validação estrutural
├── PP2 → resampled.parquet   (reamostragem para 500 Hz)
├── PP3 → filtered.parquet    (filtragem por banda + notch 50 Hz)
├── PP4 → normalized.parquet  (Z-score por paciente + clip)
│         comparacao_report.txt
├── PP5 → sqi_mask.parquet    (651/769 janelas de 5s aprovadas: 84,7%)
└── PP6 → pp6_report.txt      (0/54 canais normais; 0/54 homocedásticos)
│
│  [Fase B — Extração de Características]
│
├── Ext1 → segments.npz             (24.564 × 500 × 54)
│           windows_metadata.parquet
│
├── Ext2 → features_raw.parquet     (24.564 × 1.251 features)
│
├── Ext3 → features_engineered.parquet  (24.564 × 1.370 features)
│
├── Ext4 → features_pca.parquet     (24.564 × 447 componentes PCA)
│    [caminho paralelo ao Ext5]
│
├── Ext5 → features_selecionadas.parquet  (24.564 × 156 features)
│           feature_selection_rankings.parquet
│
└── Ext6 → dataset_final.parquet    (24.564 × 125 features)
            dataset_manifest.json
│
[Próxima etapa: classificação_1.ipynb — LOSO cross-validation]
```

**Observação sobre os caminhos paralelos:** O `extração_4` (PCA) e o `extração_5` (seleção de atributos) partem ambos do `features_engineered.parquet`. São duas abordagens diferentes para redução de dimensionalidade: o PCA transforma as features em novos eixos (perde interpretabilidade, mas pode ser mais compacto), enquanto a seleção de atributos mantém as features originais (interpretabilidade preservada). O módulo de classificação pode testar as duas abordagens de forma independente.

---

## Próximos Passos

O `dataset_final.parquet` em `data/silver/` está preparado para o módulo de classificação com o seguinte protocolo planejado:

- **Validação cruzada LOSO (Leave-One-Subject-Out):** Em cada fold, um paciente é reservado para teste e os 11 restantes são usados para treino. Isso simula o cenário real em que o sistema precisa generalizar para um novo paciente nunca visto. Com 12 pacientes, haverá 12 folds.

- **Classificadores candidatos:** Random Forest, XGBoost e SVM com kernel RBF. São escolhas naturais dado que as classes não são linearmente separáveis por características individuais (FI médio = 0,24) e que os dados são heteroscedásticos entre pacientes.

- **Balanceamento dentro do fold:** Se necessário, aplicar SMOTE ou class_weight='balanced' apenas sobre os dados de treino de cada fold, para evitar data leakage.

- **Métricas:** Sensibilidade (recall de FoG) e F1-score são as métricas mais importantes. Em aplicações médicas, errar um episódio de FoG (falso negativo) é mais grave do que um alarme falso (falso positivo).

---

## Dependências

Instale as dependências com:

```bash
pip install -r requirements.txt
```

As principais bibliotecas utilizadas são: `numpy`, `pandas`, `matplotlib`, `seaborn`, `scipy`, `scikit-learn`, `mne`, `pyarrow`, `joblib` e `pywavelets`.
