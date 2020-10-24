---
layout: post
title: "Pareando Dispositivo 2.4GHz com Modem Vivo Fibra Askey"
description: >
  Passos para configurar uma lâmpada RGB Wifi 2.4GHz com Modem da Vivo.
categories: smarthome
tags: smarthome
comments: true
image: /assets/img/blog/smarthome/modem-vivo-fibra.jpeg
---
> A maior parte dos dispositivos WiFi para casa inteligente operam na
frequência de 2.4GHz. Contudo, vários Modems não entregam essa frequência
diretamente. Este artigo oferece uma solução para este problema.
{:.lead}

- Table of Contents
{:toc}

## Contexto

Adquiri o Echo Dot há alguns meses atrás, no dia da chegada do dispositivo
na Amazon do Brasil. Desde então, tenho usado-o de forma bem básica - apenas
para lembretes e para ouvir música. Contudo, recentemente decidi criar um
projeto de casas inteligentes, para fazer melhor uso da Alexa e expandir
meu conhecimento neste assunto.

> Este é, portanto, o primeiro de vários posts que serão dedicados ao meu
> projeto de smart home na região de São Paulo, partindo do zero.

Não pude encontrar nenhuma loja que vendesse lâmpadas Smart na região do ABC
paulista. Nem Leroy Merlin, nem Fast Shop, nem Tok&Stok. Isso foi uma decepção.
Entendo que a melhor opção para aquisição de dispositivos smart aqui no ABC é,
de fato, a compra online.

### Breve Review: Lâmpada RSmart

