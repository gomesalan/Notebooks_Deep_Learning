
# 🏦 Previsão de Adesão a Depósitos a Prazo (Bank Marketing)

---

## 📌 1. Visão Geral do Projeto

Este projeto aborda o problema clássico de campanhas de telemarketing bancário: **como maximizar as adesões a depósitos a prazo fixo minimizando os custos de acionamento e o desgaste da base de clientes.**

Utilizando o dataset *Bank Marketing* (OpenML ID 1461), desenvolvemos uma pipeline completa de **Engenharia de Features**, **Pré-processamento do Dados** e uma **Rede Neural Multi-Layer Perceptron (MLP)** em PyTorch. Para lidar com o forte desbalanceamento das classes, aplicamos a estratégia de **Class Weights** diretamente na função de perda (`BCEWithLogitsLoss`), garantindo alta sensibilidade.

---

## 🛠️ 2. Arquitetura da Solução e Engenharia de Features

### 🔄 Pré-processamento e Engenharia
* **Transformação Cíclica de Tempo:** Variáveis temporais (mes e dia) foram convertidas em componentes seno e cosseno ($\sin/\cos$) para preservar a natureza contínua e periódica do calendário.
* **Mapeamento de Pdays:** Tratamento do indicador **-1** (não contatado anteriormente) separando em um indicador binário e zerando a contagem de dias.
* **Pipeline Scikit-Learn:**
  * **Categorias Nominais:** Imputador do valor mais frequente + *One-Hot Encoding*.
  * **Categorias Ordinais:** Ordenação explícita de escolaridade + *Ordinal Encoding*.
  * **Variáveis Numéricas:** Impotador por mediana + *Yeo-Johnson Power Transformer* (para normalização de assimetria) + *MinMaxScaler*.

### 🧠 Modelo (PyTorch MLP)
* **Arquitetura:** Dense Neural Network de 3 camadas ocultas (`Input` $\rightarrow$ `128` $\rightarrow$ `64` $\rightarrow$ `32` $\rightarrow$ `1`).
* **Regularização:** *Batch Normalization* e *Dropout* ($20\%$) após cada camada oculta.
* **Otimização:** Algoritmo `AdamW` com taxa de aprendizado inicial de $10^{-4}$ e *Scheduler* `ReduceLROnPlateau`.
* **Função de Perda:** `BCEWithLogitsLoss` com `pos_weight` calculado dinamicamente ($N_{negativos} / N_{positivos} \approx 7,53$).

---

## 📉 3. Convergência do Modelo

O treinamento executou por 150 épocas utilizando o critério de salvamento do melhor modelo com base no menor valor de perda(Loss).

---

## 📊 4. Resultados Técnicos e Métricas

Avaliação realizada na base de teste não vista contendo **9.043 clientes** ($1.058$ conversores reais / classe minoritária de $11,7\%$).

### Relatório de Classificação (Threshold = 0.50)

```text
                precision    recall  f1-score   support

Não Subscreveu       0.98      0.80      0.88      7985
    Subscreveu       0.37      0.89      0.52      1058

      accuracy                           0.81      9043
     macro avg       0.68      0.85      0.70      9043
  weighted avg       0.91      0.81      0.84      9043

```

---

## 💼 5. Análise de Impacto no Negócio (ROI e Simulação)

Para validar o valor do modelo no cenário real de um banco europeu, simularam-se três cenários operacionais com as seguintes premissas financeiras:

* **Custo estimado por ligação telefônica:** **€ 5,00**
* **Lucro médio estimado por depósito convertido:** **€ 100,00**

### 📊 Tabela Comparativa de Cenários Operacionais

| Métrica / Cenário | 1. Sem Modelo (Ligar p/ Todos) | 2. Modelo (Threshold = 0.35) | 3. Modelo (Threshold = 0.50) |
| --- | --- | --- | --- |
| **Volume de Ligações** | $9.043$ ($100\%$) | $2.980$ ($33,0\%$) | **$2.545$ ($28,1\%$)** |
| **Clientes Convertidos (Recall)** | $1.058$ ($100\%$) | **$984$ ($93,0\%$)** | **$942$ ($89,0\%$)** |
| **Taxa de Eficiência (Precision)** | $11,7\%$ | $33,0\%$ | **$37,0\%$** |
| **Custo de Telemarketing** | € 45.215,00 | **€ 14.900,00** | **€ 12.725,00** |
| **Receita Bruta Gerada** | € 105.800,00 | **€ 98.400,00** | **€ 94.200,00** |
| **Resultado Líquido (Lucro - Custo)** | **€ 60.585,00** | **€ 83.500,00** | **€ 81.475,00** |
| **Aumento no Lucro vs. Sem Modelo** | Base | **$+37,8\%$ (+€ 22.915)** | **$+34,5\%$ (+€ 20.890)** |

---

### 💡 Principais Conclusões Estratégicas

1. **Redução Maciça de Custos Operacionais:** O uso do modelo permite economizar mais de **€ 30.000 em custos de chamadas**, reduzindo em mais de $70\%$ o volume de acionamentos necessários.
2. **Maximização do Lucro Líquido:** O lucro líquido final da campanha salta de **€ 60.585,00 para até € 83.500,00** (aumento de **$+37,8\%$ no ROI**).
3. **Decisão Baseada em Capacidade Operacional:**
* **Se houver capacidade ociosa no Call Center:** Recomenda-se o **Threshold 0.35**, capturando $93\%$ dos potenciais clientes e gerando o lucro líquido máximo.
* **Se a equipe for enxuta:** Recomenda-se o **Threshold 0.50**, economizando **435 ligações adicionais** e aumentando a precisão das chamadas para $37\%$ com impacto financeiro mínimo.
---

