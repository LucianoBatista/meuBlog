---
layout: post
title: "#mynotes-01"
subtitle: "Generative Modeling"
date: 2023-06-21
author: "Luciano"
authorLink: "https://www.linkedin.com/in/lucianobatistads/"
featuredImage: "img/generative-ai-pt1.png"
tags:
  - AI
  - Generative
  - IA Generativa
categories: [Tutorials]
draft: false
---

Estou iniciando um série de posts no blog sobre IA Generativa. O conteúdo postado aqui serão minhas anotações sobre um livro que estou lendo, e a intenção é que a cada capítulo seja um compilado do que aprendi em junção com algum material extra que venha a consumir.

Teremos teoria e código, já que o livro segue essa linha, que eu particularmente curto bastante. Um adendo é que o hands-on do livro é em `keras`, nada contra, mas eu sou time `torch` :fire: :sweat_smile:.

O livro em questão é o _Generative Deep Learning - Teaching Machines to Paint, Write, Compose and Play_, estarei utilizando a segunda versão. Além disso, qualquer referência extra que eu utilizar estará linkada abaixo.

## Generative Modeling

{{< admonition type=tip title="Conceituando" open=true >}}

IA Generativa é um ramo do Machine Learning que involve treinamento de um modelo para geração de novos dados que sejam similares a um dado dataset.

{{< /admonition >}}

Um ponto interessante que já me chamou atenção: _"...gerar novos dados que são similares há um dado dataset"_. Isso evidencia mais ainda a importância do que você vai jogar para dentro do seu modelo, logo, a qualidade do seu dataset é muito importante.

## Probabilístico vs Determinístico

**Um modelo generativo é probabilístico e não determinístico** (o que já nos abre os olhos para vermos muita probabilidade ao longo dos próximos capítulos). Isso quer dizer que nós não queremos gerar sempre a mesma imagem, nós queremos na verdade uma certa variância nos nossos outputs.

![img](/img/posts/gen-ai-01/horse.png)

Sobre essa perspectiva, podemos imaginar que existe uma distribuição de probabilidade desconhecida que explica o por que algumas imagens são similares e outras não. Logo, nosso trabalho durante a modelagem **é encontrar uma distribuição o mais próxima possível dessa distribuição desconhecida**, e por meio dela sermos capazes de amostrar elementos que sejam similares aos elementos da origem (dados de treino) - vai ficar mais claro ao longo post =D.

O autor ainda deixa mais claro a diferença de generativo e discriminativo:

- Machine learning prevendo se uma imagem é ou não um quadro de Van Gogh: **Discriminativo**
- Machine learning capaz de gerar obras similares as obras de Van Gogh: **Generativo**

E claro, não poderia faltar algumas notações científicas:

- Discriminativo: p(y|x)
- Generativo estimamos a p(x)

## O crescimento da IA Generativa

![img](/img/posts/gen-ai-01/growth-ai.png)

O respaldo que a IA Generativa está tendo na indústria é incontestável. Um exemplo em particular que me chamou atenção, foi sobre a geração de dados artificiais para utilização como dados de input para treinamento de um modelo.

Há pouco tempo atrás, era impossível gerar imagens tão realistas como as que estão sendo geradas nos últimos meses. **Esse nível de qualidade abre margem para por exemplo a utilização desses modelos para gerar imagens da retina de um ser humano com presença de alguma anormalidade de difícil detecção, e assim treinarmos um modelo de detecção.**

Lógico que aqui existem muitas preocupações envolvidas, mas só desse tipo de possibilidade existir, pra mim, é surreal.

## As 3 Fortes razões

O autor trás um pensamento bem peculiar que até então eu nunca tinha me questionado ou lido em outro lugar. Ele cita que a evolução da IA Generativa trás 3 fortes razões para desbloquear/possibilitar uma forma de IA muito mais sofisticada do que a que temos hoje.

### Razão 1

A IA Generativa permite um entendimento mais completo do seu dado. Não se limitando a apenas categorização, mas sim tendo um entendimento da distribuição dos dados que permeiam determinada categoria.

