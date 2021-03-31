---
layout: post
title: "Criando uma API Pronta para Produção com FastAPI - PT.2"
date: 2021-03-30
author: "Luciano"
image: "img/post_0_4.png"
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

Neste tutorial iremos iniciar o **Econowallet**, essa aplicação que vai contar com uma API top para controle financeiro de suas despesas e investimentos, se você não viu o post passado (onde explico mais sobre meu objetivo com esse projeto) clique aqui. Tópicos que serão abordados nesse post:

- put here

# Setup

Como todo projeto python é uma boa prática que você crie um ambiente virtual isolado para instalar as dependências do projeto, isso evita que você possa ter conflito entre diferentes libs de outros projetos que esteja trabalhando. Para isso temos várias opções:

- Pip + virtualenv
- Poetry
- Pipenv
- Conda

Estarei utilizando aqui o Pipenv, sinta-se a vontade para usar o que tenha mais familiaridade.

Como IDE estarei usando o Pycharm 2020.3.5 (minha preferida no momento para desenvolver em python) e utilizaremos python 3.9.0. Ao clicar em "New Project" temos a seguinte imagem:

![Creating Project](/img/finance_app_tutorial/pt2/create_project.jpg)

Veja na lateral esquerda como o Pycharm já traz um template para diversos tipos de projeto, porém no momento ainda não temos a opção para o FastAPI. Após iniciado o projeto você deve ver a presença de um Pipfile (se está usando o Pipenv) ou apenas uma pasta vazia. Então iremos executar os seguintes comandos no terminal:

```bash
$ mkdir project
$ mkdir project/app
$ touch project/app/main.py
$ touch project/app/__init__.py
$ mv Pipfile project/
$ cd project
$ pipenv install fastapi==0.63.0
$ pipenv install uvicorn==0.13.4
```

Após esses comandos você deveria ter uma estrutura de diretórios semelhante à essa:

```bash
.
└── project
    ├── app
    │   ├── __init__.py
    │   └── main.py
    ├── Pipfile
    └── Pipfile.lock
```

Veja que além do FastAPI, instalamos o Uvicorn. O papel dessa lib é permitir que consigamos interagir com o serviço que estamos contruíndo (nosso web server), além disso ela utiliza o protocolo ASGI que permite assincronosidade com suporte aos verbos `async` e `wait` que o python possui. Outros frames como o Django e Flask já possuem um server built-in instalado que é muito útil para fases de desenvolvimento mas que pode gerar confusão nas etapas de produção pois o mesmo precisa ser substituído. Aqui, iremos ficar desde o início com o Uvicorn PUT DOC HERE.

## Main

Vamos configurar apenas uma rota com o mínimo de código possível para ver se nosso server está funcionando.

```python
# project/app/main.py

from fastapi import FastAPI

app = FastAPI()


@app.get("/ping")
def ping():
    return {
        "ping": "pong!"
    }
```

E então iremos rodar o uvicorn na instância que criamos do FastAPI na linha 5. O seu comando depende de qual diretório estiver:

- uvicorn project.app.main:app (do diretório root)
- uvicorn app.main:app (do diretório de projeto)

e assim em diante...

```bash
$ uvicorn project.app.main:app

INFO:     Started server process [654351]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```

Agora se navegarmos até http://127.0.0.1:8000/ping iremos ver a seguinte código:

```json
{
  "ping": "pong!"
}
```

Além disso o FastAPI automágicamente gerou um esquema baseado no padrão OpenAPI e juntamente com o Swagger UI criou uma documentação inicial (bem crua) para sua API, o que pode ser acessado em http://localhost:8000/docs ou ainda se não gosta da interface do Swager você pode acessar http://localhost:8000/redoc para uma interface diferente.

![Ping Pong Swager](/img/finance_app_tutorial/pt2/ping_pong_swager.jpg)

![Ping Pong Redocs](/img/finance_app_tutorial/pt2/ping_pong_redoc.jpg)

além disso, essa interface pode ser personalizada de acordo com seu projeto e necessidades, você pode encontrar detalhes na documentação clicando aqui https://fastapi.tiangolo.com/advanced/extending-openapi/

Agora iremos reiniciar nosso web server e adicionar um comando bem útil que iremos rodar da que em diante com o uvicorn (ctrl + c no terminal para matar o server atual):

```bash
$ uvicorn project.app.main:app --reload

INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [663530] using statreload
INFO:     Started server process [663532]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

Dessa forma, sempre que fizermos alguma alteração no código o Uvicorn vai reiniciar o server para nós. Experimente remover a exclamação do "pong!" e veja o que acontece.

## Config

Agora iremos adicionar um arquivo muito importante para nossa aplicação e que vai lidar com nossas variáveis de ambiente tanto localmente quanto quando estivermos no ambiente do Docker.

```python
# project/app/config.py

import logging
import os

from pydantic import BaseSettings

log = logging.getLogger("uvicorn")


class Settings(BaseSettings):
    environment: str = os.getenv("ENVIRONMENT", "dev")
    testing: bool = os.getenv("TESTING", 0)


def get_settings() -> BaseSettings:
    log.info("Loading config settings from the environment...")
    return Settings()

```

### O que está acontecendo aqui?

Primeiramente declaramos uma classe Settings com dois atributos:

- environment: que define nosso ambiente (prod, stage, dev)
- testing: que define se estamos ou não em um modo de teste

Outra coisa é que atente para a sintaxe da classe Settings (`testing: bool`), essa é uma forma de explicitar que tipo de variável nossa imputação terá, iremos utilizar muito ao longo do projeto e é muito útil pra que tenhámos menos bugs e que a IDE entenda os ins e outs de outras funções.
