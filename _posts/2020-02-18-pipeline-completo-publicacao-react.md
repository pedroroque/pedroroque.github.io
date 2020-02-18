---
layout: post
title: "Pipeline de deploy react"
date:   2020-02-18 12:00:00 -0300
categories: devops azure pipelines react
---
Hoje veremos como criar um pipeline de publicação de uma single page app, no caso, uma aplicação react.js.

Muitos já conhecem o método clássico para criação de pipelines para build e release, onde montados tdo o pipeline com os wizards do portal do Azure Devops.

Com esse método é relativamente fácil criar tanto pipelines de build ou release, mas não é nada prático quando você tem uma grande quantidade de pipelines para administrar.

Aí que entra a configuração por YAML. Desse jeito, os nossos pipelines são meros arquivos YAML, que podem ser reaproveitados, versionados e fácilmente replicados.

Nesse exemplo, itemos configurar o build de uma aplicação react, com deploy automático para ambiente DEV, e publicação sujeita a aprovação para staging e produção.

Para esta demo, criei uma aplicação react padrão (npx create-react-app). As únicas alterações que foram feitas:

- Criação de arquivo .env para guardarmos nossas configurações de desenvolvimento

- Alteração do App.js para utilizar nossas configurações

- Configuração do Express na pasta public, para servir o conteúdo estático do nosso site

