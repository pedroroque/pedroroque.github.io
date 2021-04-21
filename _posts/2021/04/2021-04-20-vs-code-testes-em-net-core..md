---
layout: post
title: "Execução de testes em projetos .NET Core, usando VS CODE"
date: 2021-04-21 20:00:00 -0300
categories: Testing
---

Nos últimos meses tenho feito um esforço para migrar todo meu fluxo de trabalho para VS Code, em detrimento do Visual Studio.

O VS Code é extremamente rápido, super extensivel, roda em qualquer sistema operacional e é grátis!

Uma das vantagens do VS Code, também pode ser um calcanhar de aquiles: como não é um IDE normal, optimizado para um determinado fluxo de trabalho, tudo é feito por configuração. E operações que no Visual Studio nós nem nos ligamos, aqui temos que 'perder' uns minutos para deixar tudo do nosso agrado.

Um desses casos, é a execução de testes. Enquanto que no Visual Studio só temos que criar o projeto, e no máximo adicionar um nugget, aqui precisamos de escolher a maneira como iremos rodar os testes, e configurar de acordo.

O primeiro método, é executar os testes pela CLI. Para isso, precisamos de configurar uma nova task. O VS Code tem uma pasta especifica para os arquivos de configuração do projeto:

![Localização dos arquivos de configuração](/assets/images/2021/04/vs-config-files.png)

No arquivo tasks.json, preciamos de adicionar a nossa task de teste:

{% highlight json %}
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "test solution",
            "command": "dotnet",
            "type": "process",
            "args": [
                "test",
                "--no-restore",
                "${workspaceFolder}/src/HighOnSoftware.sln"
            ]
        },
    ]
}
{% endhighlight %}

O label é o nome atribuído à tarefa, command é o cli do .NET (dotnet) e os args nada mais é do que a lista de argumentos que usaria normalmente se estivesse usando dotnet na linha de comandos. 

Depois de criada a task, pode executála tanto pelo painel de comandos ou atribuindo uma confinação de teclas.

A segunda opção é a utilização da extensão [.NET Core Test Explorer](https://marketplace.visualstudio.com/items?itemName=formulahendry.dotnet-test-explorer)

Depois de instalar a extensão, ao abrir o painel de testes deve ter uma surpresa dessas:

![Localização de testes não configurada](/assets/images/2021/04/vs-no-test-config.png)

Na pasta .vscode, podemos configurar o caminho dos nossos projetos de teste:

{% highlight json %}
{
    "dotnet-test-explorer.testProjectPath": "**/*Tests.csproj",
    "dotnet-test-explorer.autoWatch": true,
    "dotnet-test-explorer.autoExpandTree": true,
    "dotnet-test-explorer.treeMode": "merged",
    "dotnet-test-explorer.showCodeLens": true,
    "dotnet-test-explorer.runInParallel": true
}
{% endhighlight %}

Depois disso, os nossos testes já são descobertos e executados normalmente:

![Execução de testes](/assets/images/2021/04/vs-test-configured.png)

A pasta .vscode deve ser versionada, para que os parametros usados possam ser partilhados por toda a equipe de desenvolvimento. Por algum motivo, já vi várias situações em que essa pasta é adicionada no .gitignore, o que não me parece fazer o mínimo sentido!

