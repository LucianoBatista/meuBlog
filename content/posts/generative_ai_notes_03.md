---
layout: post
title: "Intro ao Pytorch"
subtitle: "Chapter 02 - Generative Deep Learning Book"
date: 2023-07-06
author: "Luciano"
authorLink: "https://www.linkedin.com/in/lucianobatistads/"
featuredImage: "img/intro-torch.png"
math:
  enable: true
tags:
  - AI
  - Generative
  - IA Generativa
categories: [Tutorials]
draft: false
---

Nesse artigo, vamos de mão na massa! Mas gostaria de fazer um disclaimer um pouco chato pra você, **vamos ver tudo de forma superficial**, cada tópico abordado aqui por si só precisaria de muitas páginas de explicação, então, vou fazer o melhor para a explicação não se tornar um Frankstein e o post virar uma _cocha de retalhos_.

Meu papel aqui é trazer de forma objetiva cada tópico desse para que você consiga correlacionar depois com o avançar dos capítulos do livro que estou trazendo os reviews.

E se você chegou apenas para esse artigo e não sabe do contexto, eu na verdade estou trazendo uma série de blog posts sobre minhas anotações sobre o livro **Generative Deep Learning**. E, já rolaram dois posts até então:

- [parte 1](https://www.lobdata.com.br/posts/my_notes/)
- [parte 2](https://www.lobdata.com.br/posts/generative_ai_notes_02/)

Você pode acompanhar na lateral (:point_right:) o TOC do post e pular para parte que mais te interessa :wink:. Vamos lá!!!

## Tudo começa com tensores...

De forma simples, tensores são uma forma 'fancy' de se representar arrays multidimensionais.

Você muito provavelmente já está acostumado a trabalhar com `numpy` arrays, porém apesar de terem um comportamento parecido, as implementações de tensores te fornecem não só uma série de outras operações matemáticas e otimizações, mas também a capacidade de rodar tudo isso em gpus ou tpus, o que basicamente torna o avanço de Deep Learning possível (do ponto de vista de força bruta para treinar os modelos).

{{< admonition type=tip title="Nota" open=true >}}
Vale falar que os tensores não são exclusividade do `Pytorch` ta? O **TENSOR**Flow leva inclusive o termo escrito bem no nome do framework.
{{< /admonition >}}

Definir um tensor no `Pytorch` é bem simples:

```python
x = torch.tensor([1, 2, 3], dtype=torch.Float16)
```

Devido a capacidade de rodar em diferentes dispositivos, como gpus, cpus e tpus, a biblioteca do `Pytorch` te possibilita migrar esse dado entre dispositivos, te dando total liberdade de como você vai executar o treinamento. Liberdade essa que se estende para multi-gpus, multi-tpus...

```python
# simples assim você leva um tensor para diferentes dispositivos
x.to("cuda")
```

Para enviar e armazenar esse tensor na memória da GPU, utilizamos o nome `cuda`. E, para CPU... É cpu mesmo.

O nome `cuda`, para quem nunca ouviu falar, vem de _Compute Unified Device Architecture_ e foi desenvolvido pela NVidia para permitir as implementações de processamento paralelo utilizando placas de vídeo.

Provavelmente por uma decisão de projeto, o nome `cuda` permanece sendo utilizado no `Pytorch` até hoje, mesmo não sendo o mais intuitivo (pelo menos na minha humilde opnião), mas, bibliotecas mais recentes e que rodam sobre o `Pytorch`, como `pytorch-lightining` utiliza `gpu` para indicar que seu treinamento vai ser executado na GPU. Nada mais intuitivo, concorda?

Com grandes poderes, vem grandes responsabilidades!! E a configuração do `device` hoje gera alguns dos erros mais comuns quando estamos trabalhando com o `Pytorch`.

Como é bastante comum estarmos rodando o treinamento utilizando GPUs, para que tudo funcione, todos objetos que você trabalha precisam estar alocados na memória de apenas um dispositivo.

- modelo
- input
- labels
- pesos
- etc

Sendo nós os responsáveis por levar esse dado para o lugar certo, muitas vezes acabamos esquecendo de fazer isso, e o resultado é uma bela mensagem de erro.

{{< admonition type=danger title="Perigo" open=true >}}
Tente executar o código abaixo, para você também entrar para a estatística:
{{< /admonition>}}

```python
x = torch.tensor([1, 2, 3]).to('cuda')
y = torch.tensor([2, 3, 4]).to('cpu')

print(x + y)
```

Gostaria de fazer uma menção honrosa aqui a uma função que eu de fato não sei se está sendo muito utilizada por quem atua diretamente com o desenvolvimento de arquiteturas de Deep Learning, mas que quando eu vi eu achei super interessante, que são os chamados `named_tensors`.

Basicamente você pode adicionar um nome para as dimensões dos tensores que você está trabalhando, o que possibilita um debug mais fácil quando algum problema acontece, e também algumas operações podem ser feitas por esses nomes. Vou deixar ao fim do post um link para documentação com uma explicação mais profunda e com alguns exemplos.

Temos também uma pohaaada de operações que são possíveis de realizar com tensores, mas devido nossa abordagem aqui, vamose vê-las a medida que fomos utilizando.

## Por debaixo do capô...

A esse ponto, eu queria que você tivesse um modelo mental de que um treinamento de uma rede neural é um encadeamento de operações, arranjadas de certa forma que nós conseguimos atualizar pesos e parâmetros que irão no final cuminar em um modelo.

Esse conjunto de operações, e as formas como eles se conectam, são criados no momento de execução e "armazenados" pelo `Pytorch`. Essa organização é feita em grafos que podem ser representados como na imagem abaixo.

![grafo](https://i.imgur.com/JL2RSfo.png)

O Autograd é um módulo do `Pytorch` que permite o cálculo do gradiente, de forma performática e de forma completamente abstraída para nós, usuários do framework.

Você pode, também ir bem deep no entendimento dos detalhes internos do `Pytorch`, deixarei um post no fim do artigo para isso.

### Diferenção automática

Blz!! Temos várias operações para realizar, e consequentemente várias derivadas para calcular, será que precisamos fazer isso na mão??

De forma alguma... É então que a diferenciação automática entra em nossas vidas, essa feature do `Pytorch` permite que o framework consiga calcular o gradiente ao longo de toda a cadeia de operações realizadas pela sua rede neural, em relação a variáveis que você indica pra ele. Similar à imagem abaixo:

![](https://i.imgur.com/77Em0MV.png)

Essa indicação das variáveis que serão consideradas na hora do cálculo do gradiente é feita pelo parâmetro `require_grad=True`. Dessa forma o `Pytorch` vai armazenar o valor do gradiente em uma propriedade chamada `.grad`.

### Optimizers

No artigo passado nós falamos sobre minimização, esse processo que acaba sendo chamado de otimização. Justamente por esse motivo, o `Pytorch` criou uma abstração chamada, adivinha o nome?, `optimizers`. Nesse módulo você vai encontrar diversos métodos de otimização, entre eles, um dos mais comuns, chamado de Stochastic Gradient Descent (SGB).

A imagem abaixo mostra como o processo de otimização acontece:

![](https://i.imgur.com/dCqfggz.png)

Então, relembrando, nós temos uma loss, nós precisamos minimizar, esse processo se chama otimização, e minimizar essa função implica que os pesos ao longo da arquitetura da rede neural sejam atualizados.

Em código, veja abaixo como esses passos se desenrolam:

```python
# learning rate
eta = 0.1

# variável que o modelo vai considerar na hora de minimizar a função
x_param = torch.nn.Parameter(torch.tensor([-3.5]), requires_grad=True)

# escolha do optimizer a ser utilizado
optimizer = torch.optim.SGD([x_param], lr=eta)

# as épocas são como nós nomeamos as iterações
for epoch in range(200):
    # como a cada iteração o torch mantém os valores antigos do gradiente
    # o zero_grad() é justamente para zerar esse dados
    optimizer.zero_grad()
    loss_incurred = f(x_param)

    # fazemos o cálculo
    loss_incurred.backward()

    # atualizamos os pesos para próxima iteração
    optimizer.step()

print(x_param.data)
```

```
tensor([2.0000])
```

Nosso resultado aqui é o mesmo do mostrado no artigo passado, só que dessa vez nós realizamos o processo de forma iterativa. Guarda esse processo, mais abaixo nós também vamos utilizá-lo para o treinamento da nossa **primeira rede neural**.

## Primeira Rede Neural

Um insight muito massa que eu tive ao ler o livro `Inside Deep Learning`, é que o `Pytorch` foi construído com uma premissa bem forte de que todo treinamento de uma rede neural é na verdade um problema de otimização.

Então, independentemente do problema que estamos atacando (classificação, regressão...), temos que pensar o problema como um problema de otimização. E de fato isso faz muito sentido, dado que todos os pesos dos modelos são alterados com base no processo de minimização de uma função, a loss.

No processo de treinamento de uma rede neural fica então evidenciado um padrão. Nós teremos sempre `dados` que irão alimentar o `modelo`, teremos o modelo (nossa arquitetura) e a `loss` que vai alterar a depender do tipo de task que estaremos atacando. A seguinte imagem traduz muito bem o processo:

![](https://i.imgur.com/cftsFDk.png)

Vamos então codar pedacinho desse e ver como desenrola na prática!!

### Training Loop

Vimos acima, com um exemplo mais simples, que esse é um processo iterativo. A implementação que você vê abaixo, é uma adaptação do anterior para contemplar uma situação real de treinamento de uma rede neural.

```python
# apenas para ter um typehint
Loss = Callable[[torch.Tensor, torch.Tensor], torch.Tensor]

def train_simple_network(
    model: nn.Module,
    loss_func: Loss,
    training_loader: DataLoader,
    epochs: int = 100,
    device: str = "cuda",
) -> None:
    # 1
    optimizer = torch.optim.SGD(model.parameters(), lr=0.001)
    model.to(device)

    for epoch in tqdm(range(epochs), desc="Epochs"):
        # 2
        model = model.train()
        running_loss = 0.0

        for input, labels in tqdm(training_loader, desc="Training"):
            # 3
            input = input.to(device)
            labels = labels.to(device)

            # 4
            optimizer.zero_grad()

            # 5
            output = model(input)
            loss = loss_func(output, labels)

            # 6
            loss.backward()
            optimizer.step()

            running_loss += loss.item()

```

Nesse fluxo o que está acontecendo é o seguinte:

1. Iniciamos o optimizer e enviamos o modelo para o device correto
2. Colocamos o modelo em modo de treino, indicando para o `Pytorch` que eu quero atualizar os pesos
3. Colocamos os dados para o device correto
4. Muito importante, zeramos o gradiente
5. Fazemos o "predict" e avaliamos o quão distante estamos do valor real, utilizando a loss para isso
6. Calculamos o gradiente e enfim atualizamos os pesos

### Data

Como vamos treinar para uma task de regressão, vamo gerar aqui alguns dados sintéticos com auxílio do `numpy` e vamos também visualizar o resultado.

```python
X = np.linspace(0, 20, num=200)
y = X + np.sin(X) * 2 + np.random.normal(size=X.shape)
sns.scatterplot(x=X, y=y)
```

![](https://i.imgur.com/eSUJdmr.png)

Como foi dito no último artigo, o `Pytorch` trabalha com duas abstrações chamadas de `Dataset` e `DataLoader`. Elas são responsáveis por alimentar seu treinamento com os dados, fazendo isso de forma bem performática.

As imagens abaixo ilustram muito bem o papel de cada um:

![](https://i.imgur.com/KTKptDw.png)
![](https://i.imgur.com/dhd1XJy.png)

Na primeira, o que a gente vê é o `Dataset` sendo o responsável por ir no nosso dado e selecionar um item. Por isso, dois métodos são obrigatórios quando estamos implementando o `Dataset`:

- `__len__`: vai nos dizer o tamanho do dataset
- `__getitem__`: vai coletar um item do dataset

Na segunda imagem, você vê a atuação do DataLoader, que tem o objetivo de pedir ao `Dataset` por específicos items. Como nós, durante o treinamento, passamos os dados em lote e embaralhados, os índices que estão sendo pedidos ao `Dataset` acabam não tendo uma ordem.

Do ponto de vista de implementação, basicamente o método `__getitem__` **precisa retornar uma tupla com o item + label**, seja os tensores das imagens, de texto, som... E o seu trabalho é basicamente adaptar o dado bruto para essa estrutura.

Em alguns casos, o `Pytorch` facilita esse trabalho e nós não precisamos codar uma classe `Dataset` customizada, como por exemplo quando trabalhamos com imagens. Veremos mais detalhes sobre, em próximos artigos.

Certo, eis aqui nosso `Dataset` e `DataLoader`:

```python
class SimpleRegressionDataset(Dataset):
    def __init__(self, X: torch.Tensor, y: torch.Tensor) -> None:
        super().__init__()
        self.X = X.reshape(-1, 1)
        self.y = y.reshape(-1, 1)

    def __len__(self) -> int:
        return self.X.shape[0]

    def __getitem__(self, idx: int) -> tuple[torch.Tensor, torch.Tensor]:
        X = torch.tensor(self.X[idx, :], dtype=torch.float32)
        y = torch.tensor(self.y[idx, :], dtype=torch.float32)
        return X, y

training_dataset = SimpleRegressionDataset(X, y)
training_loader = DataLoader(training_dataset, shuffle=True)
```

### Model e Loss

Basicamente você pode criar uma arquitetura (modelo) de Deep Learning de duas formas. Respeitando a orientação a objeto ou pelo paradigma funcional.

Por OOP nós criamos uma classe e herdamos do `Pytorch` a classe `Module` e obrigatoriamente precisamos implementar o método `forward`. Vamos simplificar aqui e criar nosso modelo utilizando o paradigma funcional, que ficaria assim:

```python
simple_model = nn.Sequential(nn.Linear(1, 10), nn.Linear(10, 1))

```

Pronto, temos nosso primeiro modelo :tada:, que de forma visual, seria algo como na seguinte imagem, lendo debaixo para cima:

![](https://i.imgur.com/ZivUyKV.png)

E então, nossa loss aqui vai ser a MSE (Mean Squared Error):

```python
loss_func = nn.MSELoss()
```

### Juntando o quebra-cabeça :sparkles:

Just run...

```python
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
train_simple_network(model, loss_func, training_loader, device=device, epochs=1000)
```

{{< admonition type=tip title="Nota" open=true >}}
É comum você encontrar esse condicional para buscar por `cuda` caso ela esteja disponível no seu computador, caso contrário use `cpu`. Caso esteja confortável em sempre utilizar a GPU, pode remover e apenas deixar `cuda`.
{{< /admonition>}}

Avaliando nossos resultados:

```python
with torch.inference_mode():
    Y_pred = (
        model(torch.tensor(X.reshape(-1, 1), device=device, dtype=torch.float32))
        .cpu()
        .numpy()
    )

sns.scatterplot(x=X, y=y)
sns.lineplot(x=X, y=Y_pred.ravel(), color="red")
```

![](https://i.imgur.com/fuKABqV.png)

A linha reta em vermelho aqui são nossas previsões. Mas, por que será que o modelo não conseguiu capturar a não lineariedade dos dados?

Isso acontece basicamente por que estamos concatenando operações lineares uma atrás da outra. E nesse caso, no final, se você utilizasse 1000 camadas no `nn.Sequential` esse modelo não conseguiria capturar esse perfil não linear dos dados.

### Funções de ativação ao resgate

Para resolver esse problema, nós adicionamos uma perturbação nas camadas internas da rede neural, que auxiliam o modelo a representar não lineariedades.

Vou deixar uma imagem aqui com algumas funções de ativação, e em seguimos vamos reimplementar o código, usando a `Tanh()`.

![](https://i.imgur.com/DRxjyPv.png)

```python
model = nn.Sequential(nn.Linear(1, 10), nn.Tanh(), nn.Linear(10, 1))
```

E então, rodamos novamente o treinamento:

```python

train_simple_network(model, loss_func, training_loader, device=device, epochs=1000)
```

Agora, como vemos na figura, foi possível capturar o formato não linear dos dados.

![](https://i.imgur.com/Sd3H5sc.png)

Show, para esse artigo era isso, espero que tenha conseguido deixar um pouco mais claro quais são as principais peças na hora de montar esse puzzle do Deep Learning.

Agora, nós iremos começar a entrar mais nas particularidades das diferentes arquiteturas, começando por CNNs, até lá!

## Links úteis

- Books: [Inside Deep Learning, Generative Deep Learning, Deep Learning with Pytorch]
- Repositório: [link](https://github.com/LucianoBatista/generative-ai)
- Named Tensors doc: [link](https://pytorch.org/docs/stable/named_tensor.html)
- Pytorch Internals: [link](https://pytorch.org/blog/computational-graphs-constructed-in-pytorch/)
