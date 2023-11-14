---
layout: post
title: "Configurando Bridge com modem Vivo Fibra Askey e roteador Asus RT-AX82U"
description: >
  Passos para configurar o modem Vivo Fibra Askey para funcionar em modo bridge 
  com o roteador Asus RT-AX82U.
categories: smarthome
tags: smarthome
comments: true
image: /assets/img/blog/smarthome/roteador-asus-rt-ax82u.jpeg
---
> Em [artigo anterior](/blog/smarthome/2020-10-24-Separando-Modem-de-Roteador/), 
o conceito de separação de Modem e Roteador foi apresentado, juntamente com 
a utilizacão de um modo "Bridge" para fazer a ponte entre os dois 
dispositivos. Neste artigo, apresento uma implementação prática deste 
conceito, utilizando o Modem Vivo Fibra Askey e o roteador Asus RT-AX82U. 
{:.lead}

- Table of Contents
{:toc}

## Introdução

Conforme detalhado [em artigo anterior](/blog/smarthome/2020-10-24-Separando-Modem-de-Roteador/), 
o modem da Vivo Fibra (aquele que o técnico instala em casa quando assinamos 
o plano de Internet) possui as funcionalidades de modem e roteador. E 
funciona muito bem - mas pode chegar o momento em que precisemos de um 
roteador com mais funcionalidades - seja para melhor performance em jogos 
online, maior capacidade de gerir muitos dispositivos em smarthome, e etc.
Neste artigo, o dispositivo da Vivo será configurado para que funcione 
apenas como modem, e delegue a funcionalidade de roteador para um outro 
dispositivo, mais performático.

Dentre as vantagens de usar um roteador mais específico, podem-se citar:

- possibilidade de lidar com configurações mais específicas (como maior 
granularidade no controle de renovação de DHCP, ou separação mais clara 
entre banda de 2.4GHz e 5GHz);
- possibilidade de criar redes Guest, para isolar os devices de uma smarthome 
dos devices como celular e notebooks;
- possibilidade de criar redes Guest para disponibilizar o Wi-Fi para visitas
apenas para uso de Internet, sem que tenham acesso à rede interna da residência;
- possibilidade de isolamento de redes para maior performance em jogos online;
- dentre muitas outras necessidades que fogem do convencional.

> Caso as diferenças entre modem e roteador não estejam claras, recomendo 
a [leitura do artigo previamente mencionado](/blog/smarthome/2020-10-24-Separando-Modem-de-Roteador/) 
antes de prosseguir neste artigo. Tal artigo elucida os conceitos de modem 
e roteador de forma bastante clara e didática.  

### Glossário dos termos utilizados neste artigo

