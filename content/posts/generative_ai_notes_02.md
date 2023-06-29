---
layout: post
title: "Deep Learning"
subtitle: "Chapter 02 - Generative Deep Learning Book"
date: 2023-06-28
author: "Luciano"
authorLink: "https://www.linkedin.com/in/lucianobatistads/"
featuredImage: "img/deep-learnig-network.png"
math:
  enable: true
tags:
  - AI
  - Generative
  - IA Generativa
categories: [Tutorials]
draft: false
---

Chegamos no capítulo 2 do livro, e o mesmo fala sobre Deep Learning e implementações de CNNs utilizando o Keras como framework base para codar as implementações. Porém, as coisas são explicadas de uma forma bem superficial, e dificilmente quem nunca teve contato com esse conteúdo vai conseguir acompanhar de forma satisfatória.

Pensando nisso, gostaria de fazer algo diferente, pensei em desmembrar o assunto e fazer uma sequência de **3 posts** para cobrir o capítulo 2.

Esse primeiro vai retratar o que é Deep Learning, trazendo para vocês um pouco do fundamento por de trás, e ao mesmo tempo trazer uma percepção de quais são as "pecinhas" que compõe esse quebra-cabeça. E sim, _eu penso o Deep Learning como um grande Lego que a gente tem montar para que a coisa funcione como esperado._

Já o artigo dois, supondo que você agora compreende alguns conceitos importantes, vamos ser responsáveis por implementar um MultiLayer Perceptron utilizando o `pytorch-lightning`. Aqui também iremos trazer mais alguns conceitos úteis para nossa caminhada no mundo da IA Generativa.

O artigo três, finaliza a sequência cobrindo CNNs. Acho que assim fica mais interessante de mais pessoas acompanharem os artigos seguintes. Dito isso, vamos lá!!

{{< admonition type=note title="Nota" open=true >}}

Vale lembrar que vou utilizar uma outra referência para escrever esses três artigos e você pode encontrar o nome do livro final do post :wink:.

{{< /admonition>}}

## O "tal" do Deep Learning

O único pressuposto que vou fazer aqui é que você já possui um entendimento sobre Machine Learning e sobre algumas nomenclaturas e conceitos da área, dessa forma conseguimos com maior facilidade trazer o Deep Learning para o seu contexto.

Vamos começar pelo clássico diagrama de Venn, batido, mas ainda assim interessante para situar você de onde o Deep Learning está, dentro do universo de IA.

![img](/img/posts/gen-ai-02/ven-diagram-dl.png)

Como você pode ver, o Deep Learning é um "subset" do Machine Learning e portanto ambos compartilham do mesmo objetivo, o aprendizado. Porém, o diferencial é que o Deep Learning vai trabalhar exclusivamente com redes neurais artificiais para "aprender" a executar as mais diferentes tasks possíveis:

- image segmentation
- object detection
- text generation

Essas redes, são compostas por diferentes arquiteturas que nascem de muita pesquisa e que funcionam muito bem para um contexto específico. Esse contexto é chamado aqui de "_Conhecimento a Priori_" e basicamente isso quer dizer que nós estamos sempre partindo de um tipo de dado que respeita um algum tipo de estrutura.

Se eu to partindo de que meu dado possui algum tipo de estrutura, um questionamento pode surgir na sua cabeça:

> _Como assim estruturado? Eu vejo muitos modelos trabalhando com imagens, textos, vídeos... E são dados desestruturados_

Justamente, esses dados, por um determinado ponto de vista :eyes:, são na verdade estruturados.

Vamos pegar uma imagem como exemplo:

