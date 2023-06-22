---
layout: post
title: "Criando uma API Pronta para Produção com FastAPI - PT.1"
date: 2021-03-30
author: "Luciano"
featuredImage: "img/fastapi.png"
tags:
  - RESTFull API
  - Web Development
  - Backend
  - Asynchronous
  - FastAPI
  - Docker
  - SQLAlchemy
categories: [Tutorials]
---

![Personal Finance](/img/finance_app_tutorial/finance.jpg)

# Motivação

Atualmente tenho tido bastante dor de cabeça em voltar a utilizar o Excel para fazer uma planilha de controle de gastos, apesar da simplicidade do software, constantemente meus registros de compras feitas ou alguma outra movimentação financeira ficam uma completa bagunça. O que pode ser justificado pela minha falta de destreza com a ferramenta, mas fato é que a experiência estava deixando a desejar e consequentemente acabei perdendo a disciplina de organizar meus gastos, o que não recomendo a ninguém.

Foi então que me veio a ideia de largar as planilhas e armazenar essa informação em um banco de dados, o que me facilitaria muita coisa, como por exemplo: facilidade para geração de um relatório com o nível de personalização muito superior ao Excel (utilizando **R** ou **Python** para consumir os dados e fazer a parte analítica), manter a integridade dos dados de forma confiável e robusta, criar modelos de machine learning para prever próximos gastos e muito mais.

Coincidentemente a tudo isso, tenho brincado bastante com o **FastAPI**, um framework fantástico para criação de RESTFull APIs utilizando linguagem python.

E, por fim, um último quesito foi que vejo poucos tutoriais em português que trazem um material interessante sobre como criar APIs com o **FastAPI** em um nível pronto (ou mais próximo possível) para produção.

# Overview do app

Diante do que foi dito acima, decidi criar um projeto com várias tecnologias que eu considero super interessante e que vão auxiliar na entrega de um produto bem robusto e escalável (não que fosse necessário para essa situação).

Então, juntamente com o **FastAPI**, vamos desenvolver usando um ambiente **Docker** e torná-lo o mais próximo possível de um ambiente de deploy. Utilizaremos o **SQLAlchemy** como nosso ORM (Object Relational Mapper) para "conversar" com o banco de dados (**Postgres**). Ainda iremos tentar desenvolver utilizando o **TDD** (Test Driven Development) com o **pytest**. Irei também adicionar o **Celery**, que vai permitir que tenhámos um produto altamente escalável ao final do projeto. Por fim, configuraremos GitHub Actions para implementar entrega e integração contínua (CI/CD) entre o desenvolvimento e o deploy no **Heroku**.

![Mix Techs](/img/finance_app_tutorial/mix_techs.png)

Muito provavelmente existem soluções prontas e aplicativos para controle financeiro, mas minha experiência com eles é justamente a falta de liberdade em criar algo personalizado para as minhas necessidades.

# Tecnologias

Nesse projeto vou trazer muitas ferramentas como citado acima e com código que você poderá reutilizar nos mais variados projetos. A seguir eu trouxe um pouco sobre cada tecnologia que será utilizada.

## FastAPI

Como disse antes, tenho tido muito contato com o **FastAPI** e é incrível como é intuitivo codar com este framework. Abaixo eu trouxe alguns motivos que fazem o **FastAPI** diferenciado dos outros frameworks em python (retirados da própria documentação).

- Tão rápido quanto NodeJS e GO (FastAPI é construído em cima do Starlette e Pydantic)
- Muito rápido para desenvolver APIs (200% a 300% em comparação a concorrentes)
- Redução de bugs
- Intuitivo
- Fácil de aprender
- Robusto
- _Automagicamente_ gera documentação para a API e JSON Schemas (**Muito Legal**)

Acesse a [documentação](https://fastapi.tiangolo.com) do FastAPI para mais detalhes, que por sinal é muito bem feita.

## Docker

Docker é uma plataforma de construção de containers utilizada para simular ambientes de desenvolvimento e deploy de forma isolada do seu sistema, ou seja, independentemente do seu SO (sistema operacional) você consegue ter o mesmo comportamento da aplicação pois cada container contém todas as dependências necessárias para sua execução.

## Pytest

**pytest** é um framework para escrever tests em Python, ele torna fácil e divertido escrever, organizar e rodar tests, quando comparado com o **unittest** (outro framework bastante conhecido). Abaixo cito algumas características do pytest:

- Requer menos código _boilerplate_
- Suporta declaração `assert` (built-in do Python) ao invés de `assertSomething` (como no unittest)
- Atualizado frequentemente
- Possui as famosas _fixtures_ que auxiliam na inicialização e finalização de configurações para determinados testes.
- Utiliza uma abordagem funcional

## SQLAlchemy

**SQLAlchemy** faz parte de um conjunto de frameworkers que são utilizados para abstrair a complexidade de lidar com o banco de dados diretamente. Desenvolver com um ORM não te prende à um banco de dados específico, onde posteriormente é possível plugar a aplicação ao Postgres, MySQL, Oracle, MariaDB ou outros (contanto que o framework tenha suporte). Escolhi o **SQLAlchemy** por alguns motivos:

- pela robustes: o SQLAlchemy possui diferentes módulos (Core e ORM) que te permite fazer muita coisa legal.
- muito tempo presente na comunidade: o que torna mais fácil lidar com problemas e bugs caso eles apareçam (e irão aparecer =D).
- ter sido bastante utilizado e testado em produção: isso atribui maior confiabilidade ao software.

## Celery

O **Celery** é um _task queue manager_ utilizado para implementar processamento assíncrono ao seu código. Fique atento que não estamos falando aqui de utilização de mais threads para realização de uma task, e sim de desiginar as tasks à diferentes "workers", o que possibilita um escalonamento horizontal e por isso acaba sendo muito atrativo ao design de microserviços.

![Celery Schema](/img/finance_app_tutorial/celery.jpeg)

O Celery é utilizado na arquitetura de várias tecnologias de Big Data como **Apache SuperSet** e **Apache Airflow**, justamente por ser um framework muito útil para escalabilidade.

## GitHub

O GitHub Actions é uma ferramente de integração contínua e entrega contínua (CI/CD), totalmente integrada com o GitHub. Temos outras plataformas que oferecem o mesmo serviço como:

- GitLab
- AWS (CodeCommit, CodeBuild, CodeDeploy, ECR)
- GCP (Cloud Source Repositories, Cloud Build, Container Registry)
- Azure (Repos, Pipelines, Container Registry)

Mas como já utilizo o GitHub para outros projetos, resolvi seguir com a mesma plataforma.

## Heroku

Heroku é uma cloud _Platform as a Service_ (PaaS) que provê um local para deployar aplicações web. Eles abstraem muita da complixidade de infraestrutura e torna fácil hospedar aplicações com segurança e escalabilidade. Para esse projeto irei utilizar planos gratuitos, para que todos possam seguir o passo-a-passo. Porém, como nossa aplicação vai estar conteinerizada você pode deployar em basicamente em qualquer plataforma cloud que existe com poucas modificações no código. Mas aqui iremos ficar com o Heroku.

Próximos tutoriais iremos partir para o hands-on!!!

![Coding fast](/img/finance_app_tutorial/coding_fast.gif)

Até lá!
