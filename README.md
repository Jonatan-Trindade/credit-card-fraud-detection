# **Detecção de fraude em cartão de crédito**

Projeto de análise e modelagem de fraude no dataset Credit Card Fraud Detection (MLG-ULB), com 284.807 transações e apenas 0,17% de fraudes.

- Notebook no Kaggle: https://www.kaggle.com/code/trinmat/creditcardfraud
- Dataset: https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud

# **O que foi feito**

1. Análise exploratória: quantidade por classe, densidade dos valores, distribuição acumulada e transações ao longo do tempo.
2. Probabilidade condicional, probabilidade total e teorema de Bayes para calcular a probabilidade de fraude por faixa de valor.
3. Comparação de modelos: Naive Bayes, Regressão Logística e Random Forest.
4. Validação cruzada (5 divisões) e análise das variáveis mais importantes.

# **Resultados (conjunto de teste)**

| Modelo | PR-AUC | ROC-AUC |
|---|---|---|
| Só o valor da transação (Bayes por faixa) | 0,0046 | 0,745 |
| Naive Bayes | 0,081 | 0,956 |
| Regressão Logística | 0,698 | 0,969 |
| Random Forest | 0,818 | 0,937 |
| Chute (proporção de fraude) | 0,0017 | 0,5 |

Na validação cruzada (conjunto de treino), o PR-AUC médio foi 0,855 para o Random Forest e 0,775 para a Regressão Logística. As variáveis mais importantes foram V10, V14, V4, V12 e V17.

# **Como rodar**

1. Baixe o dataset no Kaggle (link acima) e guarde o `creditcard.csv` em uma pasta local. O arquivo não está neste repositório por ser grande.
2. Instale as dependências: `pip install -r requirements.txt`
3. No notebook, troque o caminho do `read_csv` (que aponta para `/kaggle/input/...`) pelo caminho do seu CSV.

# **Limitações**
- V1 a V28 são componentes anônimas (PCA), então não dá para interpretá-las.
- O dataset não identifica clientes, então a análise é do padrão global de fraude, e não do comportamento individual de cada cliente.
- O conjunto de teste tem poucas fraudes (~148), então os resultados têm incerteza.


# **Tecnologias e por que usei**
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-013243?style=for-the-badge&logo=numpy&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-150458?style=for-the-badge&logo=pandas&logoColor=white)
![Matplotlib](https://img.shields.io/badge/Matplotlib-11557C?style=for-the-badge)
![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?style=for-the-badge&logo=scikitlearn&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white)
![Kaggle](https://img.shields.io/badge/Kaggle-20BEFF?style=for-the-badge&logo=kaggle&logoColor=white)


| Tecnologia | Por que usei |
|---|---|
| Python | Linguagem padrão em ciência de dados, com as bibliotecas de análise e de modelagem que o projeto precisava |
| NumPy | Cálculos numéricos: transformação logarítmica dos valores, faixas dos histogramas e densidades por classe |
| pandas | Carregar o dataset e manipular os dados em tabelas (separar classes, treino e teste, tabela de probabilidades) |
| Matplotlib | Gráficos de densidade, distribuição acumulada, curvas Precisão × Recall e ROC |
| scikit-learn | Divisão treino/teste, modelos (Naive Bayes, Regressão Logística e Random Forest), validação cruzada e métricas (PR-AUC e ROC-AUC) |
| Jupyter Notebook | Juntar código, gráficos e explicações no mesmo documento |
| Kaggle | Ambiente para rodar o notebook e acesso direto ao dataset |
- V1 a V28 são componentes anônimas (PCA), então não dá para interpretá-las.
- O dataset não identifica clientes, então a análise é do padrão global de fraude, e não do comportamento individual de cada cliente.
- O conjunto de teste tem poucas fraudes (~148), então os resultados têm incerteza.
