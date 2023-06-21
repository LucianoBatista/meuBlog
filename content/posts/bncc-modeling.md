---
layout: post
title: "Classificador BNCC Pt.2"
subtitle: "Modelagem"
date: 2022-06-14
author: "Luciano, Pedro, Will e Brisa"
featuredImagePreview: "img/bncc_classifier.png"
tags:
  - Bag of words
  - TFIDF
  - N-Grams
  - Word2Vec
  - Data Science
  - NLP
  - Education
categories: [Projetos]
---

Resolvemos criar um novo artigo para detalhar melhor o que foi feito na etapa de modelagem do projeto do [Classificador BNCC](https://www.lobdata.com.br/2022/06/08/classificador-bncc/). Assim conseguimos dar mais atenção e justificar algumas escolhas.

## Cronograma de modelagem

Nosso intuito foi buscar o **melhor baseline** para nosso problema, mapeando alguns universos de possibilidades, dentre modelos de machine learning e estratégias de transformação de texto em dado numérico.

Válido lembrar que o dado que está entrando nessa etapa do pipeline está _"limpo"_, ou seja, passou pelas etapas de pré-processamentos que julgamos necessárias. Resumido abaixo:

```python
df["questions_clean"] = (
      df["questions"]
      .astype(str)
      .apply(html.unescape)
      .apply(lambda x: cleaning.remove_html(x))
      .apply(lambda x: x.lower())
      .apply(lambda x: cleaning.remove_punctuation_2(x))
      .apply(cleaning.remove_italic_quotes)
      .apply(cleaning.remove_open_quotes)
      .apply(cleaning.remove_end_quotes)
      .apply(cleaning.remove_italic_dquotes)
      .apply(cleaning.remove_open_dquotes)
      .apply(cleaning.remove_quote)
      .apply(lambda x: cleaning.remove_pt_stopwords(x))
      .apply(lambda x: cleaning.remove_en_stopwords(x))
      .apply(word_tokenize)
      .apply(lambda x: cleaning.remove_punctuation_2(x))
      )
```

Em questão de modelos, utilizamos:

- Regressão Logística
- Random Forest
- LightGBM

Além disso, para cada um dos modelos utilizamos diferentes tipos de feature engineering, apresentadas a seguir:

- Bag of words
- TFIDF
- N-grams
- Word2Vec

> OBS: Também tentamos utilizar o algorítimo de machine learning Gaussian Naive Bayes, mas por conta do mesmo não trabalhar com matriz esparsa (na implementação do Scikit-Learn), não tivemos recurso computacional para rodar o algorítimo. O dado como matriz não esparsa ocuparia +40 Gb em memória.

## Importância da validação

Como estamos trabalhando com **múltiplas classes** nos dois classificadores que estamos otimizando, e como também estamos buscando a solução **mais robusta possível**, é importante que utilizemos alguma estratégia de validação.

A biblioteca do `scikit-learn` oferece uma série métodos que podem ser utilizados para essa finalidade. Aqui, optamos por utilizar o [`StratifiedKFold`](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.StratifiedKFold.html) com 5 splits.

Esse método de validação preserva a proporção inicial de cada uma das classes envolvidas no target.

## Pipelines

Os baselines que irão ser utilizados abaixo foram configurados como o seguinte dicionário:

```python
# baselines
models = {
    "rg_lg": LogisticRegression(max_iter=500, n_jobs=8),
    "r_forest": RandomForestClassifier(max_depth=500, n_jobs=8),
    "lgbm": LGBMClassifier(n_jobs=8),
}

```

Dessa forma, conseguimos iterar em cada um dos modelos e automatizar nossas avaliações.

### Bag Of Words

O BoW (Bag of Words), ou saco de palavras, é uma técnica onde criamos um dicionário de palavras que contemplam nosso dataset, e contamos onde cada uma delas está presente ou não.

![bow](https://ichi.pro/assets/images/max/724/1*hLvya7MXjsSc3NS2SoLMEg.png)

```python
kfold = StratifiedKFold(n_splits=5, shuffle=True, random_state=2020)
for train_idx, val_idx in kfold.split(
    modeling.X_train["questions_clean"], modeling.y_train
):
    for name, model in models.items():
        x_train, y_train = (
            modeling.X_train["questions_clean"].iloc[train_idx],
            modeling.y_train.iloc[train_idx],
        )
        x_val, y_val = (
            modeling.X_train["questions_clean"].iloc[val_idx],
            modeling.y_train.iloc[val_idx],
        )

        # this will make our bag of words strategy
        count_vectorizer = CountVectorizer()
        count_vectorizer.fit(x_train)
        X_train_cv = count_vectorizer.transform(x_train)
        X_val_cv = count_vectorizer.transform(x_val)

        # lgbm does not work with int type of data, so
        # we need to convert to float to use
        if name == "lgbm":
            X_train_cv = X_train_cv.astype("float32")
            X_val_cv = X_val_cv.astype("float32")
            y_train = y_train.astype("float32")
            y_val = y_val.astype("float32")

        cv_classifier = model
        cv_classifier.fit(X_train_cv, y_train)
        y_pred = cv_classifier.predict(X_val_cv)
        f1 = f1_score(y_val, y_pred, average="macro")
        print("Model: {}. Macro avg F1: {}".format(name, f1))
```

Resultados abaixo:

| Modelos             | Macro Avg F1    |
| ------------------- | --------------- |
| Regressão Logística | 0.743 +/- 0.03  |
| Random Forest       | 0.657 +/- 0.05  |
| LightGBM            | 0.745 +/- 0.002 |

### N-Grams

Esta técnica visa realizar o agrupamento de tokens. O tamanho desse agrupamento é escolhido pelo valor do _N_:

- Uni: agrupamento um a um
- Bi: agrupamento dois a dois
- Tri: agrupamento três a três
- ...

![n-grams](https://images.deepai.org/glossary-terms/867de904ba9b46869af29cead3194b6c/8ARA1.png)

O interessante aqui é que conseguimos pegar um pouco de contexto, já que algumas palavras normalmente aparecem acompanhdas de outras. Por exemplo: _Pedro Álvares Cabral_, _Papai Noel_, _bom dia_...

Aqui nós utilizamos duas abordagens:

- Somente bi-gram
- uni-gram + bi-gram

Como podemos ver no código abaixo:

```python
kfold = StratifiedKFold(n_splits=5, shuffle=True, random_state=2020)
for train_idx, val_idx in kfold.split(
    modeling.X_train["questions_clean"], modeling.y_train
):
    for name, model in models.items():
        x_train, y_train = (
            modeling.X_train["questions_clean"].iloc[train_idx],
            modeling.y_train.iloc[train_idx],
        )
        x_val, y_val = (
            modeling.X_train["questions_clean"].iloc[val_idx],
            modeling.y_train.iloc[val_idx],
        )

        # the param ngram_range is responsible to set
        # how we'll want the n-grams. (1, 2) is setting
        # to bring uni and bi-gram combination.
        count_vectorizer = CountVectorizer(ngram_range=(1, 2))
        count_vectorizer.fit(x_train)
        X_train_cv = count_vectorizer.transform(x_train)
        X_val_cv = count_vectorizer.transform(x_val)

        if name == "lgbm":
            X_train_cv = X_train_cv.astype("float32")
            X_val_cv = X_val_cv.astype("float32")
            y_train = y_train.astype("float32")
            y_val = y_val.astype("float32")

        cv_classifier = model
        cv_classifier.fit(X_train_cv, y_train)
        y_pred = cv_classifier.predict(X_val_cv)
        f1 = f1_score(y_val, y_pred, average="macro")
        # print('ROC AUC - {}: {}'.format(name, mean))
        print("Model: {}. Macro avg F1: {}".format(name, f1))
```

Resultados apenas para o uni-gram + bi-gram:

| Modelos             | Macro Avg F1    |
| ------------------- | --------------- |
| Regressão Logística | 0.739 +/- 0.03  |
| Random Forest       | 0.652 +/- 0.05  |
| LightGBM            | 0.729 +/- 0.003 |

### TFIDF

![tfidf](https://miro.medium.com/max/1200/1*qQgnyPLDIkUmeZKN2_ZWbQ.png)

Como pode ser visto na imagem acima, aqui nós conseguimos identificar o quão importante cada palavra é, em relação ao todo que estamos tentando prever. Por exemplo: _quantas vezes a palavra soma aparece no texto de uma questão de matemática, frente a quantidade de vezes que ela aparece em todas as questões de matemática que temos na base?_

Utilizamos para isso, o código abaixo:

```python
kfold = StratifiedKFold(n_splits=5, shuffle=True, random_state=2020)
for train_idx, val_idx in kfold.split(
    modeling.X_train["questions_clean"], modeling.y_train
):
    for name, model in models.items():
        x_train, y_train = (
            modeling.X_train["questions_clean"].iloc[train_idx],
            modeling.y_train.iloc[train_idx],
        )
        x_val, y_val = (
            modeling.X_train["questions_clean"].iloc[val_idx],
            modeling.y_train.iloc[val_idx],
        )

        # here we are performing tfidf on training data
        # and choosing n-grams of 1 and 2
        tfidf = TfidfVectorizer(ngram_range=(1, 2))
        tfidf.fit(x_train)
        X_train_cv = tfidf.transform(x_train)
        X_val_cv = tfidf.transform(x_val)

        if name == "lgbm":
            X_train_cv = X_train_cv.astype("float32")
            X_val_cv = X_val_cv.astype("float32")
            y_train = y_train.astype("float32")
            y_val = y_val.astype("float32")

        cv_classifier = model
        cv_classifier.fit(X_train_cv, y_train)
        y_pred = cv_classifier.predict(X_val_cv)
        f1 = f1_score(y_val, y_pred, average="macro")

        print("Model: {}. Macro avg F1: {}".format(name, f1))
```

Resultados apenas para o uni-gram + bi-gram::

| Modelos             | Macro Avg F1     |
| ------------------- | ---------------- |
| Regressão Logística | 0.7151 +/- 0.006 |
| Random Forest       | 0.6609 +/- 0.010 |
| LightGBM            | 0.7230 +/- 0.007 |

### Word2Vec

Esse é modelo de _word embeddings_. Esse tipo de representação de texto busca identificar o significado de uma palavra no seu contexto (conotação). O Word2Vec foi um dos primeiros a aplicar esse tipo de conceito.

Com o Word2vec, foi possível, por exemplo, realizar a interpretação abaixo:

_King - Man + Woman = Queen_

Com o Word2vec nós podemos contornar o problema de dimensionalidade, e representar um texto qualquer em um número de features pre-determinado que melhor capte o contexto analisado. Abaixo você pode ver como fizemos.

```python
# creating the wv object
wv = Word2Vec(
    sentences=modeling.X_train["questions_clean"].apply(lambda x: x.split()),
    vector_size=100,
    window=3,
    min_count=1,
    workers=10
)

# function to transform those words
def transforma_palavra(question):
    lista_vetores = [wv.wv.get_vector(x) for x in question.split()]
    return np.sum(lista_vetores, axis=0)

# applying to those questions
vetores_embeddings = modeling.X_train["questions_clean"].apply(transforma_palavra)

# Criando um Dataframe com os resultados
df_embeddings = pd.DataFrame.from_dict(
    dict(zip(vetores_embeddings.index, vetores_embeddings.values))
).T
# Definindo os nomes das colunas
df_embeddings.columns = ["embedding_" + str(i) for i in range(1, 101)]

# Vamos também trazer o tweet original e o sentimento
df_embeddings["question"] = modeling.X_train["questions_clean"]
df_embeddings["question_target"] = modeling.X_train["target"].values

# splitting those embeddings
X_emb = df_embeddings[df_embeddings.columns[:100]]
y_emb = df_embeddings.question_target

X_train_emb, X_test_emb, y_train_emb, y_test_emb = train_test_split(
    X_emb, y_emb, test_size=0.2, stratify=y_emb
)


for nome, modelo in modelos_teste.items():
    metrica = cross_val_score(
        modelo, X_train_emb, y_train_emb, cv=3, scoring="f1_macro"
    ).mean()
```

| Modelos             | Macro Avg F1 |
| ------------------- | ------------ |
| Regressão Logística | 0.565        |
| Random Forest       | 0.551        |
| LightGBM            | 0.581        |

## Tuning

Após toda experimentação, vimos que o modelo mais promissor, em termos de métrica, tempo de processamento e simplicidade, foi a regressão logística em conjunto com o Bag of Words.

Daí em diante, realizamos o tuning do _parâmetro C_ da regressão logística, e conseguimos subir a acurácia para 0.8 em média dentre as classes, para o modelo que realiza a classificação do [**segundo classificador**](https://www.lobdata.com.br/2022/06/08/classificador-bncc/).

> Além de ajustar o C, nós também retiramos o `class_balanced`, pois o mesmo estava prejudicando a performance.

## Conclusão

Ainda existe muito espaço para melhoria do modelo e espaço para melhoria da aplicação como um todo. Continuaremos atualizando de acordo fomos avançado, _obrigado pela leitura._
