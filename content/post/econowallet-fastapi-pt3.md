---
layout: post
title: "Criando uma API Pronta para Produção com FastAPI - PT.3"
date: 2021-04-08
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

# Nos Capítulos Anteriores ...

Essa é a parte 3 do nosso projeto do EconoWallet, se quiser verificar o que já fizemos até o momento, verifique os links abaixo:

- [Parte 1](https://www.lobdata.com.br/2021/03/30/criando-uma-api-pronta-para-produ%C3%A7%C3%A3o-com-fastapi-pt.1/)
- [Parte 2](https://www.lobdata.com.br/2021/04/03/criando-uma-api-pronta-para-produ%C3%A7%C3%A3o-com-fastapi-pt.2/)
- [Parte 3](https://www.lobdata.com.br/2021/04/08/criando-uma-api-pronta-para-produ%C3%A7%C3%A3o-com-fastapi-pt.3/)
- [Parte 4](https://www.lobdata.com.br/2021/07/31/criando-uma-api-pronta-para-produ%C3%A7%C3%A3o-com-fastapi-pt.4/)
- [Parte 5](https://www.lobdata.com.br/2021/08/01/criando-uma-api-pronta-para-produ%C3%A7%C3%A3o-com-fastapi-pt.5/)

Dando continuidade a nossa aplicação, hoje vamos iniciar a configuração do Docker em nosso ambiente. E antes de mais nada, precisaremos instalar o Docker e o Docker Compose na nossa máquina.

![Let's Do This](/img/finance_app_tutorial/pt3/lets_do_it.gif)

A [documentação](https://docs.docker.com/get-docker/) do Docker para instalação é muito bem explicada. Ao acessar o link você encontrará um passo-a-passo para qualquer SO que esteja usando (macOS, Linux ou Windows). Ao final da instalação, o próprio tutorial indica como verificar se está tudo funcionando corretamente, a versão que estarei utilizando aqui é:

```bash
$ docker -v
Docker version 20.10.5, build 55c4c88
```

```bash
$ docker-compose -v
docker-compose version 1.27.4, build 40524192
```

O workflow que vamos seguir aqui é o de containerizar cada parte da aplicação e depois orquestrar isso de alguma forma, para que cada parte se comunique de forma harmônica e a aplicação final funcione através dos containers e imagens criadas. Nesse processo, cada "bloco" da aplicação vai ser construído utilizando um template de instruções, nossos Dockerfiles, e a orquestração aqui vai ser feita por um arquivo chamado docker-compose.yml.

Ao final desse workflow você vai ter um _bundle_ onde cada container contêm parte da aplicação, mas tudo funciona em conjunto. Veja na imagem (de forma simplificada) o que acontece.

![Containerized Application](/img/finance_app_tutorial/pt3/container_application.png)

> Imagem retirada do Google

> Uma ressalva que gostaria de fazer é que existem outras formas de orquestrar os containers que não seja utilizando um arquivo docker-compose.yml, como por exemplo utilizando Kubernetes (k8s).

# Criando o primeiro Dockerfile

Uma regra básica na hora de criar seus Dockerfiles é considerar que suas instruções ao longo do arquivo representam camadas, e as camadas que exigem mais processamento e mais tempo para "construir" devem estar posicionadas no início. Dessa forma, você otimiza o tempo de processamento, e caso seja precise rebuildar a imagem com alguma modificação em camadas mais superiores, pois as que não sofrem modificação são recarregadas a partir de um cache.

No caso de Dockerfiles para aplicações web em python eu costumo seguir um pipeline básico do que é preciso/interessante ter na imagem, e ao longo do projeto vou modificando de acordo a necessidade:

![Diagram](/img/finance_app_tutorial/pt3/diagrams.png)

Ainda assim, é fácil encontrar modelos de Dockerfile para diversos tipos de aplicação no github, apenas busque entender o que de fato está acontecendo antes de copiar e colar.

```
FROM python:3.9.0-slim-buster

WORKDIR /usr/src/app

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

RUN apt-get update \
  && apt-get -y install netcat gcc \
  && apt-get clean

RUN pip install --upgrade pip \
  && pip install pipenv
COPY /Pipenv Pipfile.lock ./
RUN pipenv install --deploy --system

COPY . .
```

Como visto no fluxograma, primeiramente vemos uma imagem base do python, que aqui estamos utilizando o `python:3.9.0-slim-buster`, uma espécie de imagem mínima apenas com o essencial para podermos rodar uma aplicação python, além da slim-buster temos outras variações que buscam aplicar o mesmo conceito de economia de espaço. Este artigo do [Medium](https://medium.com/swlh/alpine-slim-stretch-buster-jessie-bullseye-bookworm-what-are-the-differences-in-docker-62171ed4531d) contém uma boa explicação sobre a diferença entre os tipos: Slim, Alpine, Strech, Buster, Jessie e Bullseye.

> Como são imagens mínimas e de tamanho reduzido, pode ser que a depender de sua aplicação, você precise completar a imagem com alguma dependência. É difícil saber de antemão qual a imagem que vai funcionar corretamente com seu projeto, mas graças a praticidade de trabalhar com Docker, podemos facilmente trocar e buscar a que melhor se adapte.

Voltando ao código que escrevemos, nas variáveis de ambiente (`ENV`), é uma boa prática ao trabalhar com projetos em python que adicionemos as seguintes variáveis:

- `PYTHONDONTWRITEBYTECODE`: Impede que o Python grave arquivos pyc no disco (equivalente a opção -B)
- `PYTHONUNBUFFERED`: Evita que o Python armazene um buffer stdout e stderr (equivalente a opção -u)

# Criando o primeiro docker-compose.yml

Antes de criarmos o docker-compose.yml, vamos adicionar um .dockerignore, uma boa prática para manter a segurança do nosso ambiente, já que estamos copiando arquivos locais para dentro das imagens, pode ser que em algum momento, por descuido, variáveis de ambiente vazem em produção gerando assim uma grande falha de segurança.

> Se quiser fazer um review de boas práticas de Docker para desenvolvedores python, [acesse](https://mherman.org/presentations/dockercon-2018/).

Adicione à sua pasta `project/` o arquivo `docker-compose.yml`.

```yaml
version: "3.8"

services:
  web:
    build: ./project
    command: uvicorn app.main:app --reload --workers 1 --host 0.0.0.0 --port 8000
    volumes:
      - ./project:/usr/src/app
    ports:
      - 8004:8000
    environment:
      - ENVIRONMENT=dev
      - TESTING=0
```

Se você nunca trabalhou com esse tipo de arquivo, não se assuste, é apenas uma linguagem de marcação com um conceito de maps e listas muito semelhante ao chave-valor de dicionários em python, tanto que tem um pacote chamado [PyYAML](https://pypi.org/project/PyYAML/) que mapeia o seu arquivo yml em um dicionário.

Com essas instruções nós construímos nossos serviços. O primeiro será nosso serviço web, onde foi especificado um diretório para `build` onde basicamente o Docker vai buscar o seu template (Dockerfile), depois a gente inicia o container rodando o `uvicorn` na porta 8000.

A marcação de `volumes` é extremamente interessante/importante para que tenhámos persistência de dados ao longo do desenvolvimento e em produção. Basicamente estou dizendo que o `./project` local vai ser mapeado no diretório `/usr/src/app`, e tudo que acontecer localmente também vai ser atualizado dentro do ambiente docker sem necessidade de rebuildar a imagem.

Depois disso nós expomos uma porta 8004 que vai escutar a porta 8000 dentro da nossa image Docker. E por fim adicionamos duas variáveis de ambiente, que são as mesmas que estão na nossa classe Settings no arquivo `config.py`.

Vamos buidar nossa imagem:

```bash
$ docker-compose build

```

> Eu tive um problema ao buildar a imagem pois o diretório root da imagem é diferente do que está meu projeto no pyCharm. Para resolver isso eu tive que alterar o main.py e mudar o que o pyCharm considera como root.

Modificando o `main.py`:

```python
# project/app/main.py

from fastapi import FastAPI, Depends

# modified
from app.config import Settings, get_settings

app = FastAPI()


@app.get("/ping")
async def ping(settings: Settings = Depends(get_settings)):
    return {
        "ping": "pong!",
        "environment": settings.environment,
        "testing": settings.testing
    }

```

Para configurar o pyCharm, clique com o botão direito em cima da pasta do project e vá em **Mark Directory as** > **Sources Root**. Agora rode a build novamente e provavelmente deve funcionar.

Assim que a build estiver pronta, vamos iniciar o container em background (detached `-d`):

```bash
$ docker-compose up -d

```

Vá até o link [http://localhost:8004/ping](http://localhost:8004/ping) e você verá o seguinte output:

```json
{
  "ping": "pong!",
  "environment": "dev",
  "testing": false
}
```

Em nosso diretório de projeto, você deverá ter uma estrutura semelhante a essa:

```bash
.
├── docker-compose.yml
├── project
│   ├── app
│   │   ├── config.py
│   │   ├── __init__.py
│   │   ├── main.py
│   ├── Dockerfile
│   ├── Pipfile
│   └── Pipfile.lock
└── README.md

```

# Próximo Capítulo...

Na próxima etapa, iremos adicionar um banco de dados para nossa aplicação, onde irei containerizar o PostgreSQL e adicionar um ORM (SqlAlchemy) para abstrair parte da complexidade de trabalhar diretamente com o banco.

Você pode acompanhar o repositório do projeto no link abaixo:

- [GitHub - Econowallet](https://github.com/LucianoBatista/econowallet)