[Pode encontrar o código no GitHub](https://github.com/pedroroque/react-pipeline)

Se rodar a aplicação na sua máquina, deve ver isso:

![aplicação rodando localmente](/assets/images/2020-02-18/react-local.png)

Antes de criarmos nosso pipeline, precisamos de ter configurado nossos recursos no Azure (App Service Linux e Web App node.js). Também precisamos de configurar nossos ambientes de deployment no Azure DevOps. No meu caso, criei os ambientes QA e PRD, 

![ambientes de publicação](/assets/images/2020-02-18/environments.png)

e em cada uma configurei um aprovador:

![aprovação](/assets/images/2020-02-18/approvall.png)

Finalmente, configurei as conexões com a conta Azure, e GitHub:

![conexões de serviço](/assets/images/2020-02-18/serviceconnections.png)

Agora estamos prontos para começar nosso pipeline! por uma questão de organização, gosto de criar todos os artefatos relacionados com pipelines numa pasta dedicada:

![arquivo pipeline](/assets/images/2020-02-18/2020-02-18-pipeline.png)

Começo por definir minhas variáveis:

```yaml
variables:

  azureSubscription: '------------------------------'
  srcFolder: 'pipe-demo'
  webAppNameDev: 'react-deploy-dev'
  webAppNameQA: 'react-deploy-qa'
  webAppNamePrd: 'react-deploy-prd'
  vmImageName: 'ubuntu-latest'
```

Essas variaveis nos ajudaram na hora de criarmos os vários passos do nosso pipeline, e facilitam o reaproveitamento do script.

Próximo passo será a indicação de que se trata de um pipeline multi etapa. 

```yaml
stages:
  - stage: buildDev
    displayName: 'Build React App'
```

Esse pedaço de script nos diz que se trata de uma pipeline com várias etapas, e a primeira será o build da aplicação:

```yaml
  - stage: buildDev
    displayName: 'Build React App'

    jobs:
      - job: 'build_and_test'
        variables:
          REACT_APP_HELLO: 'Running on dev'
        steps:
          - task: NodeTool@0
            inputs: 
              versionSpec: '10.x'
        
          - script: |
              cd $(srcFolder)
              npm install
              npm run build          
            displayName: 'Install and Build'

          - task: CopyFiles@2
            displayName: 'Copy build output'
            inputs:
              SourceFolder: '$(srcFolder)/build'
              Contents: '**/*'
              TargetFolder: '$(Build.ArtifactStagingDirectory)'

          - task: ArchiveFiles@2
            displayName: 'Archive output'
            inputs:
              rootFolderOrFile: $(Build.ArtifactStagingDirectory)
              archiveType: 'zip'
              archiveFile: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId).dev.zip'
              includeRootFolder: false

          - task: PublishBuildArtifacts@1
            inputs:
              pathtoPublish: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId).dev.zip'
```

Nessa etapa são declaradas as variaveis de configuração do ambiente, e executados 5 tarefas:
- Uma tarefa node para instalar as dependências e buildar a aplicação. 
- Copiamos o resultado do build (pasta build), para a pasta de staging do pipeline.
- Arquivamos a nossa aplicação para um arquivo dev.zip.
- Publicamos o arquivo nos artefatos do pipeline.

Uma vez que a variavel REACT_APP_HELLO está definida dentro de um job, o seu escopo se limita ao job onde é declarada. Pode parecer estranho estarmos declarando variaveis de configuração diretamente no YAML, mas devemos lembrar que estamos buildando uma aplicação que roda no browser do usuário. Ou seja, não adianta nos preocuparmos com segredos nessa fase. Se realmente é segredo, não o inclua num SPA!

Depois da primeira etapa concluída, vem a segunda etapa, onde a aplicação é publicada no ambuente de desenvolvimento:

```yaml
  - stage: deployDev
    displayName: 'Deploy Dev'
    dependsOn: buildDev
    condition: succeeded()
```

Aqui declaramos que a etapa de deploy é dependente da etaoa de build terminar com sucesso.

```yaml
    jobs:
      - deployment: deploy
        displayName: Deploy to Dev
        environment: 'development'
        pool:
          vmImage: 'ubuntu-latest'
```
No job de publicação é definido o ambiente, que no caso de development não tem qualquer aprovação configurado, fazendo com que o deploy seja feito automáticamente.

```yaml
        strategy:
          runOnce:
            deploy:
              steps:
                - task: DownloadPipelineArtifact@2
                  displayName: "Downloading build artifacts"
                  inputs:
                    buildType: current
                    targetPath: '$(System.ArtifactsDirectory)'

                - task: AzureRmWebAppDeployment@4
                  inputs:
                    ConnectionType: AzureRM
                    azureSubscription: '$(azureSubscription)'
                    appType: webAppLinux
                    WebAppName: '$(webAppNameDev)'
                    packageForLinux: '$(System.ArtifactsDirectory)/drop/$(Build.BuildId).dev.zip'
                    StartupCommand: 'node index.js'
                    ScriptType: 'Inline Script'
                    InlineScript: 'npm install'
```
O job de publicação em si é configurado como sendo constituído por duas tarefas, sendo que a primeira baixa o artefato de publicação, e a segunda faz a publicação numa web app do Azure. De salientar apenas a definição do script para rodar o npm install, que fará com que as dependencias do Express sejam instaladas, e a definição do ponto de entrada do servidor como sendo node index.js.

Para definirmos a publicação nos ambiente de QA e Produção, precisamos de buildar a aplicaçpão para cada um dos ambiente, e fazer a publicação como fizemos em dev.

```yaml
  - stage: buildQA
    displayName: 'Build React App'
    dependsOn: deployDev
    condition: succeeded()

    jobs:
      - job: 'build'
        variables:
          REACT_APP_HELLO: 'Running on QA'
        steps:
          - task: NodeTool@0
            inputs: 
              versionSpec: '10.x'
        
          - script: |
              cd $(srcFolder)
              npm install
              npm run build          
            displayName: 'Install and Build'

          - task: CopyFiles@2
            displayName: 'Copy build output'
            inputs:
              SourceFolder: '$(srcFolder)/build'
              Contents: '**/*'
              TargetFolder: '$(Build.ArtifactStagingDirectory)'

          - task: ArchiveFiles@2
            displayName: 'Archive output'
            inputs:
              rootFolderOrFile: $(Build.ArtifactStagingDirectory)
              archiveType: 'zip'
              archiveFile: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId).QA.zip'
              includeRootFolder: false

          - task: PublishBuildArtifacts@1
            inputs:
              pathtoPublish: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId).QA.zip'

  - stage: deployQA
    displayName: 'Deploy QA'
    dependsOn: buildQA
    condition: succeeded()

    jobs:
      - deployment: deploy
        displayName: Deploy to QA
        environment: 'QA'
        pool:
          vmImage: 'ubuntu-latest'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: DownloadPipelineArtifact@2
                  displayName: "Downloading build artifacts"
                  inputs:
                    buildType: current
                    targetPath: '$(System.ArtifactsDirectory)'

                - task: AzureRmWebAppDeployment@4
                  inputs:
                    ConnectionType: AzureRM
                    azureSubscription: '$(azureSubscription)'
                    appType: webAppLinux
                    WebAppName: '$(webAppNameQA)'
                    packageForLinux: '$(System.ArtifactsDirectory)/drop/$(Build.BuildId).QA.zip'
                    StartupCommand: 'node index.js'
                    ScriptType: 'Inline Script'
                    InlineScript: 'npm install'

  - stage: buildPrd
    displayName: 'Build React Production App'
    dependsOn: deployQA
    condition: succeeded()

    jobs:
      - job: 'build'
        variables:
          REACT_APP_HELLO: 'Running on Production'
        steps:
          - task: NodeTool@0
            inputs: 
              versionSpec: '10.x'
        
          - script: |
              cd $(srcFolder)
              npm install
              npm run build          
            displayName: 'Install and Build'

          - task: CopyFiles@2
            displayName: 'Copy build output'
            inputs:
              SourceFolder: '$(srcFolder)/build'
              Contents: '**/*'
              TargetFolder: '$(Build.ArtifactStagingDirectory)'

          - task: ArchiveFiles@2
            displayName: 'Archive output'
            inputs:
              rootFolderOrFile: $(Build.ArtifactStagingDirectory)
              archiveType: 'zip'
              archiveFile: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId).PRD.zip'
              includeRootFolder: false

          - task: PublishBuildArtifacts@1
            inputs:
              pathtoPublish: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId).PRD.zip'

  - stage: deployPrd
    displayName: 'Deploy Production'
    dependsOn: buildPrd
    condition: succeeded()

    jobs:
      - deployment: deploy
        displayName: Deploy to Production
        environment: 'QA'
        pool:
          vmImage: 'ubuntu-latest'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: DownloadPipelineArtifact@2
                  displayName: "Downloading build artifacts"
                  inputs:
                    buildType: current
                    targetPath: '$(System.ArtifactsDirectory)'

                - task: AzureRmWebAppDeployment@4
                  inputs:
                    ConnectionType: AzureRM
                    azureSubscription: '$(azureSubscription)'
                    appType: webAppLinux
                    WebAppName: '$(webAppNamePrd)'
                    packageForLinux: '$(System.ArtifactsDirectory)/drop/$(Build.BuildId).PRD.zip'
                    StartupCommand: 'node index.js'
                    ScriptType: 'Inline Script'
                    InlineScript: 'npm install'

```

Depois disso, ficamos com nosso pipeline pronto para automatizar todo o pipeline, com um script versionado no git.

![pipeline completo](/assets/images/2020-02-18/2020-02-18-full-pipeline.png)

E a nossa aplicação já está publicada em 3 ambientes, cada um com suas variáveis de configuração:

![dev](/assets/images/2020-02-18/2020-02-18-running-on-dev.png)
![dev](/assets/images/2020-02-18/2020-02-18-running-on-qa.png)
![dev](/assets/images/2020-02-18/2020-02-18-running-on-prd.png)