![](https://i.imgur.com/emXiYTb.png)
_(https://www.researchgate.net/figure/mage-of-Abraham-Lincoln-as-a-matrix-of-pixel-values_fig1_330902210)_

Ela é nada mais que uma matriz multidimensional (imagens coloridas) de números dispostos de uma dada forma que cada valor presente na matriz vai ser representado como um pixel. Esse mesmo comportamento se repete para um vídeo ou para um texto.

Então, se eu sei que meu dado tem essa característica, eu posso pensar numa arquitetura que consiga responder algumas perguntas sobre esse tipo de dado. Como por exemplo, se uma imagem pode ser classificada como um gato? Ou, qual é a probabilidade da próxima palavra ser _"quente"_, dado que eu falei _'cachorro-'_ ...?

Isso não acontece por exemplo com um dado tabular de previsão de Churn de clientes. Vamos pensar no seguinte exemplo, na empresa de João nós temos as variáveis $x$ e $y$ que explicam muito bem os motivos de Churn dos clientes, e é possível ter uma previsão que permite a empresa agir de forma proativa e minimizar esse problema com antecipação :ok_hand:.

Agora, considerando a empresa de Pedro, as variáveis que explicam o mesmo fenômeno são $z$ e $p$, logo, não existe um **Conhecimento a Priori** que poderia ser obedecido, veja que o mesmo problema podem ser explicados por variáveis diferentes com pesos diferentes. Esse é um dos motivos que normalmente você não vê um modelo de Deep Learning performando tão bem quanto um Xgboost em competições tabulares no Kaggle.

## Deep Learning como um carro

Uma comparação que para mim fez bastante sentido foi a da imagem abaixo. Nela, o autor pensa o treinamento de um modelo de Deep Learning como um funcionamento de um carro. Podemos dizer que o combustível desse carro é o **Dado**, logo, nós iremos precisar de uma forma de carregar esse dado para dentro do nosso motor, no caso do `Pytorch`, esse seria o papel do `DataLoader`. O motor, responsável pelo aprendizado, precisa de uma forma de **ajustar os parâmetros e pesos** (em conjunto com esse "Box of blocks") para levar o nosso modelo para a direção certa.

{{<image width=800 src="https://i.imgur.com/QHKlwyM.png">}}

O seu papel aqui é um pouco diferente de quando você utiliza por exemplo o `scikit-learn` para treinar uma Random Forest. Nessa abordagem, você não precisa codar o modelo e muito menos o processo de treinamento, pois tudo isso foi encapsulado em um método `.fit()` e em uma classe `RandomForest`.

Diferentemente, no Deep Learning, você se torna o responsável por juntar cada pecinha que até então, no Machine Learning "tradicional" foi abstraído de você, de uma forma que o aprendizado ocorra da forma esperada.

A depender de onde você trabalhe, codar a arquitetura de um modelo de Deep Learning, ou até mesmo criar uma arquitetura nova, não serão atividades do seu dia-a-dia. Mas, ainda assim, restam várias pecinhas que você precisa conectar, afim de realizar um treinamento de Deep Learning corretamente. E ainda assim, caso você não seja o responsável por implementar arquiteturas do zero, existem alguns ajustes que você pode vir a fazer para realizar algum retreino ou fine-tunning de um desses modelos, que conhecer a arquitetura pode vir a ajudar.

Um ponto chave para entender o aprendizado em Deep Learning, é pensar que nós estamos sempre tentando minimizar uma função, ou seja, nós queremos sempre caminhar para encontrar os pesos e parâmetros que nos darão os menores valores para essa função. Essa função é comumente chamada de **Loss Function**.

## Um pouco de matemática

Nas disciplinas de cálculo, vimos que quando é necessário encontrar o mínimo de uma função, nós precisamos calcular a derivada dessa função e iguala-la a 0. Nessa situação os valores que a função vai assumir, serão os valores que estaremos interessados (parâmetros e pesos).

Para ilustrar a situação, vamos pensar num problema simples, uma função que inclusive conseguiremos visualizar graficamente em duas dimensões.

$ f(x) = (x-2)^{2} $

Nesse caso, nós queremos saber, qual valor $x$ deve assumir para que eu tenha o menor valor de $f(x)$.

Para visualizar essa função, segue o código abaixo:

{{< admonition type=note title="Nota" open=true >}}
Todo código mostrado aqui pode ser encontrado no repositório dessa sequência de blog posts, ao final do capítulo você encontra o link.
{{< /admonition >}}

```python
import seaborn as sns
import numpy as np
import torch

def f(x):
    return torch.pow((x - 2.0), 2)

x_axis_vals = np.linspace(-10, 10, 100)
y_axis_vals_p = f(torch.tensor(x_axis_vals)).numpy()

sns.lineplot(x=x_axis_vals, y=y_axis_vals_p, label="$f(x)=(x-2)^2$")

```

{{<image width=800 src="https://i.imgur.com/PmPP787.png">}}

Ao aplicar regras de cálculo de derivada, para essa função teríamos:

$f'(x) = 2x - 4$

Logo, ao iguala-la a 0:

$x = 2$

Esse é o valor responsável por nos dar o mínimo da função $f(x)$.

Como disse acima, essa é uma representação simplificada do processo. **Normalmente o que acontece é o cálculo de gradiente**, que explicando de uma forma chula, é uma derivada para uma função que possui mais de uma variável ($g(x, z)$ por exemplo).

Na prática, não é um processo simples encontrar o valor exato do mínimo de uma função, até por que devido a complexidade dessas funções elas acabam tendo vários mínimos disponíveis.

Justamente por essas questões, que utilizamos um processo iterativo para tentar minimizar o máximo que for possível. Nesse processo, utilizamos um outro benefício do cálculo do gradiente, que é o sinal do resultado.

Sempre que é calculado o gradiente de uma função, o sinal vai nos indicar para onde devemos caminhar afim de encontrar esse ponto de minimização da função, ou seja, para quais valores de pesos e parâmetros eu preciso atualizar para caminhar para o mínimo global da minha _loss function_. Na prática, usamos o conceito de "Learning Rate" afim de ter uma cadência de como iremos caminhar nos valores dos nossos parâmetros, afim de minimizar a função.

Resumindo:

1. temos uma loss para minimizar
2. essa minimização ocorre através da variação de parâmetros
3. esses parâmetros são atualizados de acordo uma learning rate
4. sempre investigando qual o sinal do cálculo do gradiente

O gráfico abaixo mostra basicamente qual é o fluxo.

![](https://i.imgur.com/otdv1GE.png)

## O Pytorch nesse processo

Bom, provavelmente você deve ta imaginando a dor de cabeça que seria implementar toda essa matemática para as mais diversas arquiteturas que existem. Não se preocupe, o `Pytorch` ta aqui pra te auxiliar no processo, trazendo várias abstrações muito úteis na implementação de arquiteturas, treinamento, fine-tunning... de modelos de Deep Learning.

No ecossistema de frameworks de Deep Learning, o `Pytorch` seria um "concorrente" do `TensorFlow`. Basicamente os dois te permite realizar as mesmas atividades, mas recentemente tem tido uma adoção muito maior pela utilização do `Pytorch` em detrimento do `TensorFlow`. Vou deixar abaixo um gráfico de tendência de utilização de diversos frameworks em publicação de papers.

![](https://i.imgur.com/wzh3kTT.png)

Veja como a utilização do `TensorFlow` (em laranja) vem caindo ao longo do tempo. Mesmo esses papers sendo lançados primeiramente com implementação em `Pytorch`, não quer dizer que a comunidade aos poucos não faça uma implementação para o `TensorFlow`.

Para esse post era isso, espero que tenha curtido o conteúdo e no próximo continuaremos falando do `Pytorch` e `Pytorch-Lightining`.

## Fontes

- Livro: Inside Deep Learning
- Trend dos frameworks: https://paperswithcode.com/trends
- Código: https://github.com/LucianoBatista/generative-ai
