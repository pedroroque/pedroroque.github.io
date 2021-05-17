---
layout: post
title: Configurar code coverage no VS Code
date: 2021-05-16 20:00:00 -0300
categories: vscpde testing coverage
---

Na jornada para migração de todo o fluxo de trabalho em .NET do Visual Studio code para o VS Code, vamos hoje ver como resolver o problema da análise de cobertura de testes.

Com o .net core, já temos como realizar a análise da cobertura de testes, bastando para isso adicionar o pacote coverlet.msbuild aos projetos de teste, e executar os testes com o comando

{% highlight ssh %}
dotnet test /p:CollectCoverage=true /p:CoverletOutputFormat=lcov /p:CoverletOutput=./lcov.info
{% endhighlight %}

Isso já é o suficiente para que tenhamos um resumo da cobertura de teste:

![Resumo da cobertura de testes](/assets/images/2021/05/output-teste-coverage.png)

Para integrarmos a geração e visualização da cobertura de testes com o VS Code, começamos por alterar a nossa definição de task de teste, informando os parametros adicionais (essa configuração parte do principio que temos um arquivo sln na raiz do projeto):

{% highlight json %}
{
  "label": "test",
  "command": "dotnet",
  "type": "process",
  "args": [
    "test",
    "${workspaceFolder}",
    "/p:CollectCoverage=true",
    "/p:CoverletOutputFormat=lcov",
    "/p:CoverletOutput=./lcov.info",
    "/property:GenerateFullPaths=true",
    "/consoleloggerparameters:NoSummary"
  ],
  "problemMatcher": "$msCompile",
  "group": {
    "kind": "test",
    "isDefault": true
  }
}
{% endhighlight %}

Assim, sempre que rodamos a task de teste, atualizamos o arquivo de cobertura de testes.

Para visualizar o código coberto pelos testes, uso a extensão [Coverage Gutters](https://marketplace.visualstudio.com/items?itemName=ryanluker.vscode-coverage-gutters).

![Coverage Gutters](/assets/images/2021/05/coverage-gutters.png)

Caso esteja usando a extensão [.NET Core Test Explorer](https://marketplace.visualstudio.com/items?itemName=formulahendry.dotnet-test-explorer), basta adicionar a configuração adicional:

{% highlight json %}

"dotnet-test-explorer.testArguments": "/p:CollectCoverage=true /p:CoverletOutputFormat=lcov /p:CoverletOutput=./lcov.info"

{% endhighlight %}

para que os dados de cobertura sejam atualizados sempre que rodar os testes pelo test explorer.

Num próximo post, veremos como gerar um relatório completo de cobertura de testes.