Para fins de clareza e desambiguação, este artigo faz uso dos termos "_Modem_ /
_Modem Vivo_" para se referir ao hardware da Vivo Fibra Askey (maiores 
informações sobre este device podem ser encontrados em dois outros artigos que 
escrevi previamente - 
[aqui](/blog/smarthome/2020-10-24-Separando-Modem-de-Roteador/) e 
[aqui](/blog/smarthome/2020-10-24-Pareando-Dispositivo-24GHz-com-Modem-Vivo-Vibra-Askey/)
), e faz uso do termo "_Roteador_" para se referir ao hardware da Asus 
([RT-AX82U](https://www.asus.com/br/networking-iot-servers/wifi-routers/asus-gaming-routers/rt-ax82u/)). 

> Contudo, há de se saber que, tecnicamente falando, o dispositivo da Vivo em 
questão agrega tanto as funcionalidades de modem quanto de roteador (porém 
em nível básico), e o dispositivo da Asus atua especificamente como roteador, 
bastante especializado e com muitos recursos.

### Sistema operacional: Arch Linux

Neste artigo, são realizados alguns comandos de sistema operacional. 
Nestes casos, será utilizado o [Arch Linux](https://archlinux.org/). 

### Riscos e Disclaimers

#### Não siga este artigo se o dispositivo da Vivo for usado para mais do que apenas Internet

Este artigo visa transformar o dispositivo da Vivo num _Bridge_, de forma a 
apenas servir Internet para o roteador. 

Desta forma, caso você tenha outros serviços da Vivo que façam uso do 
dispositivo (como TV ou Telefone, por exemplo), **os procedimentos descritos 
neste artigo não devem ser realizados**.

> Se o conceito de _Bridge_ não estiver claro, sugere-se a leitura 
[deste artigo](/blog/smarthome/2020-10-24-Separando-Modem-de-Roteador/).

#### Não siga este artigo se não tiver um bom conhecimento de redes

Este artigo sugere algumas ações que podem indisponibilizar o acesso à 
Internet. Desta forma é importante que:

- **só se realize os procedimentos aqui citados se estiver confortável com 
manutenção de rede doméstica**: Sempre tenha em mente que algumas ações 
podem indisponibilizar sua Internet, e que você terá que resolver os 
próprios problemas, sem suporte; Se não quer passar por este nervoso, 
não realize o procedimento, ou conte com alguma pessoa com maior 
conhecimento técnico para assumir essa responsabilidade por você;
  - **pense duas vezes antes de aplicar o modo bridge no roteador, pois não 
é suportado pela Vivo**: Caso hajam problemas, há um procedimento para 
resetar o modem para modo de fábrica novamente (citado abaixo, neste artigo);
  - **conheça as customizações realizadas no seu roteador, caso tenha que 
reverter para "modo fábrica"**: caso as coisas dêem errado, sempre é possível 
reverter para o modo de fábrica. Contudo, se houver alguma customização 
feita por você ou por um técnico da Vivo, apenas o modo fábrica não vai 
ser suficiente: será necessário reaplicar as customizações, ou a Internet 
não funcionará como antes. Tenha a certeza de conhecer as customizações 
do seu Modem (caso hajam), para poder reaplicá-las no eventual caso de 
restaurar para o modo fábrica.
- **não realize este procedimento se a rede Wi-Fi for a sua única 
fonte de acesso à Internet**: Entendo que só deve prosseguir se tiver um 
plano de dados da operadora de telefonia com acesso à Internet, pois 
pode ser preciso realizar pesquisas na Internet (tutoriais no Youtube ou 
posts de outros sites especializados) caso surja algum problema que 
indisponibilize seu Wi-Fi (_"nunca fique no escuro"_);
- **saiba que este artigo é apenas um guia que conta o meu relato pessoal 
de como executei o procedimento em minha rede local**: Escrevo isto esperando 
que ajude outras pessoas a elucidar o procedimento (e até mesmo para mim 
mesmo no futuro), mas não dou suporte em casos de problemas que possam 
surgir ao tentar executar estes procedimentos. Ao assumir executar estes 
passos, saiba que estará por conta própria. Por isto é importante saber 
o que se está fazendo e os impactos neste contexto (de modificar 
configurações do modem da Vivo).

## Qual o hardware empregado?

Neste artigo, farei uso de:

- Um modem da [Vivo Fibra - Askey](/blog/smarthome/2020-10-24-Pareando-Dispositivo-24GHz-com-Modem-Vivo-Vibra-Askey/#o-problema);
- Um roteador da Asus - [RT-AX82U](https://www.asus.com/br/networking-iot-servers/wifi-routers/asus-gaming-routers/rt-ax82u/);
- Um notebook;
- E um cabo de rede.

## Passos a executar

Este artigo é estruturado em termos de passos sequenciais para configuração de 
uma rede doméstica com modem (em modo Bridge) e roteador avulso. 
Estes passos foram realizados na configuração da minha rede doméstica. 

> Os passos devem ser executados em ordem.

> Peço que leia a seção "_Riscos e Disclaimers_", acima, antes de prosseguir 
com a execução dos passos deste artigo, pois tais passos podem indisponibilizar 
sua Internet - e é importante ter conhecimento técnico para lidar com os 
problemas que porventura possam surgir, e que não estarão mapeados neste artigo.

### Garantir conexão apenas via cabo 

O primeiro passo é garantir que haja conexão via cabo, e deixar de usar Wi-Fi. 

> Como desejo desligar o Wi-Fi do modem da Vivo (e deixar apenas no roteador), 
vou fazer uso da conexão via cabo neste momento (até ter o Wi-Fi habilitado no 
roteador - ao final deste artigo), para que eu não perca a conexão com o modem 
durante todo o procedimento.

Os passos são os seguintes:

- Conecte o notebook com o modem pelo cabo de rede 
- Desconecte o Wi-Fi. Para isto, no Arch Linux, execute:

```
sudo iwctl station wlan0 disconnect
```

Nenhum output será gerado. Na sequência, digite:

```
iwctl station wlan0 show
``` 

A informação exibida deve indicar que o Wi-Fi foi desconectado:

```
Station: wlan0
--------------------------------------------------------------------------------
  Settable  Property              Value
--------------------------------------------------------------------------------
            Scanning              no
            State                 disconnected
```

Para verificar que a Internet está ativa (ou seja, vindo pelo cabo de rede, já 
que o Wi-Fi está desconectado), faça um _ping_ e veja o resultado:

```
daniel@ataraxia ~> ping duckduckgo.com
PING duckduckgo.com (191.235.123.80) 56(84) bytes de dados.
64 bytes de 191.235.123.80 (191.235.123.80): icmp_seq=1 ttl=113 tempo=5.78 ms
64 bytes de 191.235.123.80 (191.235.123.80): icmp_seq=2 ttl=113 tempo=5.62 ms
64 bytes de 191.235.123.80 (191.235.123.80): icmp_seq=3 ttl=113 tempo=5.29 ms
^C
--- duckduckgo.com estatísticas de ping ---
3 pacotes transmitidos, 3 recebidos, 0% packet loss, time 2217ms
rtt min/avg/max/mdev = 5.294/5.566/5.784/0.203 ms
```

### Acessando a interface de configuração

Agora que estamos com Internet via cabo, o próximo passo é acessar o console 
de administração do Modem da Vivo. Para isto, acesse o seguinte URL via 
browser: 
[http://192.168.15.1/instalador](http://192.168.15.1/instalador).

A seguinte tela deve ser exibida:

![](/assets/img/blog/smarthome/modem-vivo-form-login.jpg)

- No campo _Usuário_, informe **support**
> O usuário _support_ é um usuário padrão do modem para este tipo de operação 
- No campo _Senha_, informe a senha do modem Vivo. 
> Este senha pode ser encontrada na etiqueta na parte de baixo do Modem Vivo 
(procure por uma caixa com o título "_Dados de acesso ao roteador_").
- Clique no botão _Entrar_ para realizar o login. 

A autenticação deve ocorrer com sucesso, e então deve ser exibida a tela 
inicial da área de administração do modem Vivo.

### Anotar o usuário do ISP

Uma vez autenticado, precisamos agora obter os dados do usuário do ISP 
(_Internet Service Provider_ - no caso, a Vivo).

Este usuário é o usuário utilizado pelo Modem para se autenticar no 
provedor da Vivo. Como vamos fazer uma configuração de _bridge_, esta 
informação será necessária. 

Para descobrir qual esse usuário (e senha) do ISP, na área logada da 
administração do Modem Vivo, acesse o menu "_Configurações > Internet_". 
Será exibido um formulário contendo estes dados. 

Anote as informações dos campos "Usuário" e "Senha" (para utilizar mais tarde).

> Geralmente, por default o usuário é "_cliente@cliente_" e a senha é 
"_cliente_". Confirme se suas informações estão desta forma. Caso estejam 
diferentes, anote estes dados para uso futuro - serão obrigatórios.

### Baixar e Instalar o App do Roteador

Esta etapa precisa ser executada agora por que, depois deste ponto, o Wi-Fi do 
Modem será desligado.

- Instale o aplicativo Asus Router (no Android ou no IPhone)

> Neste artigo, é coberta a versão para Android.

> É possível fazer esta atividade mais para a frente, mas não haverá mais 
Wi-Fi, e o app terá que ser baixado por plano de dados. O App tem 
aproximadamente 110 MB.


### Desligar o Wi-Fi do modem

Agora é preciso desligar o Wi-Fi do modem, pois não queremos que seja utilizado 
o Wi-Fi do modem, e sim apenas do roteador (lembrando que o modem vai fazer 
apenas o papel de _Bridge_ da Internet). 

- Acesse "Configuracoes > Rede WiFi 2,4 GHz"
- Na opção "Rede WiFi Privada", selecione o radiobutton "Desabilitado". 
- Clique em "Salvar", e confirme.
- Repita o procedimento acima no menu "Configurações > Rede WiFi 5 GHz".

Acesse novamente os dois menus e confirme que as configurações foram aplicadas 
com sucesso (ambos os Wi-Fi de 2.4GHZ e 5GHz estão desativados). 
Como estamos conectados via cabo, a conexao não irá cair.

### Alterar o Modem para modo Bridge

Neste ponto, vamos realizar a configuração do Modem para _Bridge_.

- Acesse o menu "Configuracoes > Modo da Wan". A seguinte tela será exibida:

![](/assets/img/blog/smarthome/modem-vivo-form-modo-wan.jpg)

> **Observe a mensagem com atenção**: é informado que, se o modo for alterado 
para Bridge, os serviços de Internet, telefonia e TV serão desabilitados. Este 
artigo considera que você não tem nada além de Internet. Caso você tenha 
telefonia ou TV, não prossiga.

- Mude a opção do combo para "Bridge" e clique em "Avancar".

A seguinte mensagem será exibida:

![](/assets/img/blog/smarthome/modem-vivo-form-modo-wan-confirm.jpg)

Novamente, é informado que os serviços de Internet, TV e telefonia serão 
desabilitados. Além disto, explica que, caso queira restaurar para o modo 
anterior, será preciso fazer um **Reset** no modem.

- Estando ciente e concordando com as restrições mencionadas, clique em 
"Estou ciente, prosseguir".

Apos isto, o modem estará configurado para modo _Bridge_. Agora será 
preciso configurar o roteador.

#### Em caso de erro: como reverter?

Caso seja aplicado o modo _Bridge_ e queira reverter as alterações, 
será necessário realizar o Reset do modem.

- Na parte de trás do Modem, procure por um botão "Reset" ou 
"Reconfigurar". Este botão é interno - será preciso fazer uso de um alfinete 
para apertá-lo. 
- Pressione o botão "Reset" (com um alfinete ou uma ferramenta de troca de 
chip de celular) por uns 10 segundos, até que o Modem pisque algumas 
luzes. Após isto, aguarde alguns momentos e volte então a conectar como 
fazia antes.

> Eu fiz o teste do "Reset", e funcionou conforme acima mencionado. Contudo, 
meu modem não tinha nenhuma customização (foi instalado com o default de 
fábrica, e eu não alterei nada). Acredito que, caso seu modem tenha alguma 
customização (como por exemplo um usuário ISP diferente), não deverá 
funcionar, pois após o Reset, a customização terá que ser novamente 
aplicada (por você ou por um técnico da Vivo, dependendo da situação).

### Ligue o roteador e conecte-o via cabo ao modem

Os passos abaixo são para ligar o roteador e conectá-lo ao Modem:

- Ligar o roteador na tomada
- Plugar o cabo de rede: este não é o cabo de rede que está ligado no 
computador; é um cabo extra que vem com o roteador. 
  - No Roteador: plugar o cabo na entrada WAN  
  - No Modem: plugar o cabo numa entrada LAN (qualquer uma)
- Confirmar conectividade: modem deve piscar luzes na porta da conexão

### Configurar o Roteador 

> Caso ainda não tenha instalado o app Asus Router (no Android ou IPhone), 
faça agora. Acima, 
[há uma seção explicando os detalhes](#baixar-e-instalar-o-app-do-roteador).

> Este artigo considera a versão Android.

- Abra o app;
- Aceite os termos de uso (precisa ter mais de 16 anos);
- Aguarde exibir as opções "Setup - setup a new router" e "Manage - manage a 
connected router";
- Selecione "Setup";
- Selecione a primeira opção:"Asus RT/GT/TUF/GS series Tour";
- Na tela "App permissions", selecione OK e permita com que o app ASUS Router 
tenha acesso ao device location;
- Deve aparecer um ponto de conexão Wi-Fi com o nome "ASUS_E0". Clique 
sobre ele;
- Na caixa de diálogo "Connect to device - ASUS Router app wants to use a 
temporary Wi-Fi network to connect to your device", confirme (clique em 
"Connect" para confirmar);
- Aguarde a conexao. Se funcionar, será exibido o roteador, e a luz RGB do 
roteador deve piscar em azul;
- Clique no botao "Get Started";
- A mensagem aparecerá: "Detecting your internet connection status". Aguarde 
a finalização;
> Caso haja algum erro de conexão com o cabo de rede, aparecerá a mensagem: 
"Connect your router - use an ethernet cable to connect your modem to your 
other WAN port". Clique em "OK", cheque se o cabo está plugado corretamente 
nas respectivas portas WAN/LAN e clique em "Retry";
- Aguarde a abertura da tela "System Setup". Nesta tela, informe o usuário 
e senha do ISP e clique em "Check";
- Na tela de "Special Requirements", não marque nada. Apenas clique em "Next";
- Na tela "Create Wi-Fi Network":
  - Selecione o checkbox "Separate 2.4GHz and 5GHz";
  - Informe um nome e senha para cada uma das duas redes; 
  - Clique em "Next";
- Na tela "Setup Local Login Account", cadastre um usuário/senha para ser o 
administrador do roteador
- Após isto, irá surgir o texto "Setting up your Network", com barra de 
progresso. Aguarde alguns segundos;
  - Aparecerá a opção "Save This Network". Clique em "Save"
- Após isto, irá aparecer o texto "Network Optimization", com barra de 
progresso. Aguarde alguns segundos;
- Finalmente, irá aparecer a mensagem "Good Connection", com os dados:
  - usuário/senha cadastrado para a rede 2.4ghz
  - usuário/senha cadastrado para a rede 5ghz
  - usuário/senha do usuário administrador do roteador Asus
- Clique em "Finish";
- Por fim, em "Enable Remote Connection", selecione "OK".

No aplicativo, pode-se ver que o LAN IP está como 192.168.50.1. Este 
é o gateway do roteador, que usa a faixa 192.168.50.2-254 para os 
dispositivos.

### Conectar o Notebook na rede Wi-Fi do roteador

Neste ponto, tudo está pronto. Agora:

- Desconecte o cabo de rede do modem com o notebook - não será mais 
preciso;
- Conecte o Notebook no Wi-Fi do roteador. No Arch Linux, pode-se 
proceder da seguinte forma:

```
sudo iwctl station wlan0 scan
sudo iwctl station wlan0 get-networks
```

O output do comando deve mostrar as duas redes (2.4GHz e 5GHz): 

```
MarkusAurelius                    psk                 ****
Epiktetos                         psk                 ****
```

Então, basta escolher uma rede e conectar a ela:

```
sudo iwctl station wlan0 connect MarkusAurelius
```

E testar a Internet com um _ping_:

```
daniel@ataraxia ~> ping duckduckgo.com
PING duckduckgo.com (191.235.123.80) 56(84) bytes de dados.
64 bytes de 191.235.123.80 (191.235.123.80): icmp_seq=1 ttl=113 tempo=6.68 ms
64 bytes de 191.235.123.80 (191.235.123.80): icmp_seq=2 ttl=113 tempo=6.43 ms
^C
--- duckduckgo.com estatísticas de ping ---
2 pacotes transmitidos, 2 recebidos, 0% packet loss, time 1428ms
rtt min/avg/max/mdev = 6.434/6.557/6.681/0.123 ms
```

## Recapitulando

Neste artigo, realizamos os seguintes passos, em ordem:

- Conectamos o Notebook ao modem via cabo, sem Wi-Fi;
- Desligamos a rede Wi-Fi do modem (ficando apenas com conexão via cabo);
- Ligamos o modo _Bridge_ do modem (para encaminhar o sinal da Internet ao 
roteador, separando bem as funções de modem para o dispositivo da Vivo e 
roteador para o dispositivo da Asus);
- Ligamos o roteador ao modem, de forma que o modem apenas encaminha o sinal 
de Internet para o roteador;
- Configuramos as redes Wi-Fi (2.4GHz e 5GHz) no novo roteador; 
- Conectamos o Notebook no Wi-Fi do roteador.

### Próximos passos

Deste momento em diante, espera-se não mais mexer no modem, pois ele foi 
simplificado apenas à função de entregar sinal de Internet ao roteador, e 
mais nada. 

Portanto, os próximos passos estão relacionados à exploração das 
funcionalidades do software de configuração de router da Asus. Esta 
configuração está acessível via HTTP em 192.168.50.1 - com autenticação 
pelo usuário administrador do roteador, previamente criado.

