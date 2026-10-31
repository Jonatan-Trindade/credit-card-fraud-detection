![Detecção de fraude em cartão de crédito](banner.jpg)

# **Detecção de fraude em cartão de crédito**

Projeto de análise e modelagem de fraude no dataset Credit Card Fraud Detection (MLG-ULB), com 284.807 transações e apenas 0,17% de fraudes.

- Dataset: [https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud](https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud)

# **O que foi feito**

1. Análise exploratória: quantidade por classe, densidade dos valores, distribuição acumulada e transações ao longo do tempo.
2. Probabilidade condicional, probabilidade total e teorema de Bayes para calcular a probabilidade de fraude por faixa de valor.
3. Comparação de modelos: Naive Bayes, Regressão Logística, Random Forest e CatBoost (ajustado com Optuna).
4. Validação cruzada (5 divisões) dentro do treino para comparar os modelos, com média e desvio do PR-AUC.
5. Escolha do limite de alerta pela validação cruzada, sem usar o teste, e avaliação final no teste uma única vez.
6. Teste de se a variável de tempo ajuda e análise das variáveis mais importantes.

# **Resultados**

**Validação cruzada (5 divisões, dentro do treino), PR-AUC:**

| Modelo              | PR-AUC (média ± desvio) |
|---------------------|-------------------------|
| Naive Bayes         | 0,090 ± 0,011           |
| Regressão Logística | 0,762 ± 0,058           |
| Random Forest       | 0,857 ± 0,019           |
| CatBoost            | 0,869 ± 0,022           |

**Conjunto de teste (uma única avaliação):**

| Modelo                                    | PR-AUC | ROC-AUC |
|-------------------------------------------|--------|---------|
| Só o valor da transação (Bayes por faixa) | 0,0046 | 0,745   |
| Naive Bayes                               | 0,080  | 0,955   |
| Regressão Logística                       | 0,705  | 0,968   |
| Random Forest                             | 0,820  | 0,938   |
| CatBoost                                  | 0,835  | 0,977   |
| Chute (proporção de fraude)               | 0,0017 | 0,5     |

- **CatBoost e Random Forest empatam na prática:** a vantagem do CatBoost na validação cruzada (0,012) é menor que o desvio entre as divisões (~0,02). O Optuna usou o mesmo treino da validação cruzada, então o CatBoost leva uma pequena vantagem sobre o Random Forest, que não foi ajustado.
- **A variável de tempo não ajuda:** o CatBoost tem PR-AUC 0,869 com e sem ela.
- **Limite de alerta:** escolhido na validação cruzada pela regra "maior recall mantendo pelo menos 80% de precisão" (limite 0,065 no CatBoost). No teste, deu 157 alertas: 121 eram fraude e 36 eram falsos, e 27 fraudes passaram. Precisão 0,77 e recall 0,82. A precisão ficou um pouco abaixo dos 80% buscados, o que é esperado com tão poucos alertas.
- **Variáveis mais importantes (CatBoost):** V4 se destaca (cerca de 8,6% da importância), seguida de V14, V1, V26, V10, V7, V20, scaled_amount, V12 e V17. A importância é espalhada: as dez juntas somam cerca de metade do total.

# **Como rodar**

1. Baixe o dataset no Kaggle (link acima) e coloque o `creditcard.csv` na mesma pasta do notebook. O arquivo não está neste repositório por ser grande.
2. Instale as dependências: `pip install -r requirements.txt`
3. Abra o `creditcardfraud.ipynb` e use Restart and Run All. A busca de hiperparâmetros com Optuna vem desligada (`RODAR_OPTUNA = False`) e usa os parâmetros já encontrados; a validação cruzada leva alguns minutos.

# **Limitações**

- V1 a V28 são componentes anônimas (PCA), então não dá para interpretá-las.
- O dataset não identifica clientes, então a análise é do padrão global de fraude, e não do comportamento individual de cada cliente.
- O conjunto de teste tem poucas fraudes (148), então os resultados têm incerteza: o PR-AUC no teste (0,835) ficou abaixo da validação cruzada (0,869).
- O dataset contém linhas idênticas, que esta versão não remove (`REMOVER_DUPLICATAS = False`). Se uma cópia cair no treino e outra no teste, as métricas podem sair infladas.
- Os dados cobrem apenas dois dias, sem validação temporal.

# **Tecnologias e por que usei**

