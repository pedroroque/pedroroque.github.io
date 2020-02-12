---
layout: post
title:  "Usando chocolatey para configurar ambiente de desenvolvimento"
date:   2020-02-12 19:42:37 -0300
categories: dev chocolatey
---
Todos nós já passámos por isso: um belo dia chegamos no trabalho, e temos a grande notícia de que vamos ter um PC novinho para trabalhar! SSD top, I7 de última geração e memória que quase dá para manter o chrome aberto por um dia!

Mas depois você lembra: meu pai do céu, vou ter que instalar tudo de novo!

Para aliviar o sofrimento, iremos utilizar o chocolatey para instalar tudo o que precisaremos na nossa nova máquina: Visual Studio, VS Code, Python, Node, Ruby e todas as ferramentas indispensáveis numa máquina de desenvolvimento.

O primeiro passo é instalar o chocolatey na máquina. Nada complicado, basta abrir uma janela powershell e executar, em modo administrador:

{% highlight powershell %}
PS C:\> Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
{% endhighlight %}

A partir daqui, para instalar qualquer coisa é só usar:

{% highlight powershell %}
PS C:\> choco install vscode
PS C:\> choco install nodejs
PS C:\> choco install python
{% endhighlight %}

Já é um passo numa boa direção, mas podemos fazer melhor: podemos criar um script para automatizar todo o processo:

{% highlight powershell %}
[string[]]$appList=
    'nodejs',
    'notepadplusplus',
    'python',
    'ruby',
    'jekyll', 
    'googlechrome', 
    'firefox', 
    'microsoft-edge',
    'microsoft-windows-terminal',
    'git', 
    'vscode',
    'visualstudio2019professional',
    'docker-desktop'

try {
    choco config get cachelocation
} catch {
    Write-Output "Chocolatey parece não estar instalado. Tentando instalar"
    Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072;
    Invoke-Expression ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
}

foreach ($app in $appList) {
    Write-Output "Instalando   $($app)"
    & choco install $app /y 
    Write-Output "================================================================================"
    Write-Output "$($app) Instalado "
    Write-Output "================================================================================"
}
{% endhighlight %}

Como o script está localizado no meu repositório do GitHub, posso iniciar todo o processo com uma única linha de powershell.

{% highlight powershell %}
Invoke-Expression ((New-Object System.Net.WebClient).DownloadString('https://raw.githubusercontent.com/pedroroque/scripts/master/chocolatey/devmachine.ps1'))
{% endhighlight %}
