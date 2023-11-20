---
layout: post
title: "Autoencoders no Pytorch"
subtitle: "Chapter 03 - Variational Autoencoders"
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
draft: true
---

O Capítulo 3 é sobre Variational Autoencoders, mas vou quebrar aqui primeiramente sobre autoencoders e depois falamos sobre os Variational Autoencoders.

## Autoencoders

Metáforma do guarda-roupa mágico

O dataset que estamos utilizando (Mnist fashion)

### Arquitetura simples dos autoencoders

Autoencoders as denoising models

### Encoders

Sequência de CNNs para criar uma representação das imagens

### Decoders

Recebe as representações realizadas pela etapa de encoding e reconstrói a imagem utilizando o conceito da transposta da camada convolucional.

### Juntando tudo, escolhendo nossa loss function

Aqui a gente finaliza a arquitetura

### Reconstruindo imagens

Todo o ápice do artigo é chegar aqui pra gente reconstruir imagens.

### Visualizando o espaço latente

Nosso espaço latente

### Gerando novas imagens

Quais são os problemas dos Autoencoders? Por que partimos para utilização dos Variational Autoencoders.
