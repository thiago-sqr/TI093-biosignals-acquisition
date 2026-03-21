# Análise Multimodal de Biosinais para Detecção de Freezing of Gait

## Descrição do Projeto:

Este projeto tem como objetivo analisar sinais biomédicos multimodais para detectar episódios de Freezing of Gait (FoG), um sintoma da Doença de Parkinson em que o paciente “trava” durante a caminhada. 

A proposta envolve trabalhar com sinais cerebrais, musculares e de movimento para identificar padrões associados ao FoG.

## Objetivos:

-Análise descritiva dos sinais e dos dados

-Investigar padrões fisiológicos associados ao FoG

-Extração de características relevantes

-Dashboard de comparações

## Objetivos futuros:

-Classificação automática com Machine Learning

## Dataset:

- O dataset contém dados sobre 11 pacientes
- Foi feito experimentos controlados com indução de FoG
- Sinais foram coletados simultâneamente (multimodalidade)

### Tipos de sinais:

| Tipo     | Descrição                                 |
| -------- | ----------------------------------------- |
| EEG      | Atividade elétrica cerebral               |
| EMG      | Atividade muscular                        |
| ECG      | Atividade cardíaca                        |
| EOG      | Movimento ocular                          |
| ACC/Gyro | Movimento corporal (aceleração e rotação) |
| SC       | Condutância da pele                       |

### Sinais Raw e Filtered 

- Os dados Raw são os dados brutos contidos em arquivos separados com formatos .eeg, .vhdr, .vrmk e .csv (movimentos), o formato .eeg representa os dados brutos do sinal, o .vhdr é o manual que explica esses dados, enquanto o .vmrk é a marca de eventos. Já os dados .csv se tratam dos movimentos do paciente. Se utilizou o sistema "MOVE" para gerar os arquivos dos sinais EEG/EMG/ECG/EOG numa frequência de amostragem de 1000Hz (arquivo .eeg contém canais com esses tipos de sinais, não somento os EEG). Para os movimentos utilizou-se os sensores MPU6050 e LM324 numa frequência de amostragem de 500Hz. Os sinais Raw contém ruídos e frequências diferentes, sendo assim difíceis de serem usados diretamente.

- Os dados Filtered são todos os sinais que foram sincronizados, limpos e organizados. Estão convertidos em uma tabela única por tarefa (task). O arquivo task_n.txt por exemplo é a tabela de dados de sinais de um paciente ao realizar determinada atividade. Nessa tabela temos que as linhas representam o tempo e as colunas são os sinais e suas medições, sendo colunas com zeros representativas do não uso daquele determinado sensor para medição de sinal.

### Colunas Filtered

| Colunas | Conteúdo                    |
| ------- | --------------------------- |
| 1       | Tempo                       |
| 2–26    | EEG                         |
| 27–31   | EMG / ECG / EOG             |
| 32–59   | Movimento (ACC + Gyro + SC) |
| 60      | Label (FoG ou não)          |

-A coluna 1 trata-se do tempo desde o início, se minha taxa de amostragem for de 500 Hz, e o tempo do experimento dor 10 segundos por exemplo, teremos cerca de 5000 linhas de amostras.

-As colunas 2-26 são os 25 canais diferentes para os sinais do cérebro EEG.

-As colunas 27-31 são variáveis por paciente e se tratam dos sinais do corpo elas no geral contém:
-- EMG (músculos): Cada canal representa um músculo específico (ordem pode ser diferente), Tibialis anterior (TA) → frente da perna, Gastrocnemius (GS) → panturrilha precedidos por R (direita) ou L (esquerda).
-- ECG (coração)
-- EOG (olhos): IO (Electrooculogram) → sempre na coluna 29

-As colunas 32-59 contém os sinais obtidos pelos sensores de mmovimento para cada parte do corpo (perna esquerda, perna direita, cintura, braço) temos 7 colunas (acc_x, acc_y, acc_z, gyro_x, gyro_y, gyro_z, último) , totalizando 28 colunas para os movimentos. Essa coluna "último" representa a condutância de pele (SC) para o braço e é ignorada pra perna e cintura.

-A coluna 60 trata do label, se o paciente apresenta (1) ou não (o) o FoG.

## Pré-processamento
Foi feito um pré-processamento para os sinais:

-EEG
--Remoção de artefatos com ICA
--Referência: TP9, TP10 (atrás da orelha). Esses canais não aparecem no dataset final

-EMG
--Filtro: 10–500 Hz

-ACC
--Filtro passa-baixa: até 16 Hz → limite de frequência do sinal

-Notch filter
--Remove ruído elétrico (50 Hz)

## Organização do projeto (Primeira entrega)

TI093-biosignals-acquisition/
│
├── data/
│   └── raw/
│     └── README.md
│   └── README.pdf
├── .gitignore
├── notebooks/
├── README.md
├── requirements.txt
└── test.ipynb
