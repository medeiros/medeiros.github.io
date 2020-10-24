---
layout: post
title: "Melhorando a rede WiFi pela separação de Modem e Roteador"
description: >
  Abordagem para otimizar custo e performance na montagem de uma rede
  doméstica para SmartHome.
categories: smarthome
tags: smarthome
comments: true
image: /assets/img/blog/smarthome/smarthome-bridge-modo4.png
---
> A maioria de nós utiliza o modem entregue pela operadora do jeito é entregue,
e não faz nada além de configurar a senha do WiFi. Contudo, é possível melhorar
a performance e reduzir o custo pela separação de modem e roteador. Este artigo
fala um pouco sobre este tema.
{:.lead}

- Table of Contents
{:toc}

## O mito do baixo limite de dispositivos conectados em WiFi

Existe um mito para SmartHomes que afirma que rede WiFi não suporta mais do
que 20 dispositivos, e que, por conta disto, é melhor fazer uso de dispositivos
[Zigbee](https://pt.wikipedia.org/wiki/Zigbee).

Eu não acredito neste mito, e nem [Paul Hibbert](https://www.youtube.com/channel/UCYLnawaM-36HncBBUeWrlGA).
Embora Zigbee certamente tenha vantagens (por exemplo, ele demanda menor
consumo do que WiFi e permite criar uma rede _Mesh_ entre seus dispositivos,
repetindo sinal entre eles - o que é ótimo para grandes residências), Paul
derruba o mito do limite de 20 dispositivos [neste video](https://www.youtube.com/watch?v=-etrE0A333I&t=492s).
Ele demonstra o uso de 34 dispositivos WiFi em sua casa, sem nenhum problema.
Segundo ele, muito mais dispositivos poderiam ser suportados.

Ainda que eu não pretenda ter tantos dispositivos no meu projeto minimalista
de Smart Home, uma coisa que ele diz me é particularmente interessante:

> "Se sua rede WiFi não suporta muitos dispositivos, você deve ter um
roteador ruim."

## Nossos roteadores são ruins?

Talvez. Primeiro, devemos fazer a distinção (bem) básica entre modem e roteador:
- `modem`: o dispositivo que traduz o sinal que vem da rua para sua rede interna
- `roteador`: o dispositivo que pega o sinal do modem e direciona para os
dispositivos da sua casa (seja por WiFi ou por cabo).


## Mas minha operadora só me entregou um modem

As operadoras geralmente nos entregam um modem que tem função de roteador.
Desta forma, em apenas um dispositivo nós temos a modulação/demodulação do
sinal (analógico/fibra para digital) e a transferência do sinal digital para
os dispositivos de nossa rede doméstica.

Embora prático, este dispositivo geralmente não é performático, e pode ser que
não suporte a grande quantidade de conexões WiFi que podemos precisar numa
rede com Smart Devices (já que cada dispostivo inteligente requer um IP único
na rede). O problema mais comum é que alguns dispositivos podem perder conexão,
à medida em que outros dispositivos requisitam acesso à rede.

Dssta forma, ao invés de comprar dispositivos Zigbee por causa dessa aparente
limitação (e pagar mais caro), talvez seja melhor investir num roteador melhor,
que suporte a demanda de dispositivos WiFi da sua casa.

Mas ainda fica a dúvida: como eu separo o modem do roteador se a operadora
apenas me entrega um único dispositivo?

## Modos de configuração de uma rede doméstica

Uma vez que contextualizamos a diferença entre modem e roteador, precisamos
entender como eles podem ser separados. Para isto, antes é importante
entendermos o conceito de `bridge`.

Uma `bridge` é uma configuração no modem, que permite que ele envie seu IP para
um outro dispositivo da rede (mas apenas um dispositivo), fazendo uma "ponte"
com este dispositivo. O modem continua com sua atividade de modular/demodular
sinal, mas agora ele não mais faz o papel de roteador, e delega o IP para outro
dispositivo da rede.

Este dispositivo que recebe o IP do modem pode ser um dispositivo comum que
suporte WiFi, como notebook, celular, etc - mas é melhor que seja um roteador
(que é o dispositivo capaz de suportar rede para vários outros dispositivos
da rede).
{:.note}

### Configuração tradicional: Modem com roteador no mesmo device (modo Router)

A figura abaixo apresenta o modelo tradicional entregue pelas operadoras: o
modem e o roteador estão no mesmo dispositivo.

![](/assets/img/blog/smarthome/smarthome-bridge-modo1.png)

Figura: Modem em Modo Router, com roteador atendendo vários devices.
{:.figcaption}

Na figura, o modem recebe da operadora o IP externo hipotético `187.200.1.2`, e
mapeia sua rede interna para o gateway com IP `192.168.2.1`. O mesmo equipamento
faz o papel de distribuir IPs (via DHCP) dentro da faixa para os dispositivos
da rede interna.

### Utilizando Modem modo Router com outro roteador

Esta é uma outra configuração possível:

![](/assets/img/blog/smarthome/smarthome-bridge-modo3.png)

Figura: Rede em Modo Router com outro Roteador.
{:.figcaption}

Neste caso, o modem está em Modo Router, então ele gera uma rede interna
(gateway `192.168.0.1`) e repassa para o próximo roteador. Este roteador
recebe um IP da rede do primeiro roteador (`192.168.0.12`) mas, por ser um
roteador, ele cria uma nova rede, gerando um novo range de ip
(gateway `192.168.2.1`) e então repassa para os dispositivos da casa os IPs
dentro de seu range.

Este é um exemplo de configuração que resulta da adição de um novo roteador
sem mexer em nada no modem da operadora (que vem como modo Router). Contudo,
não é um cenário ótimo, por criar uma rede desnecessária.

### Utilizando Modem modo Bridge com um único dispositivo

O modem pode ser configurado para modo Bridge. Quando isto ocorre, passa a
atuar da seguinte forma:

![](/assets/img/blog/smarthome/smarthome-bridge-modo2.png)

Figura: Modem em modo Bridge, atendendo apenas 1 Dispositivo.
{:.figcaption}

Conforme já mencionado, o modo bridge repassa o IP para apenas um dispositivo.
Na figura, o primeiro dispositivo (notebook) pegou o IP, e o segundo
dispositivo ficou sem sinal.

Quando não há outro dispositivo que precise estar na rede, esta configuração
pode ser uma opção. Contudo, geralmente não é o caso.

### Utilizando Modem modo Bridge com um roteador

Este é o cenário que desejamos:

![](/assets/img/blog/smarthome/smarthome-bridge-modo4.png)

Figura: Rede em Modo: Modem em Bridge com Roteador.
{:.figcaption}

Neste cenário, temos a separaçao das responsabilidades. O dispositivo da
operadora passa a ser apenas um modem, e repassa o IP para um roteador central.
Este roteador recebe o IP externo (que vem do modem) e cria uma rede interna,
para repassar o sinal para diferentes dispositivos da rede interna.

Este cenário é o ideal para nosso caso, porque com este projeto podemos
fazer uso de um modem de melhor qualidade, capaz de suportar muitas dezenas
de dispositivos WiFi, de forma ótima.

## Como configuro o modem da minha operadora para Modo Bridge?

Isto varia de modem para modem. No modelo Askey da Vivo Fibra, deve-se ir
no menu `Configurações/Modo da WAN` e alterar o modo do Router de
**Roteador (Padrão)** para **Bridge**.

![](/assets/img/blog/smarthome/smarthome-bridge-vivo-change-mode.png)

Figura: Alterando Modo Router para Bridge no Modem Vivo Fibra Askey.
{:.figcaption}

Imagino que o procedimento seja similar para outros equipamentos da mesma
operadora, ou mesmo de operadoras diferentes.

## Conclusão

Neste artigo, analisamos o conceito de rede com modem nos modos Bridge e
Roteador, e como podemos trabalhar com isto de forma a melhorarmos a qualidade
de nossa rede pela aquisição e incorporação de um roteador mais especializado,
de forma a não comprometer o funcionamento dos dispositivos da SmartHome.

## Referências

- Youtube: [Modo Bridge - O que é? Como Funciona?](https://www.youtube.com/watch?v=AlSJrVqudvc)
- Youtube: [Zigbee Is STILL a con, 5 Myths About The Wifi Smart Home](https://www.youtube.com/watch?v=-etrE0A333I&t=492s).
