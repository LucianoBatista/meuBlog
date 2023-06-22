---
layout: post
title: "Modelagem Preditiva em IoT"
date: 2019-04-04
author: "Luciano"
featuredImagePreview: "img/iotdevice.png"
tags:
  - Project
  - Data Science
  - Machine Learning
categories: [Projetos]
---

# Previsão de Uso de Energia

Quando falamos de revolução tecnológica, a noção de Internet das Coisas, ou Internet of Things (IoT), é um dos assuntos principais. O seu desenvolvimento está mudando a forma como o ser humando tem se conectado com diversos dispositivos tecnlógicos.

IoT vem gerando uma quantidade imensa de dados, um dos maiores responsáveis pelo Big Data. Diante da geração de tanta informação, o papel do Cientista de Dados se faz cada vez mais presente, pois torna-se possível a obteção de grandes insights sobre diferentes área até então não analisadas.

Este projeto de IoT tem como objetivo a criação de modelos preditivos para a previsão de consumo de energia de eletrodomésticos. Os dados utilizados incluem medições de sensores de temperatura e umidade de uma rede sem fio, previsão do tempo de uma estação de um aeroporto e uso de energia utilizada por luminárias.

A versão completa do projeto pode ser encontrada no repositório do meu [GitHub](https://github.com/LucianoBatista/iot_energy_consumption_prediction), assim como os dados que foram utilizados.

# Análise Exploratória dos Dados

## O que representa cada variável no dataset?

- **date:** Tempo de coleta dos dados pelos sensores
- **Appliances:** Uso de energia (em W)
- **lights:** Potência de energia de eletrodomésticos na casa (em W)
- **TX:** Temperatura em um lugar da casa (em Celsius)
- **RH_X:** Umidade relativa em algum ponto da casa (em %)
- **Windspeed:** Velocidade do vento (em m/s)
- **Visibility:** Visibilidade (em Km)
- **Tdewpoint:** Não foi informado.
- **rv1:** Variável randômica adicional
- **rv2:** Variável randômica adicional
- **WeekStatus:** Indica se é dia de semana ou final de semana
- **Day_of_week:** Dia da semana
- **NSM:** Medida de tempo (em s)

Ao importar os dados foi visto que os dados possuem a mesma quantidade de colunas e que elas são iguais, sendo assim, os dados de teste e treino foram concatenados para realizar uma análise conjunta dos dados.

```python
df = pd.concat([train, test], axis=0)
```

```python
print(f'O novo dataset possui {df.shape[0]} linhas e {df.shape[1]} variáveis')
print(f'O dataset possui {sum(df.isnull().sum())} valores nulos')
```

    O novo dataset possui 19735 linhas e 32 variáveis
    O dataset possui 0 valores nulos

Optei por separar variáveis numéricas e categóricas para melhor observar algumas informações que são características de cada tipo.

```python
num_vars = [name for name in df.columns if df[name].dtype != 'object']
cat_vars = [name for name in df.columns if df[name].dtype == 'object']
df_num = df[num_vars]
df_cat = df[cat_vars]
print(f'O data set possui {len(num_vars)} variáveis numéricas e {len(cat_vars)} de tipo objeto, sendo todas relacionadas à data')
```

    O data set possui 29 variáveis numéricas e 3 de tipo objeto, sendo todas relacionadas à data.

Uma observação interessante é que precisamos estar sempre atentos ao tipo de dado que está sendo manipulado, no atual conjunto de dados a variável _'date'_ não foi importada como tipo de data. Logo, foi preciso realizar essa conversão, além disso, a data (agora convertida) foi adicionada ao index do DataFrame.

A adição da variável de data ao index do objeto DataFrame do pacote Pandas permite uma manipulação mais intuitiva desse tipo de dado. Mais à frente tal funcionalidade será observada.

```python
# converter a coluna 'date' para tipo data
df_cat['date'] = pd.to_datetime(df_cat['date'])
```

```python
df_cat.dtypes
```

    date           datetime64[ns]
    WeekStatus             object
    Day_of_week            object
    dtype: object

```python
# Transformando a coluna de data em index podemos realizar algumas análises
# de forma mais simples.
df_cat.index = pd.DatetimeIndex(df_cat['date'])
df_num.index = pd.DatetimeIndex(df_cat['date'])
```

```python
# Também precisaremos de uma coluna com as datas (utilizada para alguns plots).
df_num['date'] = df_cat['date'].copy()
```

```python
df_num.columns
```

    Index(['Appliances', 'lights', 'T1', 'RH_1', 'T2', 'RH_2', 'T3', 'RH_3', 'T4',
           'RH_4', 'T5', 'RH_5', 'T6', 'RH_6', 'T7', 'RH_7', 'T8', 'RH_8', 'T9',
           'RH_9', 'T_out', 'Press_mm_hg', 'RH_out', 'Windspeed', 'Visibility',
           'Tdewpoint', 'rv1', 'rv2', 'NSM', 'date'],
          dtype='object')

```python
df_cat.columns
```

    Index(['date', 'WeekStatus', 'Day_of_week'], dtype='object')

---

## Agora vamos observar um conjunto de plots que ajudaram a entender melhor os dados.

## Pairplot

O código abaixo permite a visualização de um pairplot filtrado por períodos. È possível visualizar os dados em diferentes intervalos de dias, semanas, meses e ano (caso nosso dataset tivesse dados de um ano diferente à 2016).

```python
# Precisa editar algumas linhas caso precise plotar um dia específico
# ou intervalo específico.
year_start = '2016'
month_start = '05'
day = int('01')
# fim
year_end = '2016'
month_end = '05'

if day < 10:
    # date_start = year_start + '-' + month_start + '-' + '0' + str(day_range) + ' 00:00:00'
    date_start = year_start + '-' + month_start
    date_end = year_end + '-' + month_end + '-' + '0' + str(day) + ' 23:00:00'
else:
    # date_start = year_start + '-' + month_start + '-' + str(day_range) + ' 00:00:00'
    date_start = year_start + '-' + month_start
    date_end = year_end + '-' + month_end + '-' + str(day) + ' 23:00:00'

print(f'{date_start} até {date_end} com {len(df_num[date_start])} pontos')

sns.set(style="ticks", color_codes=True)
df_filter = df_num[date_start] # date_start:date_end
g = sns.pairplot(df_filter)
plt.show()
```

    2016-05 até 2016-05-01 23:00:00 com 3853 pontos

![png](/img/output_21_1.png)

Podemos visualizar que a maioria das distribuição se assemelham à uma distribuição normal, sendo que algumas poucas possuem desvios. Vamos verificar agora se existe a presença de muitos outliers nos dados

## Boxplot

```python
# Novamente, cada plot tem um código que permite o filtro por data.
year_start = '2016'
month_start = '04'
day = int('01')
# fim
year_end = '2016'
month_end = '05'

if day < 10:
    date_start = year_start + '-' + month_start
    date_end = year_end + '-' + month_end + '-' + '0' + str(day) + ' 23:00:00'
else:
    date_start = year_start + '-' + month_start
    date_end = year_end + '-' + month_end + '-' + str(day) + ' 23:00:00'

print(f'{date_start} até {date_end} com {len(df_num[date_start])} pontos')

plt.subplots(figsize=(30, 20))
sns.set(style="ticks", color_codes=True)
df_filter = df_num[date_start]
df_filter = df_filter.drop(['date', 'NSM'], axis=1)
df_filter.boxplot()
plt.show()
```

    2016-04 até 2016-05-01 23:00:00 com 4320 pontos

![png](/img/output_24_1.png)

Os dados que mais possuem outliers é a variável target. Nesse caso é essencial que esse pontos sejam tratados, o que será feito mais a frente.

## Plot Comparativo

```python
# Gráficos comparativo de qualquer variável ao longo de um período
# Determine o período de início e fim que deseja visualizar
# E determine a variável name com a variável a ser observada
# início
year_start = '2016'
month_start = '04'
day = '01'
# fim
year_end = '2016'
month_end = '04'

# plot automatizado
# n = número de subplots (dias para visualizar)
n = 10
name = 'Appliances'
fig, ax = plt.subplots(n, figsize=(10, 70))
for i in range(0, n):
    day_range = int(day) + i
    if day_range < 10:
        date_start = year_start + '-' + month_start + '-' + '0' + str(day_range) + ' 00:00:00'
        date_end = year_end + '-' + month_end + '-' + '0' + str(day_range) + ' 23:00:00'
    else:
        date_start = year_start + '-' + month_start + '-' + str(day_range) + ' 00:00:00'
        date_end = year_end + '-' + month_end + '-' + str(day_range) + ' 23:00:00'
    dates, values = zip(*sorted(zip(df_num['date'],df_num[date_start: date_end][name])))
    ax[i].plot_date(dates,values, '-')
    print(f'{date_start} até {date_end} com {len(df_num[date_start: date_end][name])} pontos')
```

![png](/img/output_27_2.png)

Com o código acima é possível visualizar o comportamento de qualquer variável numérica em relação ao tempo. Por conta de ser um post no blog, eu optei em apenas mostrar uma das figuras criadas pelo código. Mas você pode acessar o [link](https://github.com/LucianoBatista/iot_energy_consumption_prediction/blob/master/modelagem_preditiva_iot.ipynb) para visualizar a saída completa dessa célula.

Podemos observar o seguinte:

- **temperatura interna:** Possui valores de 20 a 25, acredito que graus Celsius, tendo picos durante a noite. Período em que provavelmente o aquecedor está ligado.
- **temperatura externa:** Possui valores baixos, categorizando um país frio que é a Bélgica. Também apresenta variação de acordo o por e nascer do sol
- **RH e RH_out:** Apresenta uma variabilidade interessante para explicar o comportament do Appliances.
- **Pressão:** Apresenta um comportamento bem padrão.
- **rv_1, rv_2:** Possuem um comportamento bem aleatório ao longo do tempo, já que se trata realmente de uma variável aleatória adicionada aos dados.

## FacetGrid

Este é um tipo de gráfico que permite a visualização do comportamento de mais de duas variáveis ao mesmo tempo. Na figura, optei por visualizar a variável _'Appliances'_ em relação à _'rv1'_ e _'T_out'_.

```python
# Aqui basta alterar o intervalo desejado e as variáveis desejadas
g = sns.FacetGrid(df_num['2016-05'], col="Appliances", col_wrap=8, height=2, ylim=(0, 50))
g.map(sns.scatterplot, "rv1", "T_out", ci=None);
```

![png](/img/output_31_0.png)

Vemos que a maior varialibilidade dos dados se dá entre 20 a 130 do Appliance. Logo a modelagem pode ser favorecida a partir da remoção de outliers.

# Feature Engineering

## Tratamento de outliers

Como vimos que o 'Appliances' possui muito valores outliers. Podemos iniciar o tratamento pela elimação desses valores.

Para isso, identifiquei o Interquantile Range para remvover o whisker superior do boxplot. Como pode ser visto no código a seguir.

```python
# Primeiro e terceiro quartil e cálculo da distância IQR
Q1 = df_num['Appliances'].quantile(0.25)
Q3 = df_num['Appliances'].quantile(0.75)
IQR = Q3 - Q1
print(Q1)
print(Q3)
print(IQR)
```

    50.0
    100.0
    50.0

```python
# Determinação dos ranges inferiores e superiores do boxplot
Lower_Whisker = Q1 - 1.5*IQR
Upper_Whisker = Q3 + 1.5*IQR
print(Lower_Whisker, Upper_Whisker)
```

    -25.0 175.0

```python
# Remoção dos dados outiliers
new_df = df_num[df_num['Appliances'] < 175.0]
```

```python
year_start = '2016'
month_start = '04'
day = int('01')
# fim
year_end = '2016'
month_end = '05'

if day < 10:
    date_start = year_start + '-' + month_start
    date_end = year_end + '-' + month_end + '-' + '0' + str(day) + ' 23:00:00'
else:
    date_start = year_start + '-' + month_start
    date_end = year_end + '-' + month_end + '-' + str(day) + ' 23:00:00'

print(f'{date_start} até {date_end} com {len(new_df[date_start])} pontos')

plt.subplots(figsize=(30, 20))
sns.set(style="ticks", color_codes=True)
df_filter = new_df[date_start]
df_filter = df_filter.drop(['date', 'NSM'], axis=1)
df_filter.boxplot()
plt.show()
```

    2016-04 até 2016-05-01 23:00:00 com 3870 pontos

![png](/img/output_39_1.png)

```python
# Removendo variável de data, adicionada para ajudar na construção dos gráficos
new_df_num = new_df.drop(['date'], axis=1)
```

```python
# Resetando o index
new_df_num.reset_index(drop=True, inplace=True)
new_df_num.index
```

    RangeIndex(start=0, stop=17597, step=1)

## Correlation

Aqui foi observado como se dá a relação entre as variáveis. Particularmente se existe uma correlação linear entre as variáveis. A correlação é uma medida que varia de 0 a 1, onde valores próximos a 0 possuem uma correlação negativa e próximos a 1 uma correlação positiva.

A correlação negativa entre duas variáveis representa que a cada vez que eu aumento uma a outra diminui e virse-versa. O contrário também acontece quando temos uma correlação positiva, a medida que uma variável aumenta a outra também aumenta.

```python
# Using Pearson Correlation
# General correlation
plt.figure(figsize=(25,15))
cor = new_df_num.corr()
sns.heatmap(cor, annot=True, cmap=plt.cm.Reds)
plt.show()
```

![png](/img/output_43_0.png)

```python
#Correlation with output variable
cor_target = abs(cor["Appliances"])
#Selecting highly correlated features
relevant_features = cor_target[cor_target<0.2]
relevant_features
```

    RH_1           0.045596
    RH_2           0.109746
    T3             0.180061
    RH_3           0.088410
    T4             0.195689
    RH_4           0.036932
    T5             0.191782
    RH_5           0.072040
    T7             0.175519
    RH_7           0.128740
    T9             0.154471
    Press_mm_hg    0.089829
    Windspeed      0.055363
    Visibility     0.024974
    Tdewpoint      0.081550
    rv1            0.009986
    rv2            0.009986
    Name: Appliances, dtype: float64

Uma observação interessante é que maioria das variáveis possuem uma correlação negativa com a variável target, ou seja, o 'Appliances' aumenta a medida que uma variável específica diminui.

## Normalização

Como estamos trabalhando com dados numéricos e que possuem uma distribuição com intervalos diferentes, pode ser que o algoritmo de machine learning escolhido seja sensível a este tipo de comportamento. Outro ajuste que se faz necessário é a normalização dos dados.

```python
# lembrando que a variável target não tem necessidade de ser normalizada
new_df_num_Y = new_df_num['Appliances']
new_df_num = new_df_num.drop(['Appliances'], axis=1)
# normalização
min_max_scaler = MinMaxScaler()
columns = list(new_df_num.columns)
print(columns)
for name in columns:
    # convertemos para np array pois é a forma que o min_max_scaler
    # recebe os dados
    new_df_num[name] = min_max_scaler.fit_transform(np.array(new_df_num[name]).reshape(-1, 1))
```

    ['lights', 'T1', 'RH_1', 'T2', 'RH_2', 'T3', 'RH_3', 'T4', 'RH_4', 'T5', 'RH_5', 'T6', 'RH_6', 'T7', 'RH_7', 'T8', 'RH_8', 'T9', 'RH_9', 'T_out', 'Press_mm_hg', 'RH_out', 'Windspeed', 'Visibility', 'Tdewpoint', 'rv1', 'rv2', 'NSM']

Algumas variáveis ainda possuem muitos outliers, mas serão mantidas.

# Feature Selection

Aqui apliquei 2 diferentes métodos de seleção de variáveis.

## Recursive Elimination

Neste método, precisamos definir quantas variáveis queremos para rodar o modelo, e depois, quando obtido o número ótimo aplicamos a seleção de variáveis. O método identifica, recursivamente, qual conjunto de variáveis possuem maior score.

```python
new_df_num.shape
```

    (17597, 28)

```python
#no of features
nof_list=np.arange(1,28)
high_score=0
#Variable to store the optimum features
nof=0
score_list =[]
for n in range(len(nof_list)):
    X_train, X_test, y_train, y_test = train_test_split(new_df_num, new_df_num_Y, test_size = 0.3, random_state = 0)
    model = LinearRegression()
    rfe = RFE(model,nof_list[n])
    X_train_rfe = rfe.fit_transform(X_train,y_train)
    X_test_rfe = rfe.transform(X_test)
    model.fit(X_train_rfe,y_train)
    score = model.score(X_test_rfe,y_test)
    score_list.append(score)
    if(score>high_score):
        high_score = score
        nof = nof_list[n]
print("Optimum number of features: %d" %nof)
print("Score with %d features: %f" % (nof, high_score))
```

    Optimum number of features: 21
    Score with 21 features: 0.353213

```python
cols = list(new_df_num.columns)
model = LinearRegression()
#Initializing RFE model
rfe = RFE(model, 21)
#Transforming data using RFE
X_train, X_test, y_train, y_test = train_test_split(new_df_num, new_df_num_Y, test_size = 0.3, random_state = 0)
X_rfe = rfe.fit_transform(X_train,y_train)
#Fitting the data to model
model.fit(X_rfe,y_train)
temp = pd.Series(rfe.support_,index = cols)
selected_features_rfe = temp[temp==True].index
print(selected_features_rfe)
```

    Index(['lights', 'T1', 'RH_1', 'T2', 'RH_2', 'T3', 'RH_3', 'T4', 'RH_4', 'T5',
           'RH_5', 'T6', 'T7', 'RH_7', 'T8', 'RH_8', 'T9', 'RH_9', 'T_out',
           'Windspeed', 'NSM'],
          dtype='object')

## Embedded Method

```python
reg = LassoCV()
reg.fit(X_train, y_train)
print("Best alpha using built-in LassoCV: %f" % reg.alpha_)
print("Best score using built-in LassoCV: %f" %reg.score(X_train,y_train))
coef = pd.Series(reg.coef_, index = new_df_num.columns)

imp_coef = coef.sort_values()
import matplotlib
matplotlib.rcParams['figure.figsize'] = (8.0, 10.0)
imp_coef.plot(kind = "barh")
plt.title("Feature importance using Lasso Model")
```

    Best alpha using built-in LassoCV: 0.003504
    Best score using built-in LassoCV: 0.356319





    Text(0.5, 1.0, 'Feature importance using Lasso Model')

![png](/img/output_57_2.png)

# Machine Learning

Na etapa de escolha do modelo, selecionei 3 métodos para problemas de regressão, problema que está sendo modelado, e avaliei a métria de R².

```python
# removendo variáveis aletórias antes de aplicar o modelo
new_df_v = new_df_num.drop(['rv1', 'rv2'], axis=1)
```

```python
X_train, X_test, y_train, y_test = train_test_split(new_df_v, new_df_num_Y, train_size=0.7)
```

```python
# Regressão Linear Múltipla
modelo = LinearRegression()
modelo.fit(X_train, y_train)
y_pred = modelo.predict(X_test)

score =  r2_score(y_test, y_pred)
print(score)
```

    0.34058856776814994

```python
# Support Vector Regression
modelo = SVR()
modelo.fit(X_train, y_train)
y_pred = modelo.predict(X_test)

score =  r2_score(y_test, y_pred)
print(score)
```

    0.3594652532873541

```python
# Gradient Boosting Regressor
params = {'n_estimators': 500, 'max_depth': 8, 'min_samples_split': 2,
          'learning_rate': 0.01, 'loss': 'ls'}
clf = ensemble.GradientBoostingRegressor(**params)

clf.fit(X_train, y_train)
y_pred = clf.predict(X_test)
mse = mean_squared_error(y_test, y_pred)
r2score = r2_score(y_test, y_pred)
print(f"R2 SCORE: {r2score}")

```

    R2 SCORE: 0.6598

Vemos então que o modelo que apresentou melhor resultado foi o Gradient Boosting Regressor, com 65,98% na métrica de R². Tal resultado pode ser tido como um resultado preliminar, e etapas de otimização podem ser realizadas daqui em diante para melhorar o score do modelo.

Como formas de otimização que ainda não foram testadas, temos:

- Utilização da técnica de Análise por Componentes Principais para diminuição da dimensionalidade do modelo.
- Criação de novas variáveis.
- Utilização das variáveis categóricas de data como; dia da semana, mês, feriado etc.
- Utilização do gridsearch para buscar um refinamento do modelo de maior score.