[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/) [![NumPy](https://img.shields.io/badge/NumPy-013243?style=for-the-badge&logo=numpy&logoColor=white)](https://numpy.org/) [![pandas](https://img.shields.io/badge/pandas-150458?style=for-the-badge&logo=pandas&logoColor=white)](https://pandas.pydata.org/) [![Matplotlib](https://img.shields.io/badge/Matplotlib-11557C?style=for-the-badge)](https://matplotlib.org/) [![Seaborn](https://img.shields.io/badge/Seaborn-4C72B0?style=for-the-badge)](https://seaborn.pydata.org/) [![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?style=for-the-badge&logo=scikitlearn&logoColor=white)](https://scikit-learn.org/) [![CatBoost](https://img.shields.io/badge/CatBoost-FFCC00?style=for-the-badge&logoColor=black)](https://catboost.ai/) [![Optuna](https://img.shields.io/badge/Optuna-3F5FD0?style=for-the-badge)](https://optuna.org/) [![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white)](https://jupyter.org/)

| Tecnologia       | Por que usei                                                                                                                                |
|------------------|---------------------------------------------------------------------------------------------------------------------------------------------|
| Python           | Linguagem padrão em ciência de dados, com as bibliotecas de análise e de modelagem que o projeto precisava                                  |
| NumPy            | Cálculos numéricos: transformação logarítmica dos valores, faixas dos histogramas e densidades por classe                                   |
| pandas           | Carregar o dataset e manipular os dados em tabelas (separar classes, treino e teste, tabelas de resultados)                                 |
| Matplotlib       | Estrutura base dos gráficos                                                                                                                 |
| Seaborn          | Gráficos estatísticos: densidades, distribuição acumulada, curvas ROC e Precisão × Recall, mapas de calor e importância das variáveis       |
| scikit-learn     | Divisão treino/teste, escalonamento, modelos (Naive Bayes, Regressão Logística e Random Forest), validação cruzada e métricas (PR-AUC e ROC-AUC) |
| CatBoost         | Modelo de gradiente boosting em árvores, que teve o melhor PR-AUC na validação cruzada                                                      |
| Optuna           | Busca dos hiperparâmetros do CatBoost, otimizando o PR-AUC em validação cruzada                                                             |
| Jupyter Notebook | Juntar código, gráficos e explicações no mesmo documento                                                                                    |

# **Técnicas estatísticas utilizadas**

| Técnica                                                        | Onde foi usada                                                                                       |
|----------------------------------------------------------------|------------------------------------------------------------------------------------------------------|
| Histograma de densidade                                        | Comparar como os valores e o horário das transações se distribuem em fraudes e em transações normais |
| Distribuição acumulada empírica, F(x)                          | Ver a fração de transações até cada valor, em cada classe                                            |
| Transformação logarítmica, log(1 + x)                          | Tornar legível a distribuição dos valores, que tem cauda muito longa                                 |
| Escalonamento robusto (RobustScaler)                           | Reescalar valor e tempo com mediana e intervalo interquartil, aprendido só no treino                 |
| Probabilidade a priori                                         | Proporção de fraude no dataset (0,17%)                                                               |
| Verossimilhança                                                | Densidade dos valores dentro de cada classe, P(valor \| classe)                                      |
| Probabilidade total                                            | Denominador do teorema de Bayes, combinando as duas classes ponderadas pelo priori                   |
| Teorema de Bayes (probabilidade a posteriori)                  | Probabilidade de fraude dado o valor da transação, calculada por faixa                               |
| Naive Bayes                                                    | Somar a evidência de todas as variáveis, assumindo independência entre elas                          |
| Amostragem estratificada                                       | Manter a proporção de fraude no treino, no teste e em cada divisão da validação cruzada              |
| Validação cruzada estratificada (5 divisões)                   | Comparar os modelos com média e desvio, sem depender de uma única divisão dos dados                  |
| Busca de hiperparâmetros (Optuna) e parada antecipada          | Ajustar o CatBoost e definir o número de iterações sem usar o teste                                  |
| Balanceamento de classes por pesos (`class_weight="balanced"`) | Compensar o desbalanceamento (0,17% de fraudes) no treino da Regressão Logística e do Random Forest  |
| Precisão, recall e matriz de confusão                          | Contar alertas, acertos, alertas falsos e fraudes que passaram em cada limite de decisão             |
| Escolha do limite por validação cruzada                        | Definir o limite de alerta sem olhar o teste, com precisão mínima de 80%                             |
| PR-AUC e ROC-AUC                                               | Comparar os modelos sem depender de um único limite de alerta                                        |
| Comparação com e sem a variável de tempo                       | Verificar na validação cruzada se o tempo melhora o modelo                                           |
| Importância de variáveis (CatBoost)                            | Descobrir quais variáveis mais ajudam a separar fraude de transação normal                           |

# **Possíveis melhorias**

- Comparar os dois melhores modelos com o teste de McNemar (separado em fraudes e normais) e bootstrap pareado do PR-AUC.
- Escolher o limite de alerta com base no custo de cada tipo de erro (alerta falso e fraude que passa), usando o valor real de cada transação.
- Rodar com e sem as linhas idênticas e comparar os resultados.
- Testar outros modelos, como XGBoost e LightGBM.
- Comparar outras formas de lidar com o desbalanceamento, como subamostragem e sobreamostragem (SMOTE).
- Repetir a validação cruzada várias vezes para estimar melhor a incerteza dos resultados.
- Avaliar se as probabilidades dos modelos são bem calibradas.
- Explicar as decisões com SHAP e validar com divisão temporal.
- Medir a distância entre as distribuições de fraude e normal com o teste de Kolmogorov-Smirnov.
- Refazer a análise por cliente (padrão individual de gasto diário e mensal), com um dataset que identifique clientes.
