---
layout: post
title: "Alterando portas COM no Windows"
date: 2015-08-27
comments: true
categories: c++
---
## O Problema

Recentemente precisei trabalhar em uma solução para trocar portas COM de alguns dispositivos específicos no Windows. Tínhamos um problema em que, quando conectávamos o dispositivo em uma USB específica o Windows reconhecia o mesmo e determinava que ele ia operar na porta COM1 por exemplo, mas ao desconectar esse dispositivo e conectá-lo em outra porta USB o Windows dava outra porta COM pra ele. Esse é o comportamento natural do Windows, por mais que pareça estranho. <!-- more --> 

Com esse comportamento, nossa aplicação que se comunicava com o dispositivo não o encontrava, ja que inicialmente havíamos configurado para usar a COM1 e agora ele ele estava na COMX. Depois de muito pesquisar, cheguei em um aplicativo chamado [ComPortMan](http://www.uwe-sieber.de/comportman_e.html). Esse pequeno programinha é um serviço do Windows que fica "ouvindo" dispositivos que são conectados e de acordo com uma configuração específica seta uma porta COM para ele. Esse aplicativo me mostrou que sim é possível implementar uma solução para esse problema.

## As possibilidades

Depois de muito pesquisar no MSDN da Microsoft, não encontrei nenhuma API que me permitisse alterar a porta COM de um dispositivo. Então teria que ser feito de outra forma, e nesse ponto imaginei 2 possíveis soluções:

 1. Fazer um aplicativo que "simulasse" o uso do mouse pelo usuário, e entrasse na no gerenciador de dispositivos do Windows, encontrasse o dispositivo, acessasse suas configurações avançadas e fizesse a alteração da porta. (Solução porca eu sei, mas na hora do desespero vale tudo)
 2. Encontrar aonde essa informação de qual porta COM usar para o dispositivo era armazenada e tentar trabalhar nessa alteração.

## A solução

Optei pela solução 2, e deixei a 1 para nível de desespero máximo. Em minhas pesquisas, descobri que essa informação é armazenada no registro do Windows. Mais especificamente na chave "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USB\VID_XXXX&PID_YYYY\BBBBBBBBBBBBB\Device Parameters\PortName" onde:

- XXXX: Vendor ID
- YYYY: Product ID
- BBBBBBBBBBBBB - Endereço do dispositivo dentro do barramento USB

Mas não para por aí, geralmente a porta COM dos dispositivos é colocada no nome do dispositivo também, e essa informação esta na chave "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USB\VID_XXXX&PID_YYYY\BBBBBBBBBBBBB\FriendlyName". Então parti para a mesma solução do ComPortMan, fiz um serviço do Windows em C++ que ouvia dispositivos conectados pela USB e quando identificasse que era um dos nossos equipamentos, verificafa se ele ja estava na nossa porta COM correta e caso contrário alterava esses 2 valores no registro do Windows. Para as alterações terem efeito é preciso desabilitar e habilitar o dispositivo. O Windows até provê uma API para alterar algumas informações do dispositivo [link](https://msdn.microsoft.com/en-us/library/windows/hardware/ff552169%28v=vs.85%29.aspx). Até funciona muito bem para as versões 7 e 8 do Windows, mas no Windows XP tive alguns problemas, pois o nome do dispositivo ficava com lixo na String. Pode ser a minha falta de competência em manipular Strings em c++ e como tentei por 2 dias resolver isso sem sucesso, apelei para o registro do Windows. 

Infelizmente pelo acordo de não divulgação do material produzido aqui na empresa em que trabalho, não posso dispobibilizar os fontes. Mas com essa breve explicação, se você tiver atrás de uma solução parecida, é possível resolver o seu problema. Caso tenha alguma dúvida, entre em contato que posso ajudar de alguma forma.

Vou deixar alguns links de referência:

 - Detectar dispositivos inseridos no sistema. [link](https://msdn.microsoft.com/pt-br/library/windows/desktop/aa363431%28v=vs.85%29.aspx)
 - Manipular o registro do Windows. [link](https://msdn.microsoft.com/pt-br/library/windows/desktop/ms724897%28v=vs.85%29.aspx)
 - Usei o Visual Studio 2008 para fazer esse projeto, e ele até gera um esqueleto de serviço do Windows mas não consegui fazer funcionar da forma que eu queria, então encontrei esse outro exemplo. [link](https://code.msdn.microsoft.com/windowsapps/CppWindowsService-cacf4948)
 - Projeto que mostra o uso da API de detecção de inserção/remoção de dispositivos no Windows. [link](http://www.codeproject.com/Articles/31749/Device-Information)