### Razão 2

A IA Generativa resvala em outros campos da IA, como por exemplo o Reinforcement Learning. No famoso Aprendizado por Reforço, existe a ação de um agente que aprende uma determinada task, como por exemplo um robô que aprender a andar em um determinado terreno. Com a IA Generativa você é capaz de criar um mundo de possibilidades as quais esses robôs podem aprender tornando o processo mais eficiente.

### Razão 3

Qualquer forma de Inteligência que venha a ser desenvolvida para se assemelhar a um humano, precisa ter um poder generativo muito incrível, pois somos máquinas generativas muito incríveis. Estudos neurocientíficos sugerem que a nossa percepção de realidade não é algo discriminativo, mas sim generativo, e que "nosso modelo" está sendo treinado desde o nosso nascimento. Se você observar, a todo momento estamos gerando simulações ao nosso redor:

- gerando idealizações de futuro
- imaginando um final para uma série
- escrevendo esse artigo

![mind-blow](https://media0.giphy.com/media/PnXK4DIpAtgwpd2lSH/giphy.gif?cid=ecf05e47fqkye6rh9m3xcqwzab3ju6zhjbzubbq5gwl93zcp&ep=v1_gifs_search&rid=giphy.gif&ct=g)

## Nossa primeira IA Generativa

Nesse capítulo nós ainda não vamos codar (sinto te informar kkk). Mas é uma abstração bem legal e que ajuda a fixar o entendimento sobre distribuição real dos seus dados vs distribuição que o seu modelo consegue replicar.

Imagine a seguinte imagem:

![img](/img/posts/gen-ai-01/model-1.png)

Agora imagine que essa distribuição de pontos segue um determinado padrão, e que você precisa encontrar esse padrão (modelo), que vai permitir que você amostre diferentes pontos correspondentes a distribuição real dos seus dados.

![img](/img/posts/gen-ai-01/model-2.png)

Nesse caso, o autor utilizou um retângulo para descrever esse modelo. Logo, qualquer ponto gerado a partir dessa representação, seria um ponto semelhante a distribuição real.

![img](/img/posts/gen-ai-01/model-3.png)

Acontece que, pela imagem acima, na verdade o modelo que descreveria aonde nós temos pontos, seriam regiões continentais do mapa mundi. E sendo assim, os pontos gerados nas regiões de oceano, estariam fora dessa distribuição.

## Latent Space (Espaço Latente)

Pra mim, esse é o conceito mais importante do capítulo!! Fica claro ao final do capítulo que uma ideia chave que estamos tentando resolver é encontrar uma forma de representação de um espaço multidimensional, em um espaço menor porém representativo. E é justamente isso que é o espaço latente.

Por exemplo, vamos imaginar que para descrever o rosto de uma pessoa a gente precise de 100 características visuais. Agora imagine que você consiga criar um modelo que com apenas 3 características seja possível descrever o mesmo rosto, e ainda mapear essas três características para o espaço das 100 características iniciais.

Dessa forma você ta saindo do espaço latente e voltando a representação de alta dimensionalidade. E é justamente isso que queremos aqui, descobrir esse espaço latente para os mais variados de problemas.

Se você tem um background estatístico, ou de machine learning, já deve ter utilizado o PCA para redução de dimensionalidade, de forma simplificada é mais ou menos isso que estamos fazendo aqui.

## Taxonomia da IA Generativa

Para finalizar, temos aqui uma categorização das diferentes abordagens nesse universo generativo, de forma geral, são citados 3 possíveis abordagens:

- _Explicitamente modelar a função de densidade_
- _Explicitamente modelar uma aproximação tratável da função de densidade_
- _Implicitamente modelar a função de densidade, através de um processo estocástico que gera dados diretamente_

Espero entender mais nos próximos capítulos :sweat_smile:! É sobre, até a próxima e no capítulo 2 vamos falar um pouquinho sobre CNNs e vamos codar um pouquinho!!!
