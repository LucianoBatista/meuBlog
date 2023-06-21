---
layout: post
title: "Classificador BNCC"
subtitle: "Classificador de questões do Ensino Médio e Fundamental"
date: 2022-06-08
author: "Luciano, Pedro, Will e Brisa"
featuredImagePreview: "img/bncc_classifier.png"
tags:
  - Tutorial
  - Data Science
  - NLP
  - Education
categories: [Projetos]
---

Projeto desenvolvido pelos alunos Brisa Rosatti, Luciano Batista, Pedro Moreau, Wilson França do Curso de Data Science & Machine Learning da Tera em colaboração com a Studos e ArcoTech.

## PARTE I: Estrutura do projeto

### Contexto

Segundo o IBGE, no Brasil, existem registradas 124840 escolas de ensino fundamental e 28933 escolas do ensino médio. Em termos de número de matrículas, isso representa 26718830 matrículas para o ensino fundamental e 7550753 para o ensino médio [https://cidades.ibge.gov.br/brasil/pesquisa/13/5908](https://cidades.ibge.gov.br/brasil/pesquisa/13/5908). Com o intuito de harmonizar a base curricular desse grande número escolas e estudantes, oferecendo assim um modelo de ensino de qualidade e que vise capacitar os alunos para as consecutivas fases da sua vida, o Ministério da Educação publicou, em 16 de setembro de 2015 a primeira versão da [Base Nacional Comum Curricular (BNCC)]~(http://basenacionalcomum.mec.gov.br/). Este documento normativo define o conjunto de aprendizagens que todos os alunos devem desenvolver ao longo da sua formação na Educação Básica.

A BNCC está estruturada em códigos alfanuméricos, que contemplam as competências a serem desenvolvidas em cada etapa da formação do aluno na Educação Básica. O código é dividido em quatro partes, como o exemplo EF 09 AR 01. O primeiro par de letras indica a qual etapa do ensino básico essa competência está inserida, neste caso, Ensino Fundamental. O primeiro par de números indica o ano ou bloco de anos em que esta competência está incluída, neste caso 9° ano. O segundo par de letras indica qual a qual área do conhecimento a competência pertence, AR = Artes e o segundo par de números mostra a posição da habilidade na numeração sequencial do ano ou bloco de anos referente à competência. As imagens abaixo apresentam outros exemplos da descrição dos códigos da BNCC.

![bncc-code](/img/bncc/codes-demo.png)

A partir da homologação do relatório final da BNCC em 2018, pelo Ministério da Educação, as escolas devem alinhar seus currículos em consonância às diretrizes estabelecidas pela BNCC. E isso impacta em um replanejamento de estruturas curriculares, alinhamento da coordenação pedagógica, treinamento de professores para que apliquem de forma eficiente as normativas às suas aulas e avaliações, entre outras. Todo esse esforço demanda tempo, investimento e realinhamento de uma estrutura de ensino já estabelecida, para que o novo programa de currículos seja aplicado de forma eficiente. Os conselhos de educação de cada estado, juntamente com as secretarias de educação, são responsáveis pela fiscalização da aplicação das normativas da BNCC em cada escola, sejam elas públicas ou privadas, já que a aplicação desta normativa é obrigatória a todas as escolas.

### Problema de negócio

É percebido que apesar da obrigatoriedade de seguir a BNCC, as Escolas geralmente não têm o devido preparo e professores não são bem capacitados para relacionar os códigos às questões aplicadas em provas e simulados. Essa classificação se faz necessária para que as avaliações contemplem as etapas de aprendizado previstas para cada fase do ensino básico, fundamental e médio. Uma das razões para esses problemas é que além do documento da BNCC ser extenso, possui mais de mil códigos em sua totalidade, tornando todo processo de classificação moroso e ineficiente.
Assim, vemos que três problemas de negócio emergem:

1. O primeiro está relacionado à própria escola do Ensino Básico, que, em função da necessidade de adequação curricular à BNCC, demanda uma ferramenta de acompanhamento das distintas disciplinas quanto ao cumprimento da base curricular reformulada, para que se cumpra a narrativa;

2. Além disso, há a necessidade de treinamento de professores para a implementação das normativas da BNCC às disciplinas às quais são responsáveis, visto que, em sua formação acadêmica, as normativas da BNCC não foram contempladas;

3. É sabido, ainda, que profissionais das Ciências da Natureza e Matemática apresentam dificuldade e resistência quanto à aplicação da BNCC em suas disciplinas, dado o caráter subjetivo de alguns códigos e habilidades, a falta de preparo em traduzir esses códigos em suas questões, durante a avaliação e a falta de experiência com esse tipo de abordagem curricular. Esta informação foi coletada a partir de entrevistas com profissionais da educação na cidade de Ilhéus - BA.

Diante desses problemas, foi proposta, neste trabalho de conclusão de curso, uma solução de dados que auxiliará as escolas e suas divisões pedagógicas quanto ao acompanhamento e cumprimento das normativas estabelecidas pela BNCC nas diferentes disciplinas e etapas dos ensinos fundamental e médio. Essa solução consiste em uma aplicação que irá classificar questões enviadas pelo usuário (profissional de educação) no formato texto, retornando para o mesmo os prováveis códigos correspondentes à BNCC. Dessa forma, o processo pedagógico poderá ser mediado por esta solução, trazendo celeridade à reorganização demandada pelo MEC. Além das escolas, EdTechs que tem como finalidade elaborar questões para compor serviços de bancos de questões também podem se valer desta solução de dados para fornecer as questões já classificadas de acordo à BNCC, agregando valor ao seu serviço.

### Impacto

#### Porque nosso projeto importa?

Acreditamos que o Projeto impactará duas áreas diferentes: o campo dos negócios e o pedagógico.
No quesito de negócios presumimos à diminuição da necessidade de profissionais envolvidos no processo de classificação de questões, diminuindo assim o custo da empresa e possibilitando que esses profissionais possam atuar de forma mais eficiente em diferentes demandas, como na elaboração de questões para o banco de dados, por exemplo.

No âmbito de vantagem competitiva, as empresas que incorporarem nossa tecnologia inovadora teriam um diferencial de mercado dentre outras EdTechs - startups focadas no desenvolvimento de soluções tecnológicas para a educação.

No campo pedagógico, a implementação do Classificador tornaria a aplicação da BNCC mais acessível aos profissionais da educação, possibilitando assim que as avaliações e testes dos currículos escolares estejam melhor alinhados à normativa do MEC. Desse modo, alcançaremos o principal objetivo do Projeto, no qual teremos um ensino mais direcionado e eficiente, impactando diretamente e de forma positiva o aprendizado estudantil.

Além disso, temos o intuito de tornar gratuito o acesso a essa tecnologia para profissionais de instituições públicas de ensino, sem que necessariamente haja investimento estatal ou necessidade de participação de licitações, contribuindo assim para a construção de um ensino básico de qualidade.

### Desenho da solução

Este projeto visa a implementação de um algoritmo de Machine Learning usando técnicas de Processamento de Linguagem Natural para classificação dos códigos da Base Nacional Comum Curricular (BNCC). Será utilizada uma abordagem de segmentação do problema em quatro partes, referentes às quatro partes do código BNCC. Dessa forma, será obtida, ao final de todas as etapas, a classe mais provável de cada parte do código da BNCC referente à questão que está sendo inserida no modelo para ser predita.

![classificadores](/img/bncc/classifyers.png)

Na presente data da publicação deste artigo, o grupo construiu os classificadores 1 e 3. Os classificadores 2 e 4 estão nos próximos passos para o trabalho, já que demandam uma base de dados maior e com o tageamento destas classes, que até o momento não foram disponibilizadas para o grupo.
Assim, para os Classificadores 1 e 3 a solução foi desenhada da seguinte forma:

- Extração dos dados a partir da base de dados da Studos
- Limpeza e pré-processamento dos dados
- Criação de um modelo de classificação para Área do Conhecimento e para Componente Curricular
- Produtização do modelo no Heroku
- Criação de um frontend no Streamlit
- Integração do modelo produtizado com o frontend para fornecimento do serviço a usuários finais

Como modelo de Machine Learning, foi adotada a Regressão Logística e, levando em conta a diferença de balanceamento entre as classes de cada target utilizada para os classificadores 1 e 3.

## PARTE II: Dados e solução

### Dados

Para uma melhor visibilidade da problemática envolvida neste classificador BNCC, o quadro abaixo apresenta algumas hipóteses levantadas para o desenvolvimento do produto, elucidando as fontes de dados, os dados utilizados e as demandas de negócios às quais se pretende atender.

#### Data Tracking Sheet

| Perguntas de Negócio                                                          | Dados                              | Base ou calculada | Cálculo   | Fonte de dado        |
| ----------------------------------------------------------------------------- | ---------------------------------- | ----------------- | --------- | -------------------- |
| Como classificar questões de acordo com a BNCC?                               | Questões previamente classificadas | Calculada         | á definir | Base dados da Studos |
| Como desenvolver raciocínio lógico/matemático nas escolas?                    | Questões Classificadas             | Calculada         | á definir | Base dados da Studos |
| Como melhorar a eficiência do ensino na sala de aula?                         | App com Questões Classificadas     | Calculada         | á definir | Base de dados Studos |
| Como ajudar o professor na elaboração de avaliações e atividades nas escolas? | App com Questões Classificadas     | Calculada         | á definir | Base de dados Studos |

#### Fonte

Como fonte de dados, iremos utilizar uma base disponibilizada pela Studos, uma EdTech onde o integrante do grupo Luciano trabalha. A base não contém informações sensíveis e conta com questões classificadas dentro dos códigos da BNCC, assim como informações necessárias para a criação dos quatro classificadores.

#### Datasets

Contamos com a liberação de +4M de questões classificadas com suas áreas de conhecimento e assuntos. Dessas, 8.4k possuem classificação das habilidades da BNCC. Além disso, tem acesso público ao documento da (BNCC) contendo informações específicas dos códigos.
Para o primeiro modelo de classificação foi utilizado a coluna ‘etapaEnsino’ como target, a qual designa cada questão como Ensino Médio, Fundamental I ou II. Para o segundo classificador será utilizado a coluna ‘matéria’ como target, na qual cada questão é classificada de acordo com o componente curricular.

O dataset contém as seguintes variáveis:

- id (int): identificador único de cada questão
- questoes (str): texto contendo o conteúdo da questão
- tipoQuestoes (int): tipo da questão, se foi múltipla-escolha, certo ou errado, discursiva, etc.
- topico (str): assunto do componente curricular
- materia (str): componente curricular
- etapaEnsino (str): área de conhecimento (Fundamental I, Fundamental II ou Ensino Médio)

A seguir tem-se uma visualiação do head do dataset utilizado.

![head](/img/bncc/table_head.png)

### EDA (Análise Exploratória de dados)

#### Targets

Nesse primeiro momento, nós estamos focados na criação dos classificadores 1 e 3. Foram coletados 100k de observações em que seguem a seguinte distribuição para a coluna Target ‘etapaEnsino’:

```python
data_pre_process['etapaEnsino'].value_counts()
```

```bash
Médio & Pré-Vestibular    35676
Fundamental II            33790
Fundamental I             18379
Name: etapaEnsino, dtype: int64
```

E para o Target do segundo Classificador temos 76K observações distribuídas em 12 componentes curriculares distintos conforme pode ser visto abaixo:

```python
# Observing if the filtering was correctly applied by visualizing the clases
df_bncc_copy_targets_bncc["target"].value_counts()
```

```
Matemática           12539
Língua Portuguesa    11887
Geografia             9266
História              9072
Inglês                8103
Ciências              7099
Física                5537
Arte                  3681
Química               3276
Biologia              3133
Educação Física       2386
Ensino Religioso       468
Name: target, dtype: int64
```

Sendo assim, nos resta analisar a variável principal do dataset, a coluna ‘questoes’. Apesar de ter uma série de análises que seria interessante realizar, observamos que a maior parte delas necessitaria de um texto o mais limpo possível.
O texto das questões são exibidos no formato web, então temos uma quantidade bem grande de tags html para remover. Então esse se tornou nosso primeiro passo do data cleaning. Seguimos então com uma remoção de números, pois existem muitos números indicando alternativas das questões, assim como em questões somatórias existe a contagem das respostas (00, 11, 22 ...), remoção de caracteres, de palavras raras e frequentes, entre outras técnicas descritas a seguir.

### Abordagem

#### MLOps e Configuração de Ambiente

O projeto de Ciência de Dados foi abordado pelo grupo com a intenção de utilizar conceitos de MLOps. Dessa forma, todo o projeto foi versionado utilizando GitHub neste link.
O repositório foi organizado em pastas que separam as funções utilizadas para o pré processamento, as análises realizadas e notebooks contendo os modelos treinados.

Todo este ambiente foi configurado utilizando pyenv, de forma que as bibliotecas pudessem ser reproduzidas localmente em outras máquinas sem correr risco de quebrar por diferença nas versões das bibliotecas e de Python utilizados.

#### Fluxo para o treinamento

Para implementação do algoritmo de classificação dos códigos na BNCC, seguem as seguintes etapas:

1. Coleta dos dados rotulados e não rotulados da Base de questões;
2. Separação dos dados em dois (treino e teste), então decidir as métricas de avaliação;
3. Pré-processamento: remoção de pontuação, lowercase das palavras, remoção de stopwords, stemming, lemmatization, remoção palavras raras, N-grams. O código do pipeline de pré processamento das questões pode ser visualizado nas imagens a seguir:

```python
import re
import string
from collections import Counter

import nltk
import pandas as pd
from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize

nltk.download("stopwords")
nltk.download("punkt")

PUNCT_TO_REMOVE = string.punctuation
pt_stopwords = set(stopwords.words("portuguese"))
en_stopwords = set(stopwords.words("english"))


def to_lower(text: str) -> str:
    return text.lower()


def remove_tags(text: str) -> str:
    pattern = re.compile("<.*?>")
    cleantext = re.sub(pattern, " ", text).replace("\xa0", " ")
    return cleantext


def remove_punctuation(text: str) -> str:
    return re.sub(r"[^\w\s]", " ", text)


def remove_numbers(text: str) -> str:
    pattern = re.compile("[0-9]+")
    clean_text = re.sub(pattern, " ", text)
    return clean_text


def remove_standard_stopwords(text: str) -> str:
    stop_words = set(stopwords.words("portuguese"))
    word_tokens = word_tokenize(text)
    filtered_sentence = [w for w in word_tokens if w not in stop_words]
    final_sentence = " ".join(filtered_sentence)
    return final_sentence


def remove_html(text):
    html_pattern = re.compile("<.*?>")
    return html_pattern.sub(r"", text)


def remove_punctuation_2(text):
    return text.translate(str.maketrans("", "", PUNCT_TO_REMOVE))


# Função para remover aspas em itálico
def remove_italic_dquotes(text: str) -> str:
    pattern = re.compile(r'"')
    clean_text = re.sub(pattern, " ", text)
    return clean_text


# Função para remover aspas de abertura
def remove_open_dquotes(text: str) -> str:
    pattern = re.compile(r"“")
    clean_text = re.sub(pattern, " ", text)
    return clean_text


# Função para remover aspas de fechamento
def remove_end_dquotes(text: str) -> str:
    pattern = re.compile(r"”")
    clean_text = re.sub(pattern, " ", text)
    return clean_text


# Função para remover single quotes normal
def remove_italic_quotes(text: str) -> str:
    pattern = re.compile(r"'")
    clean_text = re.sub(pattern, " ", text)
    return clean_text


# Função para remover single quotes abertura
def remove_open_quotes(text: str) -> str:
    pattern = re.compile(r"‘")
    clean_text = re.sub(pattern, " ", text)
    return clean_text


# Função para remover single quotes fechamento
def remove_end_quotes(text: str) -> str:
    pattern = re.compile(r"’")
    clean_text = re.sub(pattern, " ", text)
    return clean_text


# Função para remover aspas de abertura
def remove_quote(text: str) -> str:
    pattern = re.compile(r"‛")
    clean_text = re.sub(pattern, " ", text)
    return clean_text


def remove_pt_stopwords(text):
    return " ".join([word for word in str(text).split() if word not in pt_stopwords])


def remove_en_stopwords(text):
    return " ".join([word for word in str(text).split() if word not in en_stopwords])


class RemoveFrqRare:
    def __init__(self, df: pd.DataFrame, n_frq_words: int = 10, n_rare_words: int = 10):
        self.df = df
        self.n_frq_words = n_frq_words
        self.n_rare_words = n_rare_words
        self.frq_words = []
        self.rare_words = []

    def calc_frq_words(self):
        final_list = [
            subitem for item in self.df["questions_clean"] for subitem in item
        ]
        cnt = Counter(final_list)
        self.frq_words = set([w for (w, wc) in cnt.most_common(self.n_frq_words)])
        # print(self.frq_words)

    def calc_rare_words(self):
        # joining all words
        final_list = [
            subitem for item in self.df["questions_clean"] for subitem in item
        ]
        cnt = Counter(final_list)
        self.rare_words = set(
            [w for (w, wc) in cnt.most_common()[: -self.n_rare_words - 1 : -1]]  # noqa
        )
        # print(self.rare_words)

    def remove_frq_words(self, text: str):
        return " ".join(
            [word for word in str(text).split() if word not in self.frq_words]
        )

    def remove_rare_words(self, text: str):
        return " ".join(
            [word for word in str(text).split() if word not in self.rare_words]
        )

    def remove_frq_and_rare(self):
        df_to_return = self.df.copy()
        df_to_return["questions_clean"] = (
            self.df["questions_clean"]
            .apply(lambda x: self.remove_frq_words(x))
            .apply(lambda x: self.remove_rare_words(x))
        )
        return df_to_return

```

4. Modelagem: Transformar o texto em feature vectors - Feature Extraction (representação matemática da linguagem). Nessa etapa foi utilizado o Bag of Words;

Após a limpeza dos dados utilizando as funções apresentadas acima, foi realizada a vetorização desse textos para que pudessem ser modelados.

Criamos uma classe para contemplar todas as etapas de modelagem, incluindo o save do pipeline. Isso nos ajudou no futuro quando entramos na produtização.

Com os vetores obtidos, foi possível treinar um modelo de regressão logística para a classificação das questões quanto à etapa de aprendizado (classificador 1) e quanto a área de conhecimento (classificador 3).

```python
class Modeling:
    def __init__(self, data: pd.DataFrame) -> None:
        self.data = data
        self.X_train = None
        self.X_test = None
        self.y_train = None
        self.y_test = None

    def tokenization(self):
        df = self.data.copy()
        df["question_tokens"] = df["questions_clean"].apply(word_tokenize)
        self.data = df.copy()

    def split_data(self):
        X = self.data.drop(["id", "words_counts", "target_enc"], axis=1)
        y = self.data["target_enc"]
        (
            self.X_train,
            self.X_test,
            self.y_train,
            self.y_test,
        ) = model_selection.train_test_split(X, y, random_state=1)

    def saving_pipeline(self):

        pipe = Pipeline(
            [
                ("count_vect", feature_extraction.text.CountVectorizer()),
                (
                    "logistic_regression",
                    linear_model.LogisticRegression(class_weight="balanced"),
                ),
            ]
        )
        pipe.fit(self.X_train["questions_clean"], self.y_train)

        # saving
        with open(
            "/models/lr_second_model.bin",
            "wb",
        ) as lr_out:
            pickle.dump(pipe, lr_out)

    def make_bag_of_words(self):
        vect_bow = feature_extraction.text.CountVectorizer()
        X_train_trans = vect_bow.fit_transform(self.X_train["questions_clean"])
        X_test_trans = vect_bow.transform(self.X_test["questions_clean"])

        return X_train_trans, X_test_trans

    def train_evaluate_log_reg(self, X_train_trans, X_test_trans):
        log_reg = linear_model.LogisticRegression(class_weight="balanced")
        log_reg.fit(X_train_trans, self.y_train)
        y_pred_class = log_reg.predict(X_test_trans)
        print(metrics.classification_report(self.y_test, y_pred_class))
```

6. A performance dos modelos pode ser vista a seguir.

As métricas para o Classificador 1 foram:

```bash
              precision    recall  f1-score   support

           0       0.63      0.80      0.70      4575
           1       0.65      0.62      0.64      8497
           2       0.76      0.70      0.73      8890

    accuracy                           0.69     21962
   macro avg       0.68      0.71      0.69     21962
weighted avg       0.69      0.69      0.69     21962
```

E as métricas para o Classificador 2 foram:

```bash
              precision    recall  f1-score   support

           0       0.72      0.75      0.74       991
           1       0.64      0.66      0.65       760
           2       0.62      0.62      0.62      1816
           3       0.66      0.76      0.71       586
           4       0.28      0.49      0.36       104
           5       0.86      0.84      0.85      1285
           6       0.78      0.73      0.75      2385
           7       0.80      0.76      0.78      2263
           8       0.80      0.90      0.84      1990
           9       0.85      0.83      0.84      2896
          10       0.89      0.86      0.88      3194
          11       0.78      0.76      0.77       842

    accuracy                           0.78     19112
   macro avg       0.72      0.75      0.73     19112
weighted avg       0.79      0.78      0.79     19112
```

Foi observado que o valor do F1-Score médio para o classificador 1 foi de 0,69 e para o classificador 2 foi de 0.78. Essa métrica foi adotada pelo grupo como fator decisivo para continuação do projeto. Levando em conta que o F1-score balanceia a precisão e recall, essa métrica foi utilizada para que seja levado em conta o desbalanceamento do dataset. **Já que as métricas foram satisfatórias para os dois modelos, foi escolhido seguir com a produtização dos mesmos sem que se investisse tempo, nesse momento, em otimização de hiperparâmetros dos mesmos**.

7. Deploy do modelo

A produtização do modelo foi feita da seguinte forma:

- Todas as etapas descritas anteriormente foram organizadas no repositório do [github](https://github.com/LucianoBatista/bncc-classifier), sendo os códigos organizados em arquivos .py para que sejam separados o pipeline de pré-processamento, o pipeline de treinamento e predição do classificador 1 e também do classificador 3.

- Em seguida, após o treinamento, arquivos pickle contendo os transformes de pré-processamento e dos modelos foram gerados. Estes arquivos foram utilizados para a criação de uma API Rest utilizando o [framework em Python do FastAPI](https://bncc-classifier.herokuapp.com/docs). Esta API foi encapsulada em um Dockerfile, para que pudesse ser produtizada no Heroku e as rotas de inferência dos modelos fossem expostas para serem utilizadas em uma interface.

- Para deploy no Heroku utilizamos uma sequência recomendada pela própria [documentação da plataforma](https://devcenter.heroku.com/articles/container-registry-and-runtime).

- O frontend foi feito no Streamlit, como será comentado a seguir. O link do código para o frontend está no [repositório do github](https://github.com/LucianoBatista/bncc-classifier-front).

## PARTE III: Colocando a solução à prova

### Interface

Para implementação da solução como um todo, construímos uma API (Application Programming Interface), que foi desenvolvida utilizando FastAPI, um framework em python. Essa API, funciona para servir o modelo de classificação para quem desejar utilizá-lo.

Como inputs, a API vai inicialmente receber apenas texto, com possibilidade de aumento de complexidade para receber também imagens ou extração de texto de documentos words em etapas futuras deste trabalho. E como outputs, retornaremos ao usuário os mais prováveis códigos da BNCC para aquela questão.

O usuário final, o profissional de educação, irá interagir com a API através de uma tela interativa criada a partir do framework Streamlit. A imagem a seguir demonstra esta interface.

![app](/img/bncc/app.png)

Este frontend tem o objetivo de ser prático para o usuário, consistindo em um campo onde o usuário pode inserir a questão que deseja analisar e teclar “Enter” para que esta questão seja carregada no modelo. Assim, aparecerá um botão Predict! que exporá as classes mais prováveis para o Componente Curricular e Etapa do Ensino, como apresentado na imagem anterior para a questão de exemplo, sendo o componente curricular Física e a etapa do ensino Ensino Mérico e Pré-Vestibular.

![app](/img/bncc/predict.png)

## Implicações e próximos passos

A solução de dados proposta foi desenhada para classificar questões de Ensino Fundamental I e II e Ensino Médio quanto às normas da BNCC. Para isso, uma base de dados previamente classificada foi utilizada no treinameto. Visto que essa classificação foi realizada utilizando questões escritas na Língua Portuguesa (com exceção das questões em inglês) e seguindo a norma culta, algumas instruções e premissas emergem destes fatos:

### Instruções

Para a utilização eficiente desta solução de dados, é recomendada a utilização de questões estruturadas na Língua Portugesa, que contemplem conteúdos abordados nas etapas de ensino comentadas anteriormente e que estejam escritas seguindo a norma culta.
A forma de utilização desta solução é pela inserção de questões, individualmente, no campo de texto da interface da solução. A solução não contempla, até o momento, a classificação de múltiplas questões ao mesmo tempo.

O não cumprimento dessas instruções pode ocasionar em falhas de previsão além das relacionadas às métricas do modelo, mas devido à má utilização da solução, como por exemplo devido a questões que de assuntos que não são contemplados nos currículos escolares ou pela utilização de linguagem cotidiana, como gírias e expressões idiomáticas.

### Implicações

Analisando criticamente os impactos dessa solução e externalidades que possam afetar as mesmas, principalmente devido a natureza dos dados coletados, os seguintes pontos estão associados ao classificador BNCC:

- Impacto positivo sobre ganho de eficiência no cumprimento da BNCC pelos profissionais da educação, sejam professores ou profissionais atrelados à gestão pedagógica de alguma instituição de ensino.

- Ganho de eficiência por parte de EdTechs que prestam serviço de oferecimento de questões a instituições de ensino, possibilitando uma classificação prévia desse banco de questões quanto aos códigos da BNCC.

- Um possível impacto negativo seria a desalocação de profissionais que antes eram responsáveis pela classificação de questões caso a solução de dados seja empregada para este propósito. Porém, como uma alternativa para remediar este problema é recomendado que estes profissionais sejam direcionados para a criação de novas questões para alimentação do banco de questões e supervisão da classificação gerada pela solução de dados.

### Possíveis fontes de viés

Dado que a solução foi desenvolvida utilizando uma base de questões previamente classificadas por professores que compõem um fórum específico (forum da empresa Studos), os dados possuem o viés de que a classificação depende da habilidade destes professores em classificar as questões e, no caso de uma futura classificação da competência específica (classificador 4) que esta questão aborda, há um viés da subjetividade do tema.

Analisando este último fato criticamente, dada a provável subjetividade da competência abordada na questão e que essa classificação por parte dos professores pode ter influência de variáveis que descrevem os professores que o modelo não controla, como por exemplo variáveis demográficas, nível de formação, etnia, fonte de renda, quantidade de bens, nível de formação dos seus pais, entre outras.

Diante disso, levanta-se o seguinte questionamento: será que a classificação de questões, por parte de professores, será sempre igual? Será que professores com diferentes condições de vida e de diferentes contextos classificarão as questões da mesma forma, ou existe uma margem de erro atrelada a estas variáveis não controladas, que explicam o contexto dos professores?

Além disso, dada essa provável diferença de classificação, será que a BNCC é a melhor forma de se abordar uma estruturação curricular a nível de ensino básico? Ou a fonte do problema da educação no país é outra, que não o currículo em si, como por exemplo o incentivo salarial aos professores?

## Conclusão

Como conclusões, o grupo entende que é possível utilizar soluções de Machine Learning com fins de aplicação na educação. E mais especificamente voltado a emprego de Processamento de Linguagem Natural na classificação de textos segundo um target que se deseja estudar, no nosso caso os códigos da BNCC.

Durante o desenvolvimento da solução, percebemos o que existem diversas oportunidades de emprego de Machine Learning para educação. No contexto de questões e avaliações, além da classificação de questões nos códigos da BNCC, uma possível solução seria a partir de códigos ou de competências que se deseja trabalhar no ensino dos alunos, uma solução baseada em dados recomendaria uma lista de questões que contemple estes códigos ou competências desejadas. Isso tornaria possível os dois cenários, tanto classificar quanto recomendar questões segundo a BNCC.

Como próximos passos para este projeto, o grupo visa continuar com a obtenção de dados que viabilize o desenvolvimento dos classificadores 2 e 4, complementando assim com as demais partes do código BNCC, trazendo a classificação completa. Diante disso, há a etapa de atualização da produtização desta solução de dados e atualização da interface no Streamlit.

Um outro projeto de interesse da equipe é a recomendação de questões baseada no input do código BNCC ou competência/habilidade que se deseja trabalhar.

O grupo agradece seu tempo e leitura!