Já possuindo um dispositivo Echo Dot, passei então para meu segundo dispositivo:
uma lâmpada. Adquiri a [Lâmpada Inteligente RSmart Wi-Fi LED 9W Branco Frio e Quente RGBW](https://www.amazon.com.br/gp/product/B08D6X38VV/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1)
na Amazon.

![](/assets/img/blog/smarthome/smarthome-lampada-rsmart.png)

Figure: Lâmpada Inteligente RSmart Wi-Fi.
{:.figcaption}

Esta lâmpada tinha bons reviews na Amazon. Trata-se de uma lâmpada RGBW, ou
seja, suporta várias cores mas também tem o branco - e tons de branco, como
aquele mais branco (ideal para estudo e trabalho) e aquele mais amarelado
(ideal para relaxar). Ela tem uma boa luminosidade, de 810 lúmens (número acima
de 800lm é sempre bom). A luminosidade cai bastante nas cores RGB, mas entendo
ser algo aceitável, já que não fazemos uso de cores de forma extensiva. O preço
também estava bom: perto de R$100,00 (a título de comparação, o modelo da
Philips custa por volta de R$300,00). E outra coisa que percebi, após a
instalação, foi que mesmo após corte de energia (seja por queda de fornecimento
ou por desligamento indevido do interruptor), a lâmpada não perdeu a
configuração.

A luminosidade do RGB é algo difícil de explicar. Eu li e achei estranho
que houvesse essa perda de luminosidade com luz colorida, e só fui entender
que não é de fato um problema ao testar eu mesmo, em casa.
{:.note}

Uma coisa que achei confuso é que, nas especificações da Amazon, a lâmpada diz
ser de 220 volts. Na verdade, ela é bivolt, como pude constatar pesquisando
na Internet e, posteriormente, testando em casa, na rede de 110v.

## A Instalação

Instalei a lâmpada no abajur da minha mesa de trabalho, onde está também o
Echo Dot. o roteador WiFi fica na sala, há alguns metros, mas o sinal é bom.
Entrei na Play Store e instalei o aplicativo da RSmart.

Deixei a lâmpada em modo de pareamento (é preciso ligar e desligar o
interruptor por três vezes, até que ela começa a piscar), e fui ao aplicativo
para parear. E foi aí que os problemas começaram.

## O Problema

Hoje, utilizo um modem da Vivo Fibra da Askey, que também tem função de
roteador e entrega duas frequências: 2.4GHz e 5GHz.

![](/assets/img/blog/smarthome/modem-vivo-fibra.jpeg)

Figure: Modem Vivo Fibra Askey com funçao de roteador.
{:.figcaption}

Troquei a minha rede WiFi no celular para a rede 2.4GHz e tentei parear com
a lâmpada - mas sem sucesso. Observei, então, que a rede WiFi no meu celular
continuava com 5GHz. Acessei o gateway do roteador e desliguei a rede 5GHz.
Ainda assim, permanecia conectado no celular em 5GHz. O que estava acontecendo?

> Neste modem, a opção de 5GHz é exclusiva para 5GHz, mas a opção de 2.4GHz
> assume as duas frequências (2.4GHz e 5GHz), e não parece ser possível
> alterar isto (para tornar a opção de 2.4GHz exclusiva para a frequência
> de 2.4GHz - o que seria o esperado). Ao pesquisar mais, entendi que isto
> é uma tendência em vários modems hoje, inclusive nos EUA. E no celular, o
> aparelho automaticamente escolhe a melhor frequência, sem dar opção de
> escolha (testado em Android 9 no Motorola MotoZ3 e no iPhone 8). Então, o
> 5GHz sempre era a frequência escolhida no celular.

Ao pesquisar soluções para isto, o que eu mais encontrei foi pessoas sugerindo
o seguinte:
- Saia da sua casa e se afaste até perder a rede
- Vá então voltando para a sua casa até pegar rede; nesta hora, a rede vai ser
de 2.4GHz, pois é uma rede de maior alcance que a 5GHz
- Faça o pareamento e só então volte pra casa

Impressionou-me saber que a solução mais sugerida (e a única até o momento,
inclusve nos EUA) para este problema fosse algo tão ridículo. Até cheguei a
tentar fazer, mas o
problema é que, no tempo que levava para sair de casa e pegar a rede 2.4GHz,
o dispositivo saía do modo de pareamento.

Então, o modem e o celular querem decidir por mim qual rede eu devo utilizar,
decidem que 5GHz é o melhor pra mim, e não me deixam opinar. E os
dispositivos Smart só fazem pareamento em 2.4GHz. São premissas contraditórias.

## A Solução: Dois Celulares e Gambiarra

Após algumas horas de pesquisa infrutífera e frustração, encontrei
[um video que explica uma técnica que resolveu meu problema](https://www.youtube.com/watch?v=iTaI4Aro0xc).
Contudo, é necessário que hajam dois celulares. A partir da ideia apresentada
no video, segue abaixo o procedimento que adotei, em ordem:

- No modem da Vivo:
  - acessar o gateway e desabilitar a opção 2.4GHz: a única opção disponível
  agora é de 5GHz (que entrega apenas frequência de 5GHz).
- No segundo celular:
  - manter WiFi habilitado, mas desconectar do roteador da Vivo. Deixar
  conectado na rede da operadora (4G).
  - alterar o nome da rede para ter o mesmo nome da rede (SSID)/senha do modem
  Vivo na opção 2.4GHz
  - criar um hostspot para permitir conexão do celular principal
- No celular principal:
  - conectar no hotspot do segundo celular. Por ele estar no 4G da operadora,
  vai compartilhar a rede na frequência de 2.4GHz.
  - realizar o pareamento com a lâmpada. Por estar na frequência de 2.4GHz,
  agora vai funcionar normalmente. A lâmpada está conectada agora via WiFi no
  segundo celular (que é quem distribui a rede).
- No modem da Vivo:
  - ligar novamente a opção 2.4GHz
- No segundo celular:
  - desligar o hotspot. Como a lâmpada já foi pareada com
  um SSID igual ao do modem da Vivo na opção 2.4GHz, a lâmpada vai trocar a rede
  (do segundo celular para o modem da Vivo) de forma transparente.

## Próximos Passos

Esta solução funciona, mais é ruim por depender de dois celulares. Ainda assim,
no meu caso, é melhor do que a ideia de ter que sair de casa para parear. De
qualquer forma, não está bom.

É possível utilizar o modem da Vivo Fibra com outro roteador. Desta forma,
pretendo encontrar e adquirir um roteador que tenha uma clara separação entre
as duas frequências. Com esta separação clara, o pareamento pode acontecer
naturalmente, sem necessidade de gambiarras.
