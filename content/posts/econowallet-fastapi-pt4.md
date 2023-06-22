---
layout: post
title: "Criando uma API Pronta para Produção com FastAPI - PT.4"
date: 2021-07-31
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

## Nos Capítulos Anteriores ...

Após um tempinho sem postar nada, vamos dar continuidade a essa série sobre como desenvolver uma API pronta pro Deploy! 🚀 🚀

Essa é a parte 4 do nosso projeto do **EconoWallet** e se você quiser verificar o que já fizemos até o momento, acesse os links abaixo:

- [Parte 1](https://www.lobdata.com.br/2021/03/30/criando-uma-api-pronta-para-produ%C3%A7%C3%A3o-com-fastapi-pt.1/)
- [Parte 2](https://www.lobdata.com.br/2021/04/03/criando-uma-api-pronta-para-produ%C3%A7%C3%A3o-com-fastapi-pt.2/)
- [Parte 3](https://www.lobdata.com.br/2021/04/08/criando-uma-api-pronta-para-produ%C3%A7%C3%A3o-com-fastapi-pt.3/)
- [Parte 4](https://www.lobdata.com.br/2021/07/31/criando-uma-api-pronta-para-produ%C3%A7%C3%A3o-com-fastapi-pt.4/)
- [Parte 5](https://www.lobdata.com.br/2021/08/01/criando-uma-api-pronta-para-produ%C3%A7%C3%A3o-com-fastapi-pt.5/)

Dando continuidade a nossa aplicação, hoje vamos configurar qual será nosso banco de dados. Apesar de ter tido que nesse post iríamos também configurar o **SQLAlchemy**, prefirir deixar para o próximo post, pois iria ficar muito conteúdo nesse artigo.

![I'm ready when you are](/img/finance_app_tutorial/pt4/areYouReady.gif)

## Modificações

Eu fiz algumas modificações de localização de arquivos de uma forma que fica simples a separação de competências do nosso app. Basicamente agora temos tudo que diz respeito às configurações do Docker (`Dockerfile`, `docker-compose`, `.dockerignore`) fora do diretório do projeto. Veja o resultado:

```bash
.
├── docker-compose.yml
├── Dockerfile
├── Pipfile
├── Pipfile.lock
├── project
│   └── app
│       ├── config.py
│       ├── __init__.py
│       └── main.py
└── README.md
```

E além de alterar a localização dos arquivos, foi preciso fazer um pequeno ajuste no `docker-compose.yml` para referenciar o Dockerfile no root do projeto.

```yaml
# modified
version: "3.8"

services:
  web:
    build: .
    command: uvicorn app.main:app --reload --workers 1 --host 0.0.0.0 --port 8000
    volumes:
      - ./project:/usr/src/app
    ports:
      - 8004:8000
    environment:
      - ENVIRONMENT=dev
      - TESTING=0
```

Agora provavelmente tudo deve estar funcionando normalmente. Build novamente e tente acessar o [swager da API](http://127.0.0.1:8004/docs/) (`docker-compose up --build -d`).

## Database

Nossa intenção durante o desenvolvimento é buscar ao máximo que nosso ambiente se assemelhe ao ambiente de produção, onde nossa API vai estar sendo acessada por usuários reais. Nesse cenário, as informações trafegadas precisam ser persistidas em algum lugar, e normalmente é nesse ponto onde o banco de dados se encaixa.

Saiba que existem diferentes banco de dados para diferentes tipos de problemas, para nossa aplicação iremos utilizar o **MySQL**.

### Configurando o MySQL

É realmente muito simples configurar suporte à uma base de dados à sua aplicação utilizando o **docker-compose**. Tudo que você precisa fazer é adicionar mais um _service_, veja:

```yaml
version: "3.8"

services:
  web:
    build: .
    command: uvicorn app.main:app --reload --workers 1 --host 0.0.0.0 --port 8000
    volumes:
      - ./project:/usr/src/app
    ports:
      - 8004:8000
    environment:
      - ENVIRONMENT=dev
      - TESTING=0
    # new
    depends_on:
      - db
  # new
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

O que está acontecendo aqui é bem semelhante ao serviço `web`, com algumas ressalvas:

- `depends_on`: aqui é utilizado pois agora o nosso serviço `web` depende que o `db` esteja pronto primeiro, antes de iniciar.
- estamos partindo de uma imagem base [oficial](https://hub.docker.com/_/mysql) do MySQL.
- `command` e `restart` são instruções oficiais da imagem do MySQL, doc acima 👆.

Além disso, agora precisamos especificar algumas variáveis de ambiente que serão utilizadas para criar nossa base de dados para o **EconoWallet** e a porta que iremos nos conectar para acessar as informações. E, como para nosso banco de dados estamos especificando um volume que não é um diretório, o mesmo precisa ser indicado ao final do `docker-compose.yml`.

### Conectando ao MySQL

Agora que nosso `docker-compose.yml` está configurado, podemos rebuildar nossa aplicação, eu gosto de _derrubar_ o serviço removendo os volumes sempre que tem uma modificação relativamente grande, e em seguida _subir_ novamente.

```bash
docker-compose down -v
docker-compose up --build -d
```

Você pode também consultar os logs para saber se tudo correu bem, basta rodar `docker-compose logs -f`. Se tudo correu bem você já está apto a conectar no nosso banco de dados.

Se você tem uma versão premium do PyCharm é bem simples de fazer isso, mas aqui eu mostrarei como configurar pelo **DBeaver** um software gratuito e que todos tem acesso.

![DBeaver](/img/finance_app_tutorial/pt4/dbeaver.png)

Aqui tem o [link oficial](https://dbeaver.io) do site deles, a instalação é super direta para qualquer SO.

#### Passo a Passo

1. Com o software instalado, vá na lateral superior esquerda e clique no símbolo de uma tomada:

![DBeaver](/img/finance_app_tutorial/pt4/dbeaver-1.png)

2. Agora você vai precisar preencher os campos as informações que estão no `docker-compose.yml` e clicar em **Test connection**.
   ![DBeaver](/img/finance_app_tutorial/pt4/dbeaver-2.png)

3. Provavelmente ele vai solicitar instalar alguns drives e se tudo correr bem verá uma mensagem como no print.
   ![DBeaver](/img/finance_app_tutorial/pt4/dbeaver-3.png)

4. Clique em finish e na lateral agora você deve estar vendo nosso banco já configurado!! ✨
   ![DBeaver](/img/finance_app_tutorial/pt4/dbeaver-4.png)

Apesar de estarmos com nosso banco configurado, ainda não temos nenhuma tabela para interagir e também não temos nenhuma interação entre a API e o banco. **SQLAlchemy ao resgate!!** 🩺

## SQLAlchemy

![SQLAlchemy](/img/finance_app_tutorial/pt4/sqlalchemy.png)

**SQLAlchemy** faz parte de um conjunto de frameworkers que são utilizados para abstrair a complexidade de lidar com o banco de dados diretamente. Desenvolver com um ORM não te prende à um banco de dados específico, onde posteriormente é possível plugar a aplicação ao Postgres, MySQL, Oracle, MariaDB ou outros (contanto que o framework tenha suporte). Escolhi o **SQLAlchemy** por alguns motivos:

- pela robustes: o SQLAlchemy possui diferentes módulos (Core e ORM) que te permite fazer muita coisa legal.
- muito tempo presente na comunidade: o que torna mais fácil lidar com problemas e bugs caso eles apareçam (e irão aparecer =D).
- ter sido bastante utilizado e testado em produção: isso atribui maior confiabilidade ao software.

Saiba que os desenvolvedores do `SQLAlchemy` estão trabalhando numa nova versão da lib e parte da sintaxe de como realizar queries com o ORM irá mudar. Apesar dessa nova sintaxe estar disponível na versão atual da lib ela ainda não está oficialmente lançada, para mais detalhes acesse a [documentação](https://www.sqlalchemy.org). Aqui iremos manter a sintaxe mais tradicional.

### Como funciona o SQLAlchemy OMR?

É importante deixar claro que aqui nós estamos utilizando o componente **ORM** do SQLAlchemy e não o **Core**. A diferença é que o ORM adiciona uma camada de abstração que torna a interação com o banco mais _pythonica_ e menos _SQL raiz_.

Minha intenção não é explicar todos os detalhes do SQLAlchemy, mas sim o de ajudar a compreender os componentes que iremos utilizar aqui. Existe um livro muito bom com várias informações super relevantes chamado [Essential SQLAlchemy](https://www.amazon.com.br/Essential-SQLAlchemy-Rick-Copeland/dp/0596516142).

Dito isso, eu costumo pensar o SQLAlchemy como duas sessões diferentes:

1. A primeira onde especificamos quais os **models** (traduzidos em tabelas do banco) da nossa aplicação.
1. E o segundo onde especificamos a **session** (conexão com o banco).

No diagrama abaixo eu tento ilustrar esses dois cenários:

1. **Cenário Um**

![SQLAlchemy Explain](/img/finance_app_tutorial/pt4/sqlal-1.png)

Veja que partimos de uma classe mãe chamada de `declarative_base` pelo SQLAlchemy e que a mesma é utilizada na criação de cada um dos models. Além disso, após configurados os models, nós utilizamos o `.metadata.create_all(engine)` para de fato materializar esses models em tabelas no banco. Essa **engine** é justamente onde está a informação de qual banco de dados utilizar.

1. **Cenário Dois**

![SQLAlchemy Explain](/img/finance_app_tutorial/pt4/sqlal-2.png)

Aqui nós vemos como é feita a conexão com o banco e é por meio da session que faremos transações com as nossas tabelas criadas, o famoso **C**reate/**R**ead/**U**pdate/**D**elete. Um ponto para ficar atento é que ao configurar as sessions na nossa aplicação precisamos cuidar para que as sessions sejam sempre finalizadas, mesmo quando algum problema aconteça.

## Próximo Capítulo... 🎉

Na próxima etapa, iremos configurar o **SQLAlchemy** e fazer com que nosso app comece a interagir com o banco!!

Você pode acompanhar o repositório do projeto no link abaixo:

- [GitHub - Econowallet](https://github.com/LucianoBatista/econowallet) 🥂
