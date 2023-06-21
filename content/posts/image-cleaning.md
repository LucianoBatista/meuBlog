---
layout: post
title: "Computer Vision"
subtitle: "Como excluir artefatos indesejados numa imagem?"
date: 2022-01-26
author: "Luciano"
featuredImagePreview: "img/computer_vision.png"
tags:
  - Tutorial
  - Data Science
  - Visão Computacional
categories: [Tutorials]
---

O intuito desse post é compartilhar uma solução simples que resolvou um problema complexo que estávamos enfrentando em de nossos produtos aqui na [**Studos**](https://www.studos.com.br), a _Leitora de Gabaritos_. Dessa forma, outros que estejam com problemas similares possam talvez ter um ponto de vista diferente na resolução do problema.

## O que é a leitora de gabaritos?

Dando um pouco de contexto sobre o produto, a Leitora de Gabaritos é uma API desenvolvida utilizando (principalmente) a seguinte stack:

- FastAPI
- Celery
- ReportLab
- OpenCV
- RabbitMQ
- Redis
- Docker
- Kubernets

Essa API é responsável por automatizar a criação e a leitura de cartões respostas. Utilizando ativamente o OpenCV nas etapas de leitura.

Algumas features do ponto de vista técnico da leitora, envolvem:

- Extração do qrcode da imagem
- Rotação de imagens em diversas orientações
- Identificação de shapes
- Recorte
- Extração de segmentos de leitura
- Extração das marcações

Esses segmentos, no nosso caso, são as regiões: das marcações dos itens, do código de matrícula, das questões optativas e do modelo de prova. As regiões amarelas da imagem abaixo representa melhor os _segmentos_.

![leitora_gabarito](/img/leitora_gabarito/areas-of-reading.png)

### Problema

O problema que estávamos enfrentando era justamente na leitura desses segmentos. Mais especificamente, na etapa _extração das marcações_, onde é necessário que todas as marcações sejam extraídas sem trazer nenhum resíduo do gabarito.

É nesse ponto que nos deparamos com a complexidade do problema, pois a quantidade de variáveis envolvidas no processo é absurda, e nem sempre conseguíamos fazer a melhor separação possível, impactando na ponta e gerando insatisfação do usuário. Essa **variabilidade de imagens**, conta com:

> ...imagens de qualquer faixa de DPI, imagens coloridas, escala de cinza e preto e branco, canetas pretas e azuis, pouca luz, sombra e ruídos ao longo da imagem, imagens tiradas com foto, imagens digitalizadas com diferentes scanners...

O ponto específico onde o OpenCV entra para realizar a transformação que precisamos é, mostrado abaixo no código:

```python
img_black_and_white = cv.threshold(
    img_warp_gray, thresh_param, 255, cv.THRESH_BINARY)[1]
img_threshold = cv.threshold(img_black_and_white, 215, 350, cv.THRESH_BINARY_INV)[1]
```

Veja que, como input para a função `cv.threshold`, precisamos fornecer uma **imagem em escala de cinza** e um **parâmetro de threshold** que será utilizado como um ponto de corte. É justamente esse threshold que precisava ser otimizado.

Antes dessa solução, realizávamos dois checks: _a imagem é colorida, ou escala de cinza?_ e _a imagem tem caneta azul ou preta?_. Com base nos outputs, ajustávamos o threshold de acordo.

Porém, tal ajuste não funcionava 100% das vezes, e algumas imagens estavam com a leitura impossibilitada justamente por não conseguir ajustar o threshold. Veja abaixo um caso onde o ajuste errado do threshold leva a uma leitura incorreta.

![old-technique](/img/leitora_gabarito/old-transformation.png)

Com a aplicação desse threshold, poucas marcações seriam de fato lidas pela aplicação.

## Solução

Sendo assim, tivemos a ideia de buscar por um padrão entre as diferentes imagens que populavam nosso banco. Coletamos 200 imagens das mais diversas possível e começamos olhando para a distribuição de pixels. Nosso objetivo nesse primeiro momento era conseguir definir uma hipótese mais acertiva de como abordar o problema de forma mais generalizável possível.

![pixels](/img/leitora_gabarito/pixels-distributions.png)

> Essa é a distribuição dos pixels para o segmento do cartão apresentado acima. Quanto mais valores em direção ao 255, mais marcado está esse cartão. Valores tendendo ao 0, estão relacionados ao preto.

### Distribuição de pixels

Ao analisar a distribuição de pixels, vimos que as imagens similares possuem distribuições de pixels bem características. E que todas possuem uma calda longa a esquerda com um ponto onde a distribuição começa a "subir" (aumento no valor de pixel).

![pixels](/img/leitora_gabarito/distributions.png)

1. Imagem digitalizada em preto e branco
2. Imagem em escala de cinza
3. Imagem colorida de alto dpi
4. Imagem colorida de baixo dpi

Esse ponto de inflexão da curva nos chamou a atenção, por ser uma região que estava próxima do threshold que já vínhamos utilizando. Verificamos visualmente para todas as imagens e o padrão se repetiu.

Dae em diante conseguimos definir melhor nossa abordagem e o que de fato gostaríamos de coletar das distribuições: **"Como identificar automaticamente esse ponto de inflexão da distribuição de pixel, para qualquer distribuição?"**

### Método utilizado

Antes de chegar na técnica que resolveu o problema, algumas outras foram testadas, algumas mais complexas e outras mais simples, mas nenhuma generalizou do jeito que precisávamos.

Nossa hipótese aqui foi a seguinte:

> Considerando que a maior parte da distribuição, com pixels maiores, correspondem as marcações, a zona mais escura (região da calda longa da distribuição) corresponde ao gabarito. Dessa forma, se considerarmos uma porcentagem e eliminarmos parte da calda, seria possível saber onde está o ponto ótimo de corte.

Para melhor visualizar essa abordagem, criamos um [**ECDF**](https://en.wikipedia.org/wiki/Empirical_distribution_function) das distribuições dos pixels e traçamos uma linha horizontal na porcentagem de pontos que gostaríamos de remover da distribuição.

![ecdf](/img/leitora_gabarito/ecdf.png)

Dae em diante, para testar se realmente esse método funcionava, o que precisávamos fazer era encontrar os pontos de intersercção da reta na horizontal com a distribuição, coletar o máximo e retornar como ponto de corte. A implementação está no código abaixo.

```python
def get_param(img_warp_gray):
    # x and y for the ecdf plot
    x = np.sort(img_warp_gray.ravel())
    y = np.arange(1, len(x) + 1) / len(x)

    # ecdf dataframe
    ecdf_plot = pd.DataFrame({"img": x, "prop": y})

    # get intersection values on 5% of the distribution
    thresh_params = ecdf_plot[round(ecdf_plot["prop"], 2) == 0.05]["img"].values
    thresh_params_max = max(thresh_params)
    return thresh_params_max

# running
def main():
    img = cv.imread(f"src/files/snippets/pic_7.png")
    img_warp_gray = cv.cvtColor(img, cv.COLOR_BGR2GRAY)
    try:
        thresh_param = get_param(img_warp_gray)
        img_threshold = image_transformation(thresh_param, img)
    except ValueError:
        thresh_param = 0
        img_threshold = image_transformation(thresh_param, img)

    # plotting the ecdf, 0.05 horizontal line, image before and image after the
    # transformation
    plt.subplot(3, 1, 1)
    _ = sns.ecdfplot(x=img_warp_gray.ravel())
    plt.hlines(xmin=0, xmax=255, y=0.05)

    plt.subplot(3, 1, 2)
    plt.imshow(img_warp_gray, cmap="gray")

    plt.subplot(3, 1, 3)
    plt.imshow(img_threshold, cmap="gray")

    plt.show()

if __name__ == "__main__":
    main()

```

Espantosamente, nessa linha de 5%, foi justamente a região de inflexão da distribuição. E aplicando o parâmetro que a função `get_param()` retornou, o resultado da transformação foi o seguinte:

![best-transform](/img/leitora_gabarito/best-transformation.png)

Após encontrar esse pontos, rodamos o script para as 200 imagens que estávamos utilizando como teste e o resulado foi **100% de remoção do gabarito das marcações.**

Dessa forma, agora conseguimos identificar as marcações, em qualquer tipo de imagem, qualidade e cor de caneta marcada, sem propagar para as próximas etapas de leitura os ruídos do cartão.

Caso queira conferir o código e as imagens testes, elas estão nesse [repositório](https://github.com/LucianoBatista/auto-threshold-adjustment).
