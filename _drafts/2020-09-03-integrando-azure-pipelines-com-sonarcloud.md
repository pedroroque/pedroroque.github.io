---
layout: post
title: "Integrando Azure Pipelines com SonarCloud"
date: 2020-09-04 18:00:00 -0300
categories: DevOps Sonar Azure Pipelines
---
O post de hoje é sobre a integração do SonarCloud com o Azure Pipelines. 

SonarCloud é a versão SaaS do conhecido SonarQube, ferramenta considerada o padrão de análise estática de código.

O objetivo deste post é mostrar como o SonarCloud pode ser integrado num Pipelinde de validação de pull request, para podermos maximizar a nossa capacidade de detetar problemas ou bugs com o código.

Vamos começar com um pipeline padrão de validação de PR. 

``` YAML
pool:
  vmImage: "ubuntu-latest"

variables:
  buildConfiguration: "Release"
  buildSolution: "src/discografia.sln"
  testProjects: "src/tests/**/*.csproj"

steps:
  - task: DotNetCoreCLI@2
    name: "Restore"
    inputs:
      projects: $(buildSolution)
      command: "restore"

  - task: DotNetCoreCLI@2
    name: "Build"
    displayName: "dot net build $(buildConfiguration)"
    inputs:
      command: "build"
      projects: $(buildSolution)
      arguments: "--configuration $(buildConfiguration)"

  - task: DotNetCoreCLI@2
    name: "RunTests"
    displayName: "dot net test $(buildConfiguration)"
    inputs:
      command: "test"
      projects: $(testProjects)
      arguments: "--configuration $(buildConfiguration) --logger trx"
      publishTestResults: true
```
Com essa configuração, iremos restaurar todas as dependências da soluçãoo, executar o build e rodas os testes. Zero integração com Sonar nesse instante.

Rodando o pipeline, vemos que tudo parece normal, com exceção da cobertura de testes, que aparece como não configuradp.

![first run](/assets/images/2020-09-04/1.png)

Para gerar o relatório de cobertura de testes são necessários três passos:

1. instalr o pacote coverlet.msbuild em todos os projetos de teste
2. modificar o step de test, para coletar os dados de cobertura
3. adicionar task para publicar o relatório de coberura

```YAML
  - task: DotNetCoreCLI@2
    name: "RunTests"
    displayName: "dot net test $(buildConfiguration)"
    inputs:
      command: "test"
      projects: $(testProjects)
      arguments: "--configuration $(buildConfiguration) /p:CollectCoverage=true /p:CoverletOutputFormat=Cobertura /p:CoverletOutput=$(Build.SourcesDirectory)/TestResults/Coverage/ --logger trx"
      publishTestResults: true

  - task: PublishCodeCoverageResults@1
    displayName: "Publish code coverage report"
    inputs:
      codeCoverageTool: "Cobertura"
      summaryFileLocation: "$(Build.SourcesDirectory)/**/cobertura.xml"
```
Agora já é possivel ver o relatório de cobertura no pipeline

![first run](/assets/images/2020-09-04/2.png)