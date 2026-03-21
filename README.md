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

## Organização do Projeto (Primeira Entrega)

```text
TI093-biosignals-acquisition/
├── data/
│   ├── raw/
│   │   └── README.md
│   └── README.pdf
├── notebooks/
├── .gitignore
├── README.md
├── requirements.txt
└── test.ipynb
