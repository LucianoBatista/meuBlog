---
layout: post
title: "Criando uma API Pronta para Produção com FastAPI - PT.2"
date: 2021-04-03
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

![fastapi](/img/finance_app_tutorial/pt2/fast_api.png)

Neste tutorial iremos iniciar o **Econowallet**, essa aplicação que vai contar com uma API para controle financeiro de suas despesas e investimentos, se você não viu o post passado (onde explico mais sobre meu objetivo com esse projeto) [clique aqui](https://www.lobdata.com.br/post/finance_control_app/).

Tópicos que serão abordados nesse post:

- Setup Inicial: main.py e config.py
- Rotas async

# Setup

Como todo projeto python é uma boa prática que você crie um ambiente virtual isolando as dependências do projeto, isso evita que você possa ter conflito entre diferentes libs de outros projetos que esteja trabalhando. Para isso temos várias opções:

- Pip + virtualenv
- Poetry
- Pipenv
- Conda

Estarei utilizando aqui o Pipenv, sinta-se a vontade para usar o que tenha mais familiaridade.

Como IDE estarei usando o Pycharm 2020.3.5 (minha preferida no momento para desenvolver em python) e utilizaremos python 3.9.0. Ao clicar em "New Project" temos a seguinte imagem:

![Creating Project](/img/finance_app_tutorial/pt2/create_project.png)

Veja na lateral esquerda como o Pycharm já traz um template para diversos tipos de projeto, porém no momento ainda não temos a opção para o FastAPI. Após iniciado o projeto, você deve encontrar um chamado Pipfile (se está usando o Pipenv) ou apenas uma pasta vazia. Então iremos executar os seguintes comandos no terminal:

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

Aqui vale uma ressalva, não sou nenhum expert em arquitetura de software e talvez a estrutura de projeto utilizada aqui não seja a melhor possível, mas busco sempre dividir o código em blocos que contenham uma lógica ou função semelhante. Isso permite que tenhámos um código bom para da manutenção, bom para ser entendido e também para colaborar.

Após os comandos anteriores você deveria ter uma estrutura de diretórios semelhante à mostrada abaixo:

```bash
.
└── project
    ├── app
    │   ├── __init__.py
    │   └── main.py
    ├── Pipfile
    └── Pipfile.lock
```

Veja que além do **FastAPI**, instalamos o **Uvicorn**. O papel dessa lib é permitir que consigamos interagir com o serviço que estamos contruíndo (nosso web server), além disso ela utiliza o protocolo ASGI que permite assincronosidade com suporte aos verbos `async` e `await` do python. Outros frames como o Django e Flask já possuem um server built-in que é muito útil para fases de desenvolvimento mas que pode gerar confusão nas etapas de deploy pois o mesmo precisa ser substituído. Aqui, iremos ficar desde o início com o Uvicorn, saiba mais na documentação clicando no [link](https://www.uvicorn.org).

## Main

Vamos configurar apenas uma rota inicial com o mínimo de código possível para ver se nosso web server está funcionando.

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

E então iremos rodar o uvicorn buscando a instância que acabamos de criar do FastAPI (linha 5). Observe que o comando depende de qual diretório estiver:

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

Agora, se navegarmos até http://127.0.0.1:8000/ping iremos ver a seguinte resposta:

```json
{
  "ping": "pong!"
}
```

Além disso o FastAPI automágicamente gerou um esquema baseado no padrão OpenAPI e juntamente com o Swagger UI criou uma documentação inicial (bem crua) para sua API. Podendo ser acessada em http://localhost:8000/docs ou ainda se não gostou dessa interface do Swager você pode acessar http://localhost:8000/redoc para uma interface diferente.

![Ping Pong Swager](/img/finance_app_tutorial/pt2/ping_pong_swager.png)

![Ping Pong Redocs](/img/finance_app_tutorial/pt2/ping_pong_redoc.png)

...além disso, essa interface pode ser personalizada de acordo com seu projeto e necessidades, você pode encontrar detalhes na documentação [clicando aqui](https://fastapi.tiangolo.com/advanced/extending-openapi/).

Agora iremos reiniciar nosso web server e adicionar um comando bem útil que iremos utilizar daqui em diante com o uvicorn, o `--reload`:

```bash
$ uvicorn project.app.main:app --reload

INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [663530] using statreload
INFO:     Started server process [663532]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

Dessa forma, sempre que fizermos alguma alteração no código, o Uvicorn vai reiniciar o server para nós. Experimente remover a exclamação do "pong!" e veja o que acontece.

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

<img src="/img/finance_app_tutorial/pt2/what.gif" alt="what_gif" style="width:200px;"/>

![WHAT?!](/img/finance_app_tutorial/pt2/what.gif)

> O que está acontecendo aqui?

Primeiramente, declaramos uma classe `Settings` com dois atributos:

- `environment`: que define nosso ambiente (prod, stage, dev).
- `testing`: que define se estamos ou não em um modo de teste.

Outra ponto importante a ressaltar é a sintaxe da classe `Settings` (`testing: bool`), essa é uma forma de explicitar o tipo da variável que a nossa imputação retornará, iremos utilizar muito ao longo do projeto. **Pontos positivos** de adotar essa sintaxe ao longo do projeto são:

- código mais legível.
- auxilia a IDE a fornecer melhores formas de auto-complete e sugestões de métodos.
- previne alguns bugs, pois a IDE tem maior entendimento do que acontece com as funções, classes...

Agora iremos atualizar nosso arquivo main.py:

```python
# project/app/main.py

from fastapi import FastAPI, Depends

from project.app.config import Settings, get_settings

app = FastAPI()


@app.get("/ping")
def ping(settings: Settings = Depends(get_settings)):
    return {
        "ping": "pong!",
        "environment": settings.environment,
        "testing": settings.testing
    }

```

Dessa forma nós estamos setando dependências à aplicação sempre que acessamos a rota http://127.0.0.1:8000/ping. De uma forma mais intuitiva, o que estamos dizendo é o seguinte:

![Flow Dependencies](/img/finance_app_tutorial/pt2/flow_dependencies.png)

Agora, se rodarmos novamente nosso server, iremos ver uma resposta da seguinte forma:

```json
{
  "ping": "pong!",
  "environment": "dev",
  "testing": false
}
```

Legal, agora sabemos, por exemplo, que estamos no ambiente de desenvolvimento e não de teste. Outro ponto importante é que da forma como nosso config.py está, sempre que _batemos_ na rota http://127.0.0.1:8000/ping ele reconfigura as settings, porém nós não queremos esse comportamento. Pois futuramente ao migrarmos o carregamento das variáveis de ambient a partir de um arquivo (`.env` por exemplo) esse comportamento irá diminuir a velocidade que nosso app responde aos requests.

Usaremos então o decorator `@lru_cache` para cachiar as settings, de forma que get_settings é chamada apenas uma vez e reusará valores de chamadas recentes para atribuir às variáveis. Tenha em mente que isso é indicado apenas para casos onde você não tem a necessidade de recomputar esses valores várias vezes.

```python
# project/app/config.py

import logging
import os
from functools import lru_cache

from pydantic import BaseSettings

log = logging.getLogger("uvicorn")


class Settings(BaseSettings):
    environment: str = os.getenv("ENVIRONMENT", "dev")
    testing: bool = os.getenv("TESTING", 0)


@lru_cache()
def get_settings() -> BaseSettings:
    log.info("Loading config settings from the environment...")
    return Settings()
```

Agora se atualizarmos várias vezes o http://127.0.0.1:8000/ping não veremos o recarregamento do "Loading config settings from the environment...", indicando que nosso cache funcionou 🎉.

## Rotas Async

Como dito no início do post, estamos usando um web server ASGI e além disso um frame com suporte ao `async` e `await` (FastAPI). Portanto, podemos facilmente converter nosso endpoint para uma rota assíncrona simplesmente adicionando o verbo `async`.

```python
from fastapi import FastAPI, Depends

from project.app.config import Settings, get_settings

app = FastAPI()


@app.get("/ping")
async def ping(settings: Settings = Depends(get_settings)):
    return {
        "ping": "pong!",
        "environment": settings.environment,
        "testing": settings.testing
    }

```

Teste novamente a rota http://127.0.0.1:8000/ping para ver se tudo continua functionando.

> **ATENÇÃO!**: O async e await do python não está relacionado ao uso de mais threads ou de processamento paralelo. O conceito aqui é permitir que o código execute outras funcionalidades enquanto aguarda resposta de um outro serviço (server por exemplo).

Irei adicionar um .gitignore ao projeto para irmos adicionando coisinhas que não queremos expor no repositório:

```
__pycache__
env
```

É um bom momento para inicializar nosso repositório e comitarmos nosso código até o momento.

Como irei atualizando o repo do projeto de acordo for postando aqui no blog, é possível que quando você esteja lendo esse conteúdo o repositório esteja com o projeto completo. Mas, qualquer coisa, segue o link do repositório:

[GitHub - Econowallet](https://github.com/LucianoBatista/econowallet)
