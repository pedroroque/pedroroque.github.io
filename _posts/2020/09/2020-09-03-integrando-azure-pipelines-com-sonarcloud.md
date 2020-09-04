---
layout: post
title: "Integrando Azure Pipelines com SonarCloud"
date: 2020-09-04 18:00:00 -0300
categories: DevOps Sonar Azure Pipelines
---

O post de hoje é sobre a integração do SonarCloud com o Azure Pipelines.

SonarCloud é a versão SaaS do conhecido SonarQube, ferramenta considerada o padrão de análise estática de código.

O objetivo deste post é mostrar como o SonarCloud pode ser integrado num Pipelinde de validação de pull request, para podermos maximizar a nossa capacidade de detetar problemas ou bugs com o código.

Partindo do princípio de que já foi criada uma conexão de serviço no Azure para o SoundCloud, vamos começar com um pipeline padrão de validação de PR.

```yaml
trigger: none

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
2. modificar o passo de teste, para coletar os dados de cobertura
3. adicionar uma tarefa para publicar o relatório de cobertura

```yaml
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
      summaryFileLocation: "$(Build.SourcesDirectory)/**/*.cobertura.xml"
```

Agora já é possível ver o relatório de cobertura no pipeline

![first run](/assets/images/2020-09-04/2.png)

Para fazermos a integração com SonarCloud, precisamos adicionar 3 novoa etapas no nosso pipeline, e corrigir a geração do relatório de cobertura:

1. Preparar a análize de código pelo SonarCloud, imadiatemente antes da etape de build

```yaml
  - task: SonarCloudPrepare@1
    inputs:
      SonarCloud: $(sonar.connection)
      organization: $(sonar.organization)
      scannerMode: "MSBuild"
      projectKey: $(sonar.projectKey)
      projectName: $(sonar.projectName)
      extraProperties: |
        sonar.exclusions=**/obj/**,**/*.dll
        sonar.cs.opencover.reportsPaths=$(Build.SourcesDirectory)/**/coverage.opencover.xml
        sonar.cs.vstest.reportsPaths=$(Build.SourcesDirectory)/**/*.trx
```
2. Modificar a geração de relatório de cobertura

Para podermos ter o relatório de cobertura de código tanto no Azure Pipelines como no SonarClound, temos que resolver um problema: Azure Pipelines não aceita o formato OpenCover, e por sua vez o SonarCloud não aceita o Cobertura! 

Como solução, geramos o relatório no formato OpenCover e adicionamos um script para converter para Cobertura. 

```yaml
  - task: DotNetCoreCLI@2
    name: "RunTests"
    displayName: "dot net test $(buildConfiguration)"
    inputs:
      command: "test"
      projects: $(testProjects)
      arguments: "--configuration $(buildConfiguration) /p:CollectCoverage=true /p:CoverletOutputFormat=opencover /p:CoverletOutput=$(Build.SourcesDirectory)/TestResults/Coverage/ --logger trx"
      publishTestResults: true

  - script: |
      dotnet tool install dotnet-reportgenerator-globaltool --tool-path . 
      ./reportgenerator "-reports:$(Build.SourcesDirectory)/TestResults/Coverage/coverage.opencover.xml" "-targetdir:coverage/Cobertura" "-reporttypes:Cobertura;HTMLInline;HTMLChart"
    condition: eq( variables['Agent.OS'], 'Linux' )
    displayName: Run Reportgenerator on Linux

  - script: |
      dotnet tool install dotnet-reportgenerator-globaltool --tool-path .
      .\reportgenerator.exe "-reports:$(Build.SourcesDirectory)/TestResults/Coverage/coverage.opencover.xml" "-targetdir:coverage/Cobertura" "-reporttypes:Cobertura;HTMLInline;HTMLChart"
    condition: eq( variables['Agent.OS'], 'Windows_NT' )
    displayName: Run Reportgenerator on Windows
```
3. Por fim, executamos o Code Analyse e publicamos os resultados

```yaml
  - task: SonarCloudAnalyze@1

  - task: PublishCodeCoverageResults@1
    displayName: "Publish code coverage report"
    inputs:
      codeCoverageTool: "Cobertura"
      summaryFileLocation: "$(Build.SourcesDirectory)/**/Cobertura.xml"

  - task: SonarCloudPublish@1
    inputs:
      pollingTimeoutSec: "300"
```

Pipeline completo:

```yaml
trigger: none

pool:
  vmImage: "ubuntu-latest"

variables:
  buildConfiguration: "Release"
  buildSolution: "src/discografia.sln"
  testProjects: "src/tests/**/*.csproj"
  sonar.connection: "SonarCloud"
  sonar.organization: "highonsoftware"
  sonar.projectKey: "rcm-discografia"
  sonar.projectName: "RCM Backend"

steps:
  - task: DotNetCoreCLI@2
    name: "Restore"
    inputs:
      projects: $(buildSolution)
      command: "restore"

  - task: SonarCloudPrepare@1
    inputs:
      SonarCloud: $(sonar.connection)
      organization: $(sonar.organization)
      scannerMode: "MSBuild"
      projectKey: $(sonar.projectKey)
      projectName: $(sonar.projectName)
      extraProperties: |
        sonar.exclusions=**/obj/**,**/*.dll
        sonar.cs.opencover.reportsPaths=$(Build.SourcesDirectory)/**/coverage.opencover.xml
        sonar.cs.vstest.reportsPaths=$(Build.SourcesDirectory)/**/*.trx

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
      arguments: "--configuration $(buildConfiguration) /p:CollectCoverage=true /p:CoverletOutputFormat=opencover /p:CoverletOutput=$(Build.SourcesDirectory)/TestResults/Coverage/ --logger trx"
      publishTestResults: true

  - script: |
      dotnet tool install dotnet-reportgenerator-globaltool --tool-path . 
      ./reportgenerator "-reports:$(Build.SourcesDirectory)/TestResults/Coverage/coverage.opencover.xml" "-targetdir:coverage/Cobertura" "-reporttypes:Cobertura;HTMLInline;HTMLChart"
    condition: eq( variables['Agent.OS'], 'Linux' )
    displayName: Run Reportgenerator on Linux

  - script: |
      dotnet tool install dotnet-reportgenerator-globaltool --tool-path .
      .\reportgenerator.exe "-reports:$(Build.SourcesDirectory)/TestResults/Coverage/coverage.opencover.xml" "-targetdir:coverage/Cobertura" "-reporttypes:Cobertura;HTMLInline;HTMLChart"
    condition: eq( variables['Agent.OS'], 'Windows_NT' )
    displayName: Run Reportgenerator on Windows

  - task: SonarCloudAnalyze@1

  - task: PublishCodeCoverageResults@1
    displayName: "Publish code coverage report"
    inputs:
      codeCoverageTool: "Cobertura"
      summaryFileLocation: "$(Build.SourcesDirectory)/**/Cobertura.xml"

  - task: SonarCloudPublish@1
    inputs:
      pollingTimeoutSec: "300"

```

Com isso, temos um pipeline de validação de pull request, devidamente integrado com SonarCloud. 

Num próximo post, iremos ver como integrar o resultado da análise de código diretamente no nosso PR.
