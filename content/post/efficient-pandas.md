---
layout:     post
title:      "Efficient Pandas"
date:       2021-10-24
author:     "Luciano"
image:      "img/post_0_4.png"
tags:
    - Tutorial
    - Data Science
    - Pandas
categories: [ Tutorials ]
---

![efficient-pandas](/img/efficient-pandas/part1-2.gif)


Fala galera, tudo certo com vocês? Se você é um Cientista/Analista de dados, ou curte utilizar python para realizar diferentes análises de dados, certamente já utilizou a biblioteca `Pandas`. Pandas é uma biblioteca open-source para estruturação e análise de dados, seus comandos se assemelham muitas vezes ao SQL, porém sua API traz um conjunto maior operações, maior robustez e se executado de forma correta, mais parformance. 

Nesse tutorial estou me baseando em um minicurso do DataCamp [Writing Efficient Code with pandas](https://app.datacamp.com/learn/courses/writing-efficient-code-with-pandas). Como nem todos tem acesso a esse material, ou por não ter familiariedade com o inglês, pode ser de grande valia compartilhar esse conhecimento com a comunidade.

Bom, mas sem enrolação, aqui vamos tratar de 4 principais tópicos:
- Parte 1: Selecionando colunas e linhas de forma eficiente.
- Parte 2: Substituindo valores em colunas do `DataFrame`.
- Parte 3: Iterando no dataset de forma eficiente.
- Parte 4: Manipulação de dados com o `.groupby`, `.transform` e `.filter`.

Todos os tópicos serão abordados utilizando tarefas cotidianas de manipulação de dados. Estaremos utilizando uma base de dados pública oriunda do kaggle, que são os dados das Olimpíadas de Tokyo 2021. Você pode realizar o download dos arquivos pelo site ou clonar o repositório deste post, links abaixo:

- Kaggle: https://www.kaggle.com/arjunprasadsarkhel/2021-olympics-in-tokyo
- Github do post: 

Como bibliotecas estarei utilizando: `numpy`, `pandas`, `pyjanitor` e `time`. Certifique-se de ter todas elas instaladas no seu ambiente.

```python
import time

import numpy as np
import pandas as pd
from janitor import clean_names

# importing and cleaning the data
medals_df = pd.read_excel("Data/Medals.xlsx").clean_names()

# tidying the data
medals_df.drop(["total", "rank_by_total"], axis=1, inplace=True)
medals_reshaped_df = medals_df.melt(
    id_vars=["rank", "team_noc"],
    value_vars=["gold", "silver", "bronze"],
    var_name="medals",
    value_name="qtt",
)
```

Após importar os dados eu realizei algumas transformações nos dados:

- .clean_names: é um método do pyjanitor, que limpa o nome das colunas automaticamente.
- .melt: é um pivoteamento nos dados para o formato longer, melhor para análise.
- .drop: removi algumas colunas que não são necessárias nesse momento.

Como iremos calcular um comparativo várias vezes durante o tutorial, eu criei uma função para nos ajudar a calcular o quanto um método é mais rápido que o outro para executar uma função. Aqui entramos como input, com os valores do tempos gastos por cada método e depois calculamos o quanto um é maior que o outro.

```python
# helper function to calculate pct diff between the times
def calculate_how_much_diff(method_1: float, method_2: float) -> str:
    diff = method_2 / method_1

    if diff > 1:
        ans = f"Método 1 é {round(diff*100, 2)}% vezes mais rápido que o Método 2"
    elif diff < 1:
        new_diff = method_1 / method_2
        ans = f"Método 2 é {round(new_diff*100, 2)}% vezes mais rápido que o Método 1"

    return ans
```

Agora estamos prontos para começar!!

# Parte I: Selecionando colunas e linhas de forma eficiente

## .iloc e .loc

Essa é uma tarefa bem comum utilizada com DataFrames do Pandas, muitas vezes precisamos selecionar apenas uma porção dos dados, seja por colunas, seja por linhas. É ae que entra a utilização dos métodos `.iloc` e `.loc`.

- `.iloc`: Localizador numérico
- `.loc`: Localizador nominal

Existe diferença em utilizar um ou outro? Vamos decobrir:

```python
# timing how long will run each of those: .iloc and .loc
# loc time
start_time = time.time()
medals_reshaped_df.loc[:, ["team_noc", "medals", "qtt"]]
delta_loc = time.time() - start_time

# iloc time
start_time = time.time()
medals_reshaped_df.iloc[:, [1, 2, 3]]
delta_iloc = time.time() - start_time

# who is faster?
calculate_how_much_diff(method_1=delta_loc, method_2=delta_iloc)
```

```bash
'Método 2 é 219.69% vezes mais rápido que o Método 1'
```

```python
# loc time
start_time = time.time()
medals_reshaped_df.loc[1:30, :]
delta_loc = time.time() - start_time

# iloc time
start_time = time.time()
medals_reshaped_df.iloc[1:30, :]
delta_iloc = time.time() - start_time

# who is faster?
calculate_how_much_diff(method_1=delta_loc, method_2=delta_iloc)
```

```bash
'Método 2 é 169.56% vezes mais rápido que o Método 1'
```

Pelos resultados acima, conseguimos ver que em ambos os casos, utilizar o `.iloc` é muito mais eficiente para selecionar tanto linhas quanto colunas de um `DataFrame` do `Pandas`. Porém, é muito comum também autilizarmos apenas uma lista com o nome das colunas desejadas do dataset, vamos verificar então se o `.iloc` continua vitorioso nessa comparação:

```python
# iloc time
start_time = time.time()
medals_reshaped_df.iloc[:, 1:4]
delta_iloc = time.time() - start_time

# directly selecting columns
start_time = time.time()
medals_reshaped_df[["team_noc", "medals", "qtt"]]
delta_direct = time.time() - start_time

# who is faster?
calculate_how_much_diff(method_1=delta_iloc, method_2=delta_direct)
```

```bash
'Método 1 é 179.69% vezes mais rápido que o Método 2'
```

Como vimos, o `.iloc` ainda se mostra vitorioso nesta comparação. Neste tópico nós vemos uma divergência entre o que está no material do DataCamp e o que observamos em nosso código, pois lá o método de selecionar as colunas diretamente é mais eficiente.

Como toda biblioteca sofre atualizações e otimizações ao longo do tempo, é possível que o método `.iloc` tenha sido otimizado desde que o curso do DataCamp foi lançado.


## Amostragem aleatória

Outra tarefa bem comum dentro do dia-a-dia de Cientistas e Analistas de dados é a amostragem aleatória do nosso dataset, tal tarefa pode ter diferentes objetivos:

- diminuir o tamanho do dataset original
- validação de alguma técnica
- separação entre dados de teste e treino

Para isso, podemos prosseguir por duas formas diferentes com o Pandas:

- utilizar o módulo do numpy de geração de números randômicos e aplicar ao `DataFrame`
- utilizar o método `.sample` do Pandas

Vamos checar quem é mais eficiente:

```python
# numpy random integers
start_time = time.time()
medals_reshaped_df.iloc[np.random.randint(low=0, high=medals_reshaped_df.shape[0], size=50), :]
delta_numpy = time.time() - start_time

# using .sample
start_time = time.time()
medals_reshaped_df.sample(50, axis=0)
delta_sample = time.time() - start_time

# who is faster?
calculate_how_much_diff(method_1=delta_numpy, method_2=delta_sample)
```

```bash
'Método 2 é 192.69% vezes mais rápido que o Método 1'
```

Veja que o método do Pandas é muito mais efetivo, então, prefira utilizar o `.sample` caso precise de uma amostra aleatória a nível de linhas do seu dataset. Caso precise de selecionar amostras aleatórias a nível de colunas (menos comum), também podemos adaptar o que foi feito acima e verificar se o `.sample` continua vencendor.

```python
# numpy random integers
start_time = time.time()
medals_reshaped_df.iloc[:, np.random.randint(low=0, high=medals_reshaped_df.shape[1], size=3)]
delta_numpy = time.time() - start_time

# using .sample
start_time = time.time()
medals_reshaped_df.sample(3, axis=1)  # adjusting the axis = 1 for columns
delta_sample = time.time() - start_time

# who is faster?
calculate_how_much_diff(method_1=delta_numpy, method_2=delta_sample)
```

```bash
'Método 2 é 188.66% vezes mais rápido que o Método 1'
```

Ainda assim, temos o .sample como vencedor!

Com isso nós finalizamos a **Parte I** dessa série de tutoriais, e em resumo temos o seguinte:

- iloc é mais rápido que loc
- iloc é mais rápido que selecionar diretamente as colunas
- sample é mais rápido utilizar `numpy` para gerar um array de números aleatórios

Vamos à segunda parte!!

# Parte II: Substituindo valores em colunas do `DataFrame`

## Substituindo valores únicos utilizando o `.replace`

Você provavelmente deve ter cruzado com situações onde precisou substituir valores de algumas colunas baseando-se em algum condicional, algumas linguagens de programação chamam esse padrão de *switch case*, em python não possuíamos essa feature anterior a versão 3.10 (recém lançada). No Pandas, esse padrão já existia no método `.replace`, sendo assim, vamos analisar qual forma mais efetiva de realizar tal tarefa.

```python
# pure Pandas
medals_reshaped_replace_pd_df = medals_reshaped_df.copy()
start_time = time.time()
medals_reshaped_replace_pd_df.medals.loc[medals_reshaped_replace_pd_df.medals=="gold"] = 'Gold'
delta_numpy = time.time() - start_time

# using .replace
medals_reshaped_replace_df = medals_reshaped_df.copy()
start_time = time.time()
medals_reshaped_replace_df["medals"].replace("gold", "Gold")  # adjusting the axis = 1 for columns
delta_sample = time.time() - start_time

# who is faster?
calculate_how_much_diff(method_1=delta_numpy, method_2=delta_sample)
```

```bash
'Método 2 é 204.11% vezes mais rápido que o Método 1'
```
Como podemos ver, utilizar o `.replace` é incrivelmente mais rápido e pythônico que apenas fazer isso com o Pandas básico.


## Substituindo múltiplos valores com listas e dicionários

Uma outra comparação que podemos fazer é em relação a alteração de múltiplos valores numa columa de um `DataFrame`, e para isso o `.replace` nos oferece o recurso de utilizar listas e dicionários. Os quais veremos agora.

Antes de realizar os comparativos, eu montei um esquema que nos permite visualizar como funciona a sintaxe para o `.replace` com listas e dicinários

![Replace-lists](/img/efficient-pandas/replace-lists.png)
>Utilizando listas

![Replace-dict](/img/efficient-pandas/replace-dict.png)
>Utilizando dicionários

```python
# pure Pandas
medals_reshaped_replace_pd_df = medals_reshaped_df.copy()
start_time = time.time()
medals_reshaped_replace_pd_df["rank"].loc[
    (medals_reshaped_replace_pd_df["rank"] == 1)
    | (medals_reshaped_replace_pd_df["rank"] == 2)
    | (medals_reshaped_replace_pd_df["rank"] == 3)
] = "TOP 3"

medals_reshaped_replace_pd_df["rank"].loc[
    (medals_reshaped_replace_pd_df["rank"] != "TOP 3")
] = "OUTRAS"
delta_numpy = time.time() - start_time

# using .replace
medals_reshaped_replace_df = medals_reshaped_df.copy()
start_time = time.time()
max_rank = medals_reshaped_replace_df["rank"].max()
medals_reshaped_replace_df["rank"].replace(
    [[1, 2, 3], [range(4, max_rank + 1)]], ["TOP 3", "OUTRAS"], inplace=True
)
delta_sample = time.time() - start_time

# who is faster?
calculate_how_much_diff(method_1=delta_numpy, method_2=delta_sample)
```

```bash
'Método 2 é 136.03% vezes mais rápido que o Método 1'
```

Novamente, obtemos o mesmo resultado e de forma muito mais eficiente ao utilizar o `.replace` do pandas, mesmo quando queremos substituir valores baseando-se em mais de uma condição ao dataset.

Uma outra estrutura de dados que o Pandas nos permite usar é o dicionário, vamos realizar o comparativo:

```python
# pure Pandas
medals_reshaped_replace_df = medals_reshaped_df.copy()
start_time = time.time()
medals_reshaped_replace_df["medals"].replace("gold", "GOLD", inplace=True)
medals_reshaped_replace_df["medals"].replace("silver", "Silver", inplace=True)
medals_reshaped_replace_df["medals"].replace("bronze", "Bronze", inplace=True)
delta_numpy = time.time() - start_time

# using .replace
medals_reshaped_replace_df = medals_reshaped_df.copy()
start_time = time.time()
medals_reshaped_replace_df["medals"].replace(
    {"gold": "Gold", "silver": "Silver", "bronze": "Bronze"}, inplace=True
)
delta_sample = time.time() - start_time

# who is faster?
calculate_how_much_diff(method_1=delta_numpy, method_2=delta_sample)
```

```bash
'Método 2 é 171.75% vezes mais rápido que o Método 1'
```

Aqui nós vemos então, que até mesmo dentro do próprio método `.replace` é possível encontrar formas mais performáticas de realizar uma terefa, que nesse caso é por meio dos dicionários.

Com isso nós finalizamos a segunda parte do tutorial. Resumindo o que vimos até aqui, temos:

- `.replace` é muito mais efetivo que operações "*puras*" do `Pandas`
- `.replace` utilizando dicionários é mais efetivo que utilizando listas.


Fique ligado para o próximo post, onde será tratado das Parte III e Parte IV.
