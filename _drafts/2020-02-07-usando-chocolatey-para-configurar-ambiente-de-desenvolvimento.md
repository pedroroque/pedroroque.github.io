---
layout: post
title:  "Usando chocolatey para configurar ambiente de desenvolvimento"
date:   2020-02-07 19:42:37 -0300
categories: dev chocolatey
---
Todos nós já passámos por isso: um belo dia chegamos no trabalho, e temos a grande notícia de que vamos ter um PC novinho para trabalhar! SSD top, I7 de última geração e memória que quase dá para manter o chrome aberto por um dia!

Mas depois você lembra: meu pai do céu, vou ter que instalar tudo de novo!

Para aliviar o sofrimento, iremos utilizar o chocolatey para instalar tudo o que precisaremos na nossa nova máquina: Visual Studio, VS Code, Python, Node, e Ruby.

O primeiro passo é instalar o chocolatey na máquina. Nada complicado, basta abrir uma janela powershell e executar, em modo administrador:

{% highlight powershell %}
PS C:\> Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
{% endhighlight %}

