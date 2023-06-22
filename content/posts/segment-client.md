---
layout: post
title: "Segmentação de clientes de Food Delivery"
date: 2020-06-19
author: "Luciano"
featuredImagePreview: "img/segmentation.png"
tags:
  - Project
  - Data Science
  - Machine Learning
  - K-Means
categories: [Projetos]
---

# Segmentação de clientes de Food Delivery

Neste projeto o objetivo é segmentar clientes de um food delivery. A segmentação permite que profissionais de marketing e product managers possam identificar subconjuntos de público-alvo para melhor adaptar suas estratégias.

O dataset utilizado foi cedido pela Data Science Academy, e pode ser encontrado no meu [github](https://github.com/LucianoBatista/food-deliver) juntamente com todo o código apresentado aqui.

Um breve overview do que será abordado no projeto:

- Preparação de dados
- Visualização de dados
- Padronização de variáveis
- Clusterização com K-Means
- Avaliação do melhor número de cluster com o índice de Calinski-Harabasz
- Bootstrap para avaliar consistência do cluster
- Visualização dos clusters

Sem mais delongas, vamos ao código!

![food_delivery](/img/food_delivery.png)

## Libs

```r
library(tidyverse)
library(scales)
library(lubridate)
library(ggridges)
library(gghalves)
library(tidytext)
library(tidymodels)
library(wrapr)
library(fpc)
library(wesanderson)
library(skimr)
```

## Análise Exploratória dos Dados

Vamos importar os dados e visualizar quais são as variáveis que iremos trabalhar ao longo desse projeto.

```r
orders_client_raw <- read_csv("data/dataset.csv")

glimpse(orders_client_raw)
```

```r
Rows: 260,645
Columns: 8
$ id_transacao    <chr> "0x7901ee", "0x7901ee", "0x7901ee", "0x12b47f", "0x12b47f", "0x6d6979", "0x6d6979", "0x78dd1e", "0x78dd1e", "0x78dd1e", "0x4df8ab", "…
$ horario_pedido  <dttm> 2019-01-16 18:33:00, 2019-01-16 18:33:00, 2019-01-16 18:33:00, 2019-09-04 12:36:00, 2019-09-04 12:36:00, 2019-03-18 00:27:00, 2019-0…
$ localidade      <dbl> 7, 7, 7, 3, 3, 6, 6, 2, 2, 2, 8, 8, 6, 6, 7, 7, 7, 7, 6, 6, 6, 2, 2, 2, 7, 7, 7, 7, 2, 2, 6, 6, 6, 6, 2, 2, 2, 9, 9, 9, 9, 9, 9, 5, 5…
$ nome_item       <chr> "bebida", "pizza", "sobremesa", "salada", "sobremesa", "pizza", "sobremesa", "bebida", "pizza", "sobremesa", "salada", "sobremesa", "…
$ quantidade_item <dbl> 2, 2, 2, 1, 1, 2, 2, 2, 2, 2, 3, 3, 1, 1, 3, 3, 1, 4, 1, 4, 4, 1, 2, 2, 2, 2, 1, 3, 1, 1, 1, 1, 3, 4, 1, 2, 2, 4, 4, 4, 4, 4, 4, 1, 1…
$ latitude        <dbl> 41.79413, 41.79413, 41.79413, 41.88449, 41.88449, 41.78458, 41.78458, 42.04931, 42.04931, 42.04931, 41.89420, 41.89420, 41.78458, 41.…
$ longitude       <dbl> -88.01014, -88.01014, -88.01014, -87.62706, -87.62706, -87.60756, -87.60756, -87.67761, -87.67761, -87.67761, -87.62096, -87.62096, -…
$ nome_item_fac   <fct> bebida, pizza, sobremesa, salada, sobremesa, pizza, sobremesa, bebida, pizza, sobremesa, salada, sobremesa, pizza, sobremesa, bebida,…
```

A nível de melhor compreender os dados podemos estabelecer um dicionário com o que cada preditor representa:

- **id_transacao**: ID da transação. Um mesmo ID pode ter vários itens de um pedido.
- **horario_pedido**: Horário exato do pedido.
- **localidade**: Localidade que processou o pedido (unidade do restaurante).
- **nome_item**: Nome do item (pizza, salada, bebida, sobremesa).
- **quantidade_item**: Quantidade de itens no pedido.
- **latitude**: Latitude da localidade onde o pedido foi gerado.
- **longitude**: Longitude da localidade onde o pedido foi gerado.

Observamos que a variável nome_item possui um tipo de character, porém é mais indicado trabalhar com a mesma como fator.

```r
orders_client_raw$nome_item_fac <- as.factor(orders_client_raw$nome_item)
```

No R temos um pacote muito interessante que permite ter um panorama geral sobre os dados, com várias estatísticas mais básicas, chamado `skimr`.

```r
skim(orders_client_raw)
```

```r
── Data Summary ────────────────────────
                           Values
Name                       orders_client_raw
Number of rows             260645
Number of columns          8
_______________________
Column type frequency:
  character                2
  factor                   1
  numeric                  4
  POSIXct                  1
________________________
Group variables            None

── Variable type: character ───────────────────────────────────────────────────────────────────────────────────────
  skim_variable n_missing complete_rate   min   max empty n_unique whitespace
1 id_transacao          0             1     7     8     0   100000          0
2 nome_item             0             1     5     9     0        4          0

── Variable type: factor ──────────────────────────────────────────────────────────────────────────────────────────
  skim_variable n_missing complete_rate ordered n_unique top_counts
1 nome_item_fac         0             1 FALSE          4 sob: 100000, piz: 76122, beb: 46156, sal: 38367

── Variable type: numeric ─────────────────────────────────────────────────────────────────────────────────────────
  skim_variable   n_missing complete_rate   mean    sd    p0   p25   p50   p75  p100 hist
1 localidade              0             1   5.13 2.55    1     3     5     7     9   ▆▆▂▇▆
2 quantidade_item         0             1   2.45 1.33    1     1     2     4     5   ▇▆▅▅▂
3 latitude                0             1  41.8  0.144  41.5  41.8  41.9  41.9  42.0 ▂▁▅▇▂
4 longitude               0             1 -87.7  0.136 -88.0 -87.8 -87.7 -87.6 -87.6 ▂▂▂▁▇

── Variable type: POSIXct ─────────────────────────────────────────────────────────────────────────────────────────
  skim_variable  n_missing complete_rate min                 max                 median              n_unique
1 horario_pedido         0             1 2019-01-01 00:00:00 2019-12-30 23:59:00 2019-07-01 11:49:00    76799
```

De mais importante no resumo acima, podemos ver que os dados não possuem valores missing (=D), temos 100.000 valores únicos no `id_transacao` e temos 4 produtos de diferentes categorias.

Localidade possívelmente precisará ser convertida à fator, pois cada valor representa um local diferente de onde o pedido processado. Mas caso venhámos a fazer alguma análise com esta variável podemos converte-la mais a frente.

```r
orders_client_v1 <- orders_client_raw %>%
  mutate(month = month(horario_pedido, label = T, abbr = F),
         year = year(horario_pedido))

# visualizando o período referente aos dados
orders_client_v1 %>%
  select(year) %>%
  unique()

```

```r
# A tibble: 1 x 1
   year
  <dbl>
1  2019
```

Vemos que os dados são referentes ao ano de 2019. Uma visualização interessante seria dos pedidos ao longo dos meses, podendo entender o que vende mais em determinado período.

```r
# pedidos em cada mês
order_by_month_name <- orders_client_v1 %>%
  select(month, nome_item_fac, quantidade_item) %>%
  group_by(month, nome_item_fac) %>%
  summarise(tot = sum(quantidade_item))

# visualização
order_by_month_name %>%
  ggplot(aes(x = tot,
             y = reorder(nome_item_fac, tot),
             color = nome_item_fac)) +
  geom_pointrange(aes(xmin = 0, xmax = tot), fatten = 5) +
  scale_color_manual(values = wes_palette(n = 4, name = "BottleRocket2")) +
  facet_wrap(~month) +
  scale_x_continuous(labels = label_number(scale = 1/1000)) +
  guides(fill = F) +
  labs(x = "Quantidades de pedidos em milhar",
       y = NULL,
       color = "Pedidos") +
  theme_minimal()
```

![graph1](/img/graph1.png)

Vimos que ao longo dos meses essa loja recebeu praticamente a mesma quantidade de pedidos todos os meses. Sempre com a menor preferência por salada e a maior preferência por sobremesa.

Como nós também temos os dados durante as horas do dia, vamos realizar mais uma preparação nos dados.

```r
# incluindo a hora do dia para visualizar depois
orders_client_v2 <- orders_client_raw %>%
  mutate(month = month(horario_pedido),
         year = year(horario_pedido),
         hour = hour(horario_pedido))

order_by_hour_name <- orders_client_v2 %>%
  select(month, hour, nome_item_fac, quantidade_item) %>%
  group_by(month, hour, nome_item_fac) %>%
  summarise(tot = sum(quantidade_item))

```

Agora que o dataset `order_by_hour_name` possui as informações necesssárias para fazer a visualização, vamos visualizar então como foram os pedidos para cada hora do dia ao longo dos meses e do ano de 2019.

```r
order_by_hour_name %>%
  ggplot(aes(x = tot,
             y = reorder(nome_item_fac, tot),
             fill = nome_item_fac)) +
  geom_col() +
  scale_fill_viridis_d() +
  facet_grid(month~hour) + # hora na vertical e mês na horizontal
  scale_x_continuous(labels = label_number(scale = 1/1000)) +
  guides(fill = F) +
  labs(x = "Quantidades de pedidos em milhar",
       y = NULL,
       color = "Pedidos")
```

_Para uma melhor visualização, rode o código acima na sua máquina e visualize o gráfico em um maior tamanho._

![graph2](/img/graph2.png)

Mas com esse preview já vemos que os dados seguem um comportamento bastate padrão em todas as horas dos dias de todos os messes do ano.

Conclusões que podemos tirar dos dados até agora:

- entre o período de almoço existe um consumo muito grande de sobremesas e salada.
- das 17 até as 19 horas se consome muito bebida, pizza e sobremesa.
- na 0 hora existe um pico de pedidos de sobremesa e pizza.

Bom, até agora nós visualizamos os pedidos durante o mês e durante as horas nos meses do ano. Mas seria interessante também observar o total da distribuição de pedidos no ano como um todo.

Para isso eu gosto muito de utilizar o pacote `ggridgs`, com ele podemos observar as várias distribuições de uma forma mais compacta e ainda traçar linhas referentes aos diferentes quartis.

```r
order_by_hour_name %>%
ggplot(aes(x = tot,
           y = reorder(nome_item_fac, tot),
           fill = nome_item_fac)) +
  geom_density_ridges(scale = 3, quantile_lines = TRUE, quantiles = 2) +
  scale_fill_viridis_d() +
  facet_wrap(~hour, scales = "free_x") +
  scale_x_continuous(labels = label_number(scale = 1/1000)) +
  guides(fill = F) +
  labs(x = "Quantidades de pedidos em milhar",
       y = NULL)
```

![graph3](/img/graph3.png)

Legal, nesse gráfico nós vemos que em algumas distribuições nós temos caudas muito longas, e outras parecem ser bimodal. Pode ser interessante visualizar se esse comportamento está sendo provocado por outliers ou se é um comportamento "real" desses dados.

Para isso chamaremos o bom e velho boxplot/violinoplot, porém adicionamos à ele uma camada de visualização dos pontos que o compões. Vamos lá!

```r
order_by_hour_name %>%
  ggplot(aes(x = nome_item_fac,
             y = tot)) +
  geom_half_boxplot(aes(fill = nome_item_fac), side = 'r', alpha = .3) +
  geom_half_point(aes(color = nome_item_fac), side = 'l', size = .6) +
  coord_flip() +
  labs(y = "Quantidade total de pedidos",
       x = NULL) +
  guides(color = F, fill = F)

ggsave("graph5.png", width = 7, height = 5)
```

![graph5](/img/graph5.png)

```r
order_by_hour_name %>%
  ggplot(aes(x = nome_item_fac,
             y = tot)) +
  geom_half_violin(aes(fill = nome_item_fac), side = 'r', alpha = .3, scale = 3) +
  geom_half_point(aes(color = nome_item_fac), side = 'l', size = .6) +
  coord_flip() +
  labs(y = "Quantidade total de pedidos",
       x = NULL) +
  guides(color = F, fill = F)
```

![graph4](/img/graph4.png)

Vamos colocar um highlight na região de outliers

```r
order_by_hour_name %>%
  ggplot(aes(x = nome_item_fac,
             y = tot)) +
  geom_half_boxplot(aes(fill = nome_item_fac), side = 'r', alpha = .3) +
  geom_half_point(aes(color = nome_item_fac), side = 'l', size = .6) +
  coord_flip() +
  annotate(geom = "rect", xmin = 2.7, xmax = 3.1, ymin = 1100 , ymax = 2300,
           fill = "grey40", alpha = .2) +
  annotate(geom = "rect", xmin = 0.7, xmax = 1.1, ymin = 1900 , ymax = 3700,
           fill = "grey40", alpha = .2) +
  labs(y = "Quantidade total de pedidos",
       x = NULL) +
  guides(color = F, fill = F) +
  theme_minimal()
```

![graph6](/img/graph6.png)

Ótimo, na sequência acima nós podemos visualizar as regiões com outliers para cada classe de pedido. Como nós estamos tentando entender melhor os dados, não faremos nenhum tratamento neles e levaremos para o modelo final (k-means).

Uma observação interessante é que, realizando essa segmentação, poderemos entender melhor qual grupo de clientes que consomem mais saladas e que também bebem mais que outros consumidores (outliers do nossos dados).

Vamos visualizar como está sendo os pedidos referente a cada localidade.

```r
# como dito antes, aqui nós convertemos a localidade para fator
order_by_localidade <- orders_client_v2 %>%
  select(localidade, month, hour, nome_item_fac, quantidade_item) %>%
  mutate(localidade = as.factor(localidade)) %>%
  group_by(localidade, month, hour, nome_item_fac) %>%
  summarise(tot = sum(quantidade_item))

order_by_localidade %>%
  ggplot(aes(x = tot,
             y = reorder_within(nome_item_fac, tot, localidade),
             fill = nome_item_fac)) +
  geom_col() +
  scale_fill_viridis_d() +
  facet_wrap(~localidade, scales = "free_y") +
  scale_y_reordered() +
  guides(fill = F) +
  labs(x = "Quantidade total de pedidos",
       y = NULL)
```

![graph7](/img/graph7.png)

Essas observações com certeza são importantíssimas para uma empresa de delivery, pois sabendo qual central recebe mais pedidos sobre um determinado produto, fica mais fácil fazer um **controle de estoque** ou até mesmo optar por **não vender** determinado produto numa região devido uma saída muito pequena, caso a economia com matéria-prima seja realmente interessante.

Referente a `localidade`, podemos concluir:

- A maioria dos pedidos de salada saem da localidade 1, 3 e 5.
- Maior consumo de pizza, sobremesa e bebida do grupo 4, 7 e 9.

Por fim, como temos dados períodos bem completos pode ser interessante dá uma olhada num heatmap da quantidade de pedidos ao longo do ano.

```r
order_by_localidade %>%
  ggplot(aes(x = hour,
             y = as.factor(month),
             fill = tot)) +
  geom_tile() +
  scale_fill_viridis_c() +
  scale_x_continuous(breaks = 0:24) +
  coord_equal() +
  labs(x = "Hora do dia",
       y = "Meses do ano",
       fill = "Totais de\nPedidos") +
  theme_minimal()
```

![graph8](/img/graph8.png)

Aqui nós vemos claramente o período do dia em que ocorre maior número de pedidos. E também o vácuo (das 2 às 11 horas) que representa a não ocorrência de pedidos.

# Cluster analysis

Até agora nós tivemos um panorama geral dos dados, retirando bons insights. Dentre as variáveis analisadas, uma que não trabalhamos até agora é a `id_transacao`, até por que a mesma possui 100k valores únicos. Porém, ela é importantíssima para uma empresa de delivery, por que trás a informação do cliente.

Uma forma de analisar uma informação com tamanha dimensionalidade é tentar agrupa-la em clusters. Para isso fazemos uso de métodos de machine learning não-supervisionados que visam _agrupar grupos_ similares, essa similaridade é medida considerando diferentes tipos de distâncias: euclidiana, hamming, manhattan, similaridade cosseno...

Cada distância dessa é indicada para um tipo de dado, sendo a mais usada a euclidiana (dados numéricos). Ao mudar a medida de distância damos origem a diferentes algorítmos de clusterização, e como você pode imaginar, essa pluralidade é tão grande que facilmente encontramos livros inteiros apenas sobre clusterização.

Aqui nós optamos por utilizar o mais popular entre eles, o k-means, e primeiramente vamos pivotar os dados para um formato que será melhor analisado pelo algorítmo.

## Pivotando

```r
order_client_clustering <- orders_client_raw %>%
  select(id_transacao, nome_item_fac, quantidade_item) %>%
  pivot_wider(id_transacao, names_from = nome_item_fac, values_from = quantidade_item, values_fill = 0)

order_client_clustering
```

```r
# A tibble: 100,000 x 5
   id_transacao bebida pizza sobremesa salada
   <chr>         <dbl> <dbl>     <dbl>  <dbl>
 1 0x7901ee          2     2         2      0
 2 0x12b47f          0     0         1      1
 3 0x6d6979          0     2         2      0
 4 0x78dd1e          2     2         2      0
 5 0x4df8ab          0     0         3      3
 6 0x3be6d3          0     1         1      0
 7 0x755a0b          3     3         4      1
 8 0x653685          1     4         4      0
 9 0x44fe1b          1     2         2      0
10 0x8ade21          2     2         3      1
# … with 99,990 more rows
```

Como falamos antes, o k-means e outros algorítmos dessa classe de modelos, trabalham com distâncias, por isso é extremamente interessante que tenhamos os dados numa mesma escala e unidade. Aqui aplicaremos uma padronização (média = 0 e desvio = 1), assumindo assim que nossa distribuição é uma distribuição normal, o que de fato vimos que na maioria dos casos essa premissa se fez verdade.

## Padronização

```r
# variáveis para padronizar
vars_to_use <- colnames(order_client_clustering)[-1]
pmatrix <- scale(order_client_clustering[, vars_to_use])
pcenter <- attr(pmatrix, "scaled:center") # parâmetro que foi utilizado para centralizar
pscale <- attr(pmatrix, "scaled:scale") # parâmetro que foi utilizado para escalonar

rm_scales <- function(scaled_matrix) {
  attr(scaled_matrix, "scaled:center") <- NULL
  attr(scaled_matrix, "scaled:scale") <- NULL
  scaled_matrix
}

# matriz que utilizaremos para modelagem
pmatrix <- rm_scales(pmatrix)

```

## K-means

Antes de iniciar a implementação do k-means, vale pontuar algumas questões importantes sobre o algorítimo e que levaram a adotar o workflow abaixo.

- O k-means agrupa os dados para um número pré-determinado de k-clusters. Então é interessante ecolher um número de clusters anteriomente.
- O k-means inicia o processo a partir de um ponto aleatório do seu espaço de dados, por isso uma boa prática é rodar o modelo várias vezes para observar o comportamento dos dados nos clusters.
- Os grupos (clusters) formados pelo k-means não necessariamente serão "clusters reais" (explico abaixo). E para isso teremos que avaliar a estabilidade desses clusters.

O último ponto trás o conceito de "clusters reais", que de forma simplificada nada mais é do que um cluster de pontos que não são bem explicados por nenhum outro cluster. Para verificar isso vamos utilizar uma reamostragem bootstrap e rodar o kmeans() várias vezes e em diferentes porções dos dados e medir isso.

E por fim, iremos utilizar o index **Calinski-Harabasz** para identificar o melhor cluster. Existem _váaarias_ formas de fazer isso, mas eu preferi escolher essa e aqui segue um [link](https://ethen8181.github.io/machine-learning/clustering_old/clustering/clustering.html) caso você queira entender melhor sobres as definições matemáticos por trás desse método, **muito bem explicado** por sinal.

```r
# 1 - kmeansruns para encontrar o melhor número de cluster e visualizar o calinski-harabasz index
# 4 - clusterboot() para analisar a estabilidade do cluster
# 2 - col_to_print
# 3 - print_cluster()
# 5 - visualizar cada grupo

# kmeansruns nos nossos dados padronizados, com um range em k de 1 a 10, e critério do calinski
clustering_ch <- kmeansruns(pmatrix, krange = 1:10, criterion = "ch")

# best k retornado pelo modelo
clustering_ch$bestk

# pequeno tratamento nos dados para poder visualizar o CH-index em função do cluster
coef <- clustering_ch$crit
k <- 1:10
data <- tibble(coef, k)

ggplot(data, aes(k, coef)) +
  geom_point() +
  geom_line(linetype="dashed") +
  scale_x_continuous(breaks = 0:10) +
  theme_minimal() +
  labs(y = "Calinski-Harabasz Index")
```

![graph9](/img/graph9.png)

Quanto maior o índice, melhor. Então eu optei por escolher dois dos maiores pontos (3 e 8 clusters) e visualizar como se dá o comportamento deles quando realizado o bootstrap.

```r
# essa célula pode demorar um pouco para rodar dependendo da sua máquina
# running o clusterboot pra melhor escolher o k
# k = 3
cboot_3 <- clusterboot(pmatrix, clustermethod = kmeansCBI, runs = 100, iter.max = 100, krange = 3, seed = 2020)
# k = 8
cboot_8 <- clusterboot(pmatrix, clustermethod = kmeansCBI, runs = 100, iter.max = 100, krange = 8, seed = 2020)

```

No objeto `cboot_x` retornado pela função `clusterboot()` podemos chamar o camapo `$bootmean` que indica justamente qual foi a estabilidade do cluster ao longo das 100 iterações do bootstrap.

```r
> cboot_3$bootmean
[1] 1 1 1
```

```r
> cboot_8$bootmean
[1] 0.6846583 0.9253406 0.8652381 0.7604600 0.6980723 0.9149958 0.8921955 0.7588892
```

O que vemos aqui é que estranhamente o modelo com 3 clusters tem uma estabilidade de 1, o que seria perfeito em se tratando que o range de valores possíveis dessa técnica é de 0 a 1, provavelmente tem alguma coisa errada com esse número de clusters já que nada na vida é tão perfeito assim. Por isso, vamos optar de seguir com 8 clusters, utilizando então o objeto `cboot_8`.

O que os números do `$bootmean` nos dizem?

- clusters com valores **menores que 0.6** são considerados instáveis.
- valores **entre 0.6 e 0.75** indicam que o cluster encontrou algum nível de padrão nos dados.
- valores acima disso possui maior estabilidade, sendo os mais estáveis os valores **acima de 0.85**.

```r
# groups recebe os datasets com os cluster
groups <- cboot_8$result$partition

# colunas que serão utilizadas dataset final com os clusters
cols_to_print <- wrapr::qc(id_transacao, bebida, pizza, sobremesa, salada)

print_clusters <- function(data, groups, columns) {
  groupedID <- split(data, groups)
  lapply(groupedID, function(df) df[, columns])
}

clusters <- print_clusters(order_client_clustering, groups, cols_to_print)

```

Nesse ponto, o que temos é uma lista com um dataframe para cada cluster, ou seja, todos os pontos do nosso dataset original nos clusters que o modelo identificou.

Vamos agora fazer algumas visualizações em cada cluster

## Clusters Visualization

Nas próximas visualizações teremos basicamente o mesmo código de plotagem, onde cada gráfico trás a informação de um cliente (id_transacao), e qual o número de pedidos (1, 2, 3, 4 ou 5) que o mesmo fez em relação a cada produto (bebida, pizza, salada, sobremesa).

Uma linha horizontal foi traçada para representar um cliente dentre os milhares presentes nos plots.

### Cluster 1

O cluster 1 trás os clientes que possuem um maior consumo de sobremesa e salada, e que consomem pouco, ou não consomem nem bebida nem pizza. Aqueles que querem ser saudáveis mas não dispensam um docinho.

```r
cluster_1 <- clusters[[1]] %>%
  pivot_longer(cols = c("bebida", "pizza", "sobremesa", "salada"), names_to = "pedidos", values_to = "values")


cluster_1 %>%
  ggplot(aes(x = values,
             y = id_transacao,
             color = pedidos)) +
  geom_point(position = position_jitter(height = 0, width = .3)) +
  scale_x_continuous(breaks = 0:4) +
  annotate(geom = "rect", xmin = -0.5, xmax = 4.5, ymax = 16500, ymin = 16400, alpha = .3) +
  labs(x = "Número de pedidos realizado pelo cliente",
       title = "Pedidos feitos por cada cliente",
       subtitle = "A linha horizontal indica a informação de um cliente e seus pedidos") +
  theme(axis.text.y.left = element_blank(),
        axis.title.y = element_blank(),
        axis.ticks.y = element_blank())
```

![graph10](/img/graph10.png)

### Cluster 2

O cluster 2 é um público que consome bem pouco, e quando consome, consome mais pizza, sobremesa e alguns ainda consomem bebida. Esses são realmente aversos a saladas.

```r
cluster_2 <- clusters[[2]] %>%
  pivot_longer(cols = c("bebida", "pizza", "sobremesa", "salada"), names_to = "pedidos", values_to = "values")


cluster_2 %>%
  ggplot(aes(x = values,
             y = id_transacao,
             color = pedidos)) +
  geom_point(position = position_jitter(height = 0, width = .3)) +
  scale_x_continuous(breaks = 0:1) +
  annotate(geom = "rect", xmin = -0.5, xmax = 1.5, ymax = 16500, ymin = 16400, alpha = .3) +
  labs(x = "Número de pedidos realizado pelo cliente",
       title = "Pedidos feitos por cada cliente",
       subtitle = "A linha horizontal indica a informação de um cliente e seus pedidos") +
  theme(axis.text.y.left = element_blank(),
        axis.title.y = element_blank(),
        axis.ticks.y = element_blank())
```

![graph11](/img/graph11.png)

### Cluster 3

No cluster 3 nós vemos os clientes que pedem em maior quantidade pizza e sobremesa e optam por não pedir ou pedir pouco salada e bebida.

```r
cluster_3 <- clusters[[3]] %>%
  pivot_longer(cols = c("bebida", "pizza", "sobremesa", "salada"), names_to = "pedidos", values_to = "values")


cluster_3 %>%
  ggplot(aes(x = values,
             y = id_transacao,
             color = pedidos)) +
  geom_point(position = position_jitter(height = 0, width = .3)) +
  scale_x_continuous(breaks = 0:5) +
  annotate(geom = "rect", xmin = -0.5, xmax = 5.5, ymax = 3500, ymin = 3470, alpha = .3) +
  labs(x = "Número de pedidos realizado pelo cliente",
       title = "Pedidos feitos por cada cliente",
       subtitle = "A linha horizontal indica a informação de um cliente e seus pedidos") +
  theme(axis.text.y.left = element_blank(),
        axis.title.y = element_blank(),
        axis.ticks.y = element_blank())
```

![graph12](/img/graph12.png)

### Cluster 4

O cluster 4 trás as pessoas que gostam de sobremesa mas que pedem um pouco de pizza e salada. Além disso não costuma pedir bebidas.

```r
cluster_4 <- clusters[[4]] %>%
  pivot_longer(cols = c("bebida", "pizza", "sobremesa", "salada"), names_to = "pedidos", values_to = "values")


cluster_4 %>%
  ggplot(aes(x = values,
             y = id_transacao,
             color = pedidos)) +
  geom_point(position = position_jitter(height = 0, width = .3)) +
  scale_x_continuous(breaks = 0:2) +
  annotate(geom = "rect", xmin = -0.5, xmax = 2.5, ymax = 11500, ymin = 11430, alpha = .3) +
  labs(x = "Número de pedidos realizado pelo cliente",
       title = "Pedidos feitos por cada cliente",
       subtitle = "A linha horizontal indica a informação de um cliente e seus pedidos") +
  theme(axis.text.y.left = element_blank(),
        axis.title.y = element_blank(),
        axis.ticks.y = element_blank())
```

![graph13](/img/graph13.png)

### Cluster 5

Aqui vemos mais um cluster onde os clientes gostam de pedir muita salda com sobremesa, mas não dispensam uma pizza em algumas situações.

```r
cluster_5 <- clusters[[5]] %>%
  pivot_longer(cols = c("bebida", "pizza", "sobremesa", "salada"), names_to = "pedidos", values_to = "values")


cluster_5 %>%
  ggplot(aes(x = values,
             y = id_transacao,
             color = pedidos)) +
  geom_point(position = position_jitter(height = 0, width = .3)) +
  scale_x_continuous(breaks = 0:5) +
  annotate(geom = "rect", xmin = -0.5, xmax = 5.5, ymax = 2800, ymin = 2770, alpha = .3) +
  labs(x = "Número de pedidos realizado pelo cliente",
       title = "Pedidos feitos por cada cliente",
       subtitle = "A linha horizontal indica a informação de um cliente e seus pedidos") +
  theme(axis.text.y.left = element_blank(),
        axis.title.y = element_blank(),
        axis.ticks.y = element_blank())
```

![graph14](/img/graph14.png)

### Cluster 6

Os clientes do grupo 6 são também bem aversos a salada, optando por uma combinação perfeita, sobremesa e pizza.

```r
cluster_6 <- clusters[[6]] %>%
  pivot_longer(cols = c("bebida", "pizza", "sobremesa", "salada"), names_to = "pedidos", values_to = "values")


cluster_6 %>%
  ggplot(aes(x = values,
             y = id_transacao,
             color = pedidos)) +
  geom_point(position = position_jitter(height = 0, width = .3)) +
  scale_x_continuous(breaks = 0:3) +
  annotate(geom = "rect", xmin = -0.5, xmax = 3.5, ymax = 15500, ymin = 15400, alpha = .3) +
  labs(x = "Número de pedidos realizado pelo cliente",
       title = "Pedidos feitos por cada cliente",
       subtitle = "A linha horizontal indica a informação de um cliente e seus pedidos") +
  theme(axis.text.y.left = element_blank(),
        axis.title.y = element_blank(),
        axis.ticks.y = element_blank())
```

![graph15](/img/graph15.png)

### Cluster 7

No cluster 7 temos aqueles clientes que juntamente a pizza com sobremesa, adicionam uma bebida.

```r
cluster_7 <- clusters[[7]] %>%
  pivot_longer(cols = c("bebida", "pizza", "sobremesa", "salada"), names_to = "pedidos", values_to = "values")


cluster_7 %>%
  ggplot(aes(x = values,
             y = id_transacao,
             color = pedidos)) +
  geom_point(position = position_jitter(height = 0, width = .3)) +
  scale_x_continuous(breaks = 0:5) +
  annotate(geom = "rect", xmin = -0.5, xmax = 5.5, ymax = 17500, ymin = 17400, alpha = .3) +
  labs(x = "Número de pedidos realizado pelo cliente",
       title = "Pedidos feitos por cada cliente",
       subtitle = "A linha horizontal indica a informação de um cliente e seus pedidos") +
  theme(axis.text.y.left = element_blank(),
        axis.title.y = element_blank(),
        axis.ticks.y = element_blank())
```

![graph16](/img/graph16.png)

### Cluster 8

E por fim, o cluster os loucos por sobremesa.

```r
cluster_8 <- clusters[[8]] %>%
  pivot_longer(cols = c("bebida", "pizza", "sobremesa", "salada"), names_to = "pedidos", values_to = "values")


cluster_8 %>%
  ggplot(aes(x = values,
             y = id_transacao,
             color = pedidos)) +
  geom_point(position = position_jitter(height = 0, width = .3)) +
  scale_x_continuous(breaks = 0:5) +
  annotate(geom = "rect", xmin = -0.5, xmax = 5.5, ymax = 9500, ymin = 9420, alpha = .3) +
  labs(x = "Número de pedidos realizado pelo cliente",
       title = "Pedidos feitos por cada cliente",
       subtitle = "A linha horizontal indica a informação de um cliente e seus pedidos") +
  theme(axis.text.y.left = element_blank(),
        axis.title.y = element_blank(),
        axis.ticks.y = element_blank())
```

![graph17](/img/graph17.png)
