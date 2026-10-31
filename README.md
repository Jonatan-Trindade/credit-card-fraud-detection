# 💳 Detecção de Fraude em Cartão de Crédito

Projeto de Machine Learning para identificar transações fraudulentas no dataset
[Credit Card Fraud Detection](https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud) (Kaggle).

## O problema

Fraude é rara e cara. No dataset, só **492 de 284.807 transações (0,17%)** são fraude. Isso muda a forma de avaliar:

- Um modelo que diz "nunca é fraude" acerta 99,83% e não serve para nada. **Acurácia não é a métrica.**
- Cada fraude que passa é prejuízo. Cada alerta falso é um cliente incomodado e trabalho de revisão.
- Por isso a métrica principal é o **PR-AUC** (área sob a curva Precisão × Recall), com o ROC-AUC como referência. Um chute aleatório tem PR-AUC igual à proporção de fraude (~0,0017).

## Abordagem

1. **Baseline probabilístico:** teorema de Bayes usando só o valor da transação.
2. **Modelos:** Naive Bayes, Regressão Logística, Random Forest e CatBoost. Regressão Logística e Random Forest usam `class_weight="balanced"`; o CatBoost foi ajustado com **Optuna** (PR-AUC em validação cruzada).
3. **Sem vazamento:** divisão treino/teste 70/30 estratificada **antes** do pré-processamento (o `RobustScaler` aprende só no treino). O teste tem 85.443 transações e 148 fraudes e só é usado no final.
4. **Comparação por validação cruzada** (5 folds, dentro do treino): média e desvio do PR-AUC de cada modelo.
5. **Limite de alerta escolhido na validação cruzada**, sem olhar o teste, pela regra "maior recall mantendo pelo menos 80% de precisão".

Entradas dos modelos: `V1` a `V28` (anonimizadas por PCA) e `Amount` e `Time` escalados.

## Resultados

**Validação cruzada (5 folds, dentro do treino), PR-AUC:**

| Modelo | PR-AUC (média ± desvio) |
|---|---|
| Naive Bayes | 0,090 ± 0,011 |
| Regressão Logística | 0,762 ± 0,058 |
| Random Forest | 0,857 ± 0,019 |
| **CatBoost** | **0,869 ± 0,022** |

**Conjunto de teste (uma única avaliação):**

| Modelo | PR-AUC | ROC-AUC |
|---|---|---|
| Bayes (só o valor) | 0,005 | 0,745 |
| Naive Bayes | 0,080 | 0,955 |
| Regressão Logística | 0,705 | 0,968 |
| Random Forest | 0,820 | 0,938 |
| **CatBoost** | **0,835** | 0,977 |

Saber só o valor da transação quase não ajuda (0,005 contra 0,0017 do chute). As variáveis anonimizadas fazem toda a diferença.

**CatBoost e Random Forest empatam na prática:** a vantagem do CatBoost (0,012 na validação cruzada) é menor que o desvio entre folds (~0,02). O CatBoost levou a vantagem nos dois conjuntos, mas com 148 fraudes no teste não dá para afirmar que um é melhor que o outro.

**O `Time` não ajuda:** o CatBoost com e sem a variável de tempo tem o mesmo PR-AUC na validação cruzada (0,869 ± 0,022 contra 0,869 ± 0,024).

**Variáveis mais importantes no CatBoost:** `V4` se destaca (cerca de 8,6% da importância total), seguida de `V14`, `V1`, `V26`, `V10`, `V7`, `V20`, `scaled_amount`, `V12` e `V17`, cada uma entre ~3,6% e ~5,3%. As dez juntas somam cerca de metade da importância, ou seja, o modelo se apoia em muitas variáveis e não em uma só. O valor da transação (`scaled_amount`) aparece em 8º, e o `Time` fica fora do top 10, o que combina com ele não ter ajudado. Como V1 a V28 vêm de PCA, o ranking diz *quais* variáveis pesam, mas não o que elas representam.

**Recall (fraudes pegas) para cada precisão mínima dos alertas (validação cruzada):**

| Precisão mínima | Naive Bayes | Reg. Logística | Random Forest | CatBoost |
|---|---|---|---|---|
| 10% | 0,79 | 0,92 | 0,89 | 0,91 |
| 20% | 0,00 | 0,90 | 0,89 | 0,90 |
| 50% | 0,00 | 0,86 | 0,87 | 0,88 |
| 80% | 0,00 | 0,81 | 0,85 | 0,85 |
| 90% | 0,00 | 0,00 | 0,82 | 0,83 |

**Limite de alerta do CatBoost** (validação cruzada, 344 fraudes no treino):

| Limite | Alertas | Fraudes pegas | Alertas falsos | Fraudes que passaram | Precisão | Recall |
|---|---|---|---|---|---|---|
| 0,1 | 338 | 289 | 49 | 55 | 0,86 | 0,84 |
| 0,2 | 320 | 287 | 33 | 57 | 0,90 | 0,83 |
| 0,3 | 309 | 284 | 25 | 60 | 0,92 | 0,83 |
| 0,5 | 289 | 274 | 15 | 70 | 0,95 | 0,80 |
| 0,7 | 277 | 266 | 11 | 78 | 0,96 | 0,77 |

**No teste, com o limite 0,065** (escolhido na validação cruzada pela regra de 80% de precisão): 157 alertas, dos quais 121 eram fraude e 36 eram falsos, e 27 fraudes passaram. **Precisão 0,77 e recall 0,82.** A precisão ficou um pouco abaixo dos 80% buscados: com 157 alertas, essa oscilação é esperada, e é justamente o que a separação entre validação e teste revela.

## Limitações

- **Variáveis anonimizadas:** V1 a V28 vêm de PCA, então sabemos *quais* pesam, mas não o que significam no mundo real.
- **Poucos positivos no teste:** 148 fraudes. O PR-AUC no teste (0,835) ficou abaixo da validação cruzada (0,869), o que é compatível com essa variação, e por isso os dois números são reportados.
- **Ajuste favorece o CatBoost:** o Optuna usou o mesmo conjunto de treino da validação cruzada, então o CatBoost leva uma pequena vantagem sobre o Random Forest, que não foi ajustado.
- **Linhas idênticas:** o dataset original contém transações repetidas, e esta versão não as remove (`REMOVER_DUPLICATAS = False` no notebook). Se uma cópia cair no treino e outra no teste, as métricas podem sair infladas.
- **Janela curta:** os dados cobrem apenas dois dias de transações, sem validação temporal.

## Estrutura do repositório

```
├── notebooks/creditcardfraud.ipynb   # análise completa e comparação dos modelos
├── src/train.py                      # treina o CatBoost e salva modelo, scalers e limite
├── app/app.py                        # app Streamlit: envia CSV, recebe alertas
├── data/                             # coloque aqui o creditcard.csv (veja data/README.md)
├── models/                           # modelo treinado (não versionado)
└── requirements.txt
```

## Como rodar

```bash
pip install -r requirements.txt

# 1. Baixe o creditcard.csv do Kaggle e coloque em data/
# 2. Treine e salve o modelo (a escolha do limite usa validação cruzada, leva alguns minutos)
python src/train.py

# 3. Abra o app
streamlit run app/app.py
```

No app, envie um CSV no formato do dataset (`V1`..`V28`, `Amount` e `Time`). Ele devolve o score de fraude de cada transação e destaca os alertas, com um controle deslizante para o limite.

## Tecnologias

Python, pandas, NumPy, scikit-learn, CatBoost, Optuna, Matplotlib, Seaborn, Streamlit.
