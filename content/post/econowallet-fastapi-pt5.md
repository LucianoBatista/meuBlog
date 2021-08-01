---
layout: post
title: "Criando uma API Pronta para Produção com FastAPI - PT.5"
date: 2021-08-01
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

## Nos Capítulos Anteriores ...

Como dito anteriormente, hoje vamos configurar o **SQLAlchemy**!!

Essa é a parte 5 do nosso projeto do **EconoWallet** e se você quiser verificar o que já fizemos até o momento, acesse os links abaixo:

- [Parte 1](https://www.lobdata.com.br/2021/03/30/criando-uma-api-pronta-para-produ%C3%A7%C3%A3o-com-fastapi-pt.1/)
- [Parte 2](https://www.lobdata.com.br/2021/04/03/criando-uma-api-pronta-para-produ%C3%A7%C3%A3o-com-fastapi-pt.2/)
- [Parte 3](https://www.lobdata.com.br/2021/04/08/criando-uma-api-pronta-para-produ%C3%A7%C3%A3o-com-fastapi-pt.3/)
- [Parte 4](https://www.lobdata.com.br/2021/07/31/criando-uma-api-pronta-para-produ%C3%A7%C3%A3o-com-fastapi-pt.4/)
- [Parte 5](https://www.lobdata.com.br/2021/08/01/criando-uma-api-pronta-para-produ%C3%A7%C3%A3o-com-fastapi-pt.5/)

Sem enrolação, vamos logo ao que interessa!!!

![I'm ready when you are](/img/finance_app_tutorial/pt4/letscode.gif)

## Reorganizando o projeto

Antes de configurar o ORM, vamos ajustar algumas coisas no nosso diretório. Como estamos seguindo para um estágio mais maduro da aplicação e que você já compreende o básico de configuração do FastAPI, precisaremos agora **separar melhor as competências** dentro do projeto.

```bash
mkdir project/app/api      
mkdir project/app/database           
mkdir project/app/models  
touch project/app/api/__init__.py           
touch project/app/database/__init__.py         
touch project/app/models/__init__.py        
touch project/app/api/status.py     
```

Veja que agora nós temos alguns diretórios novos✨ que irão comportar diferentes lógicas no nosso projeto:

- **api**: todas as nossas rotas da api.
- **database**: toda a lógica de configuração do SQLAlchemy.
- **models**: toda a especificação de como deverão ser nossas tabelas.

Nós também adicionamos um arquivo chamado `status.py`, esse irá conter a rota de status da nossa aplicação, que antes estava no nosso arquivo `main.py`. Veja como está nosso `main.py` e nosso `status.py`:

```python
# main.py
import logging

from fastapi import FastAPI

from app.api import status

log = logging.getLogger("uvicorn")


def create_application() -> FastAPI:
    application = FastAPI()
    application.include_router(
        status.router,
        tags=["Hello"],
        prefix="/api/v1"
    )
    return application


app = create_application()


@app.on_event("startup")
async def startup_event():
    log.info("Starting up...")
```

Aqui nós não configuramos mais diretamente as rotas no `main.py`, elas agora ficam em diretórios separados, e sempre na inicialização da aplicação é carregado todas as rotas ao chamar a função `create_application`. Dentro dessa função nós adicionamos um `include_router` que faz algumas coisas interessantes:

- inclui todas as rotas de um determinado arquivo.
- adiciona uma tag para separar melhor no contexto de exibição no swager.
- adiciona um prefixo à rota: eu normalmente utilizo `/api/v1` pelo motivo de que fica fácil alterar para uma versão 2 caso a mesma venha a existir.

Um decorator muito legal do FastAPI é `@app.on_event`, onde você pode configurar métodos que irão rodar sempre que a aplicação inicializar ou finalizar. Iremos adicionar mais coisas nesse *event*, mas por enquanto apenas estamos printando um log no console.

```python
# status.py
import os

from fastapi import APIRouter, Depends

from app.config import Settings, get_settings

router = APIRouter()
APP_VERSION = os.getenv("APP_VERSION", "1.0.0")


@router.get("/status")
async def ping(settings: Settings = Depends(get_settings)):
    return {
        "ping": "pong!",
        "version": APP_VERSION,
        "environment": settings.environment,
        "testing": settings.testing
    }
```

Veja que aqui nós temos uma modificação da rota */ping* para */status* e ela é definida por *@router.get* e não mais *@app.get*. Eu adicionei também uma variável chamada `APP_VERSION` que é responsável por manter atualizado o registro da verssão da aplicação.

Executamos assim nosso workflow de teste, para saber se tudo está funcionando:

```bash
docker-compose down -v
docker-compose up --build -d
```

Ao acessar [http://127.0.0.1:8004/docs/](http://127.0.0.1:8004/docs/) você deverá ver uma tela igual a de baixo:

![status](/img/finance_app_tutorial/pt4/status-router.png)

E nossa árvore de diretórios deve estar como no seguinte snippet:

```bash
.
├── docker-compose.yml
├── Dockerfile
├── Pipfile
├── Pipfile.lock
├── project
│   └── app
│       ├── api
│       │   ├── __init__.py
│       │   └── status.py
│       ├── config.py
│       ├── database
│       │   └── __init__.py
│       ├── __init__.py
│       ├── main.py
│       └── models
│           └── __init__.py
└── README.md
```

## Configurando o SQLAlchemy no Projeto

No intuito de ter uma tabela inicial bem simples no nosso banco de dados, apenas para iniciar a configuração do SQLAlchemy, vamos iniciar o processo criando um model. Como dito no post [anterior](https://www.lobdata.com.br/2021/08/01/criando-uma-api-pronta-para-produ%C3%A7%C3%A3o-com-fastapi-pt.4/), estamos no que estou chamando de *primeiro cenário do SQLAlchemy*, onde precisamos da classe `declarative_base` antes de iniciar a configuração do model.

Antes de tudo, primeiro iremos instalar o **SQLAlchemy**: `pipenv install sqlalchemy==1.4.20`. Agora, vamos criar dois novos arquivos:

```bash
touch project/app/database/modelbase.py
touch project/app/models/register.py
```

O `modelbase.py` é onde eu normalmente coloco a `declarative_base`, sendo assim, é desse arquivo que nossos models irão buscar a **classe mãe**.

```python
# modelbase.py
from sqlalchemy.ext import declarative

Base = declarative.declarative_base()
```

Podemos agora criar o model. Um model é uma abstração de tudo que você faria se estivesse criando uma tabela diretamente no banco, com a diferença de que aqui a gente tem uma abordagem *python-like*. Cada tipo de ORM tem uma sintaxe própria, e no SQLAlchemy a gente tem a configuração como no arquivo abaixo:

```python
# register.py
from datetime import date

from sqlalchemy import Column, Date, BigInteger, String

from app.database.modelbase import Base


class Register(Base):
    __tablename__ = "registers"

    id: int = Column(BigInteger, primary_key=True, autoincrement=True)
    products: str = Column(String(20), nullable=False)
    created_at: date = Column(Date, nullable=False, index=True)
    expire_at: date = Column(Date, nullable=False, index=True)
```

Pronto, nosso primeiro model está criado. Como você pode ver, é basicamente uma classe que herda da `declarative_base`, e nosso dever é configurar como vai ser cada coluna da tabela. Diferentemente de outros ORMs como o `django`, aqui nós precisamos explicitamente determinar a coluna de id.

Para as outras colunas, nós temos formato de string e data. Sendo que nenhuma delas podem ser atribuídos valores nulos, ou seja, são campos obrigatórios da nossa tabela. Veja que eu já estou adicionando alguns indexes, que basicamente permitem que a consulta à essas colunas seja feita de forma mais rápida.

Até agora nós já temos alguns elementos do primeiro cenário do SQLAlchemy, porém, para criar as tabelas no banco e expor uma session, nós iremos precisar criar um novo arquivo: `touch project/app/database/database_session.py`.

```python
# database_session.py
import logging
import os
from typing import Callable, Optional

import sqlalchemy
from sqlalchemy.orm import Session

from app.database.modelbase import Base

__factory: Optional[Callable[[], Session]] = None
log = logging.getLogger("uvicorn")


def get_db() -> Session:
    db = create_session()
    try:
        yield db
    finally:
        db.close()


def global_init() -> None:
    global __factory

    if __factory:
        return

    conn_str = str(
        os.environ.get("DATABASE_URL", "sqlite:///project/db/local_database.db")
    )
    log.info("Connecting to the database...")
    engine = sqlalchemy.create_engine(conn_str, echo=False)
    __factory = sqlalchemy.orm.sessionmaker(bind=engine)

    from app.models.register import Register

    Base.metadata.create_all(engine)


def create_session() -> Session:
    global __factory

    if not __factory:
        raise Exception("You must call global_init() before using this method")

    session: Session = __factory()
    session.expire_on_commit = False

    return session
```

![what?](/img/finance_app_tutorial/pt4/what.gif)

Calma, vou traduzir o que está acontecendo aqui:

1. Criamos uma função para inicializar uma variável global chamada `__factory`, essa é responsável por expor uma conexão com o banco para toda a aplicação.
1. Por meio do `create_session()` essa variável é acessada e a session é enfim estabelecida.
1. Setamos uma forma padrão de acessar essa session por meio da função `get_db()`, que expõe a session e garante que a mesma vai ser finalizada (`finaly`), **aconteça o que acontecer**.

Eu ainda não comentei sobre variáveis de ambiente, mas vai ser algo que precisaremos fazer aqui. Uma variável de ambiente é basicamente um valor atribuído dinamicamente que pode afetar o modo como alguns processos irão se comportar em seu projeto.

Em python, normalmente acessamos essas variáveis utilizando a lib `os`, builtin da linguagem, por meio do comando: `os.environ.get("NOME_DA_VARIAVEL")`.

### Alerta Boas Práticas 🚨

Gostaria de pontuar aqui a importância de utilizarmos variáveis de ambiente em nossos projetos. Normalmente projetos reais possuem informações sensíveis que pessoas externas ao projeto não podem ter acesso, ou informações que precisam ser alteradas dinamicamente, como:

- Senha do banco
- Token de um bucket
- Token de alguma API de terceiro
- Senha de login em outro serviço
...

É extremamente indicado não **hard coded** essas informações diretamente no código. Ao invés disso, que coloquemos as mesmas como variáveis de ambiente em um arquivo separado e oculto do repositório onde você está versionando o seu projeto. Vamos ver então como fazer isso!!

### Configurando Variáveis de Ambiente

Graças ao Docker, é simples configurar variáveis de ambiente para nosso projeto. Primeiro, vamos criar alguns arquivos:

```bash
mkdir env
touch env/.dev
```

Dentro do `.dev` é onde você vai colocar todas as suas variáveis de ambiente:

```
ENVIRONMENT=dev
TESTING=0
DATABASE_URL=mysql://user:password@db:3306/econowallet

SQL_DATABASE=econowallet
SQL_USER=user
SQL_PASSWORD=password
SQL_HOST=db
SQL_PORT=3306
```

E agora, atualizamos nosso `.gitignore`:

```bash
# .gitignore
__pycache__
# new
env
```

Show! Agora, localmente teremos como testar nossa aplicação, e quando a mesma for funcionar em um ambiente de produção, essas variáveis de ambiente serão substituídas dinamicamente para funcionar com as configurações de produção 😍.

Por fim, dentro do `docker-compose.yml` nós iremos apontar onde o arquivo de variáveis de ambiente está localizado:

```yaml
version: '3.8'

services:
  web:
    build: .
    volumes:
      - .project:/usr/src/app
    ports:
      - 8004:8000
    # new
    env_file:
      - env/.dev
    environment:
      - ENVIRONMENT=dev
      - TESTING=0
    depends_on:
      - db

  db:
    image: mysql:8.0
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    volumes:
      - mysql_data:/var/lib/mysql_data/data/
    environment:
      MYSQL_DATABASE: econowallet
      MYSQL_USER: user
      MYSQL_PASSWORD: password
      MYSQL_ROOT_PASSWORD: password
    ports:
      - 3320:3306

volumes:
  mysql_data:
```

### Inicializando em Conjunto com o SQLAlchemy

Lembrando que precisamos inicializar uma sessão global por meio do `global_init()` que vimos anteriormente, vamos então fazer uso de uma feature do FastAPI que vimos anteriormente, `@app.on_event()`:

```python
import logging

from fastapi import FastAPI

from app.api import status
from app.database.database_session import global_init

log = logging.getLogger("uvicorn")


def create_application() -> FastAPI:
    application = FastAPI()
    application.include_router(
        status.router,
        tags=["Hello"],
        prefix="/api/v1"
    )
    return application


app = create_application()

# updated
@app.on_event("startup")
async def startup_event():
    log.info("Starting up...")
    global_init()
```

Sendo assim, sempre que o app inicializar, vamos iniciar as configurações do SQLAlchemy em conjunto!! 

Estamos quaseee lá!

### Bug do MySQL 🐛

Se você tentar buildar o projeto, provavelmente irá se deparar com um problema onde o serviço web não irá conseguir inicializar pois não vai ter encontrado um banco MySQL disponível. Eu já busquei explicação do por que disso acontecer, mesmo especificando o `depends_on: db` no `docker-compose.yml` o serviço **web** tenta inicializar primeiro, e como não encontra um banco online, temos uma mensagem de erro.

Para driblar isso, vamos utilizar um pouquinho de conhecimento de bash e adicionar o que chamamos de `entrypoint` para garantir que nosso serviço web apenas siga em frente após conectar com o banco.

Crie um arquivo chamado `local-entrypoint.sh` e copie o seguinte código:

```bash
#!/bin/bash
# if any of the commands in your code fails for any reason, the entire script fails
set -o errexit
# fail exit if one of your pipe command fails
set -o pipefail
# exits if any of your variables is not set
set -o nounset

mysql_ready() {
python << END
import sys

import mysql.connector
from mysql.connector import Error

try:
    connection = mysql.connector.connect(host="${SQL_HOST}",
                                         database="${SQL_DATABASE}",
                                         user="${SQL_USER}",
                                         password="${SQL_PASSWORD}")
except:
    sys.exit(-1)
sys.exit(0)

END
}
until mysql_ready; do
  >&2 echo 'Waiting for Mysql to become available...'
  sleep 1
done
>&2 echo 'Mysql is available'

uvicorn project.app.main:app --reload --workers 1 --host 0.0.0.0 --port 8000
```

Nesse script é executado um loop que finaliza apenas quando o MySQL estiver disponível. Precisamos agora apontar nosso `Dockerfile` para esse entrypoint.

```dockerfile
# Dockerfile
FROM python:3.9.0-slim-buster

WORKDIR /usr/src/app

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

RUN apt-get update \
  && apt-get -y install netcat gcc \
  # mysql dependencies
  && apt-get -y install default-libmysqlclient-dev build-essential \
  && apt-get clean

RUN pip install --upgrade pip \
  && pip install pipenv
COPY ./Pipfile .
COPY ./Pipfile.lock .
RUN pipenv install --deploy --system

COPY . .

# entrypoint
COPY ./local-entrypoint.sh /usr/src/app/local-entrypoint.sh
RUN chmod +x /usr/src/app/local-entrypoint.sh

CMD ["/bin/bash", "-c", "./local-entrypoint.sh"]
```

Nosso `docker-compose.yml` também precisará ser alterado:

```yaml
# docker-compose.yml
version: '3.8'

services:
  # updated
  web:
    build: .
    volumes:
      - .:/usr/src/app
    ports:
      - 8004:8000
    env_file:
      - env/.dev
    environment:
      - ENVIRONMENT=dev
      - TESTING=0
    depends_on:
      - db

  db:
    image: mysql:8.0
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    volumes:
      - mysql_data:/var/lib/mysql_data/data/
    environment:
      MYSQL_DATABASE: econowallet
      MYSQL_USER: user
      MYSQL_PASSWORD: password
      MYSQL_ROOT_PASSWORD: password
    ports:
      - 3320:3306

volumes:
  mysql_data:
```

Aqui tivemos **duas mudanças**:
1. nosso volume não é referente ao `.project` e sim a pasta root `.` (caso contrário o `local-entrypoint.sh` não será encontrado).
1. removemos o `command` que estava inicializando app, e jogamos para dentro do entrypoint.

Por fim, garanta a instalação do drive do MySQL e do pacote `mysql-connector-python`:

```bash
pipenv install mysqlclient==2.0.3
pipenv install mysql-connector-python==8.0.25
```

### Refatoração 🔨

No início do projeto nós fizemos uma configuração no PyCharm para considerar o diretório **project** como root. Porém, teremos que retornar para o diretório anterior (efetuando os mesmos passos [link de como fazer](https://www.lobdata.com.br/2021/04/08/criando-uma-api-pronta-para-produ%C3%A7%C3%A3o-com-fastapi-pt.3/).

E além disso, iremos alterar todas as referências nos nossos arquivos de volta para `project.app`... Os arquivos que precisarão sofrer essa refatoração são:

- `status.py`
- `database_session.py`
- `modelbase.py`
- `register.py`
- `main.py`

Sorry!

Agora sim, vamos rebuildar a aplicação e observar no DBeaver se veremos uma nova tabela (como especificado nosso model).

```bash
docker-compose down -v
docker-compose up --build -d
docker-compose logs -f
```

![dbeaver-registers](/img/finance_app_tutorial/pt4/dbeaver-registers.png)

🎊 🎉 Agora temos nossa tabela materializada no banco e podemos iniciar as transações!! 🎊 🎉


## Próximo Capítulo...

Na próxima etapa, iremos configurar quais rotas teremos na aplicação. Assim como que as rotas interagem com o banco de dados 👋🏽.

Você pode acompanhar o repositório do projeto no link abaixo:

- [GitHub - Econowallet](https://github.com/LucianoBatista/econowallet) 🥂
