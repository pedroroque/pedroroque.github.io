---
layout: post
title: "Deploy de aplicação React, em um webApp node.js, hospedado no Azure"
date:   2020-02-14 18:00:00 -0300
categories: devops azure pipelines react
---

Recentemente, tive que criar um pipeline de publicação de uma aplicação React em azure. Normalmente, publicaria a aplicação num blog storage, e problema resolvido. Só que nesta aplicação, usamos o routing do React, e queremos oferecer a possibilidade do usuário chegar na página com um libk do género https://site.com/produto/xyz.

Acontece que esse link não leva a lado nenhum do servidor. Se usarmos o alojamento em blob storage, o resultado seria um 404. Not good!

Precisamos de ter um servidor que nos dê a possibilidade de, para qualquer rota, devolva sempre o index.html.

Como toda o processo de desenvolvimento de React usa e abusa de node.js, decidimos que o melhor mesmo implantar a aplicação numa webApp node.js.

Para isso, foi preciso começar por configurar o servidor express. Felizmente que é uma coisa que se faz com meia dúzia de linhas de código.

A estrutura dos nossos projetos é uma coisa desse género:

```
application
└───build
└───public
└───src
│   │   file011.jsx
│   │   file012.jsx
│   │
│   └───subfolder1
```

Quando buildamos a aplicação, o resultado é combinado com o conteúdo da pasta públic, e tudo vai parar na build. Então parece óbvio que o nosso server tem que ser configurado de alguma maneira na pasta public. E isso é feito com apenas dois comandos, **executados na pasta public**:

```
npm init
npm instal express
```

Isso vai criar um novo arquivo package.json na pasta public (na realidade são dois, mas quem liga mesmo para o package-lock.json?)

{% highlight json %}
{
  "name": "reactApp",
  "version": "1.0.0",
  "description": "my react app",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.17.1"
  }
}
{% endhighlight %}

Agora falta o nosso servidor. Pelo conteúdo de package.json, vemos uma referencia a um tal de index.js. Talvez seja uma boa ideia criar um index.js!

{% highlight javascript %}
const express = require("express");
const server = express();
const options = {
  index: "index.html"
};

server.use("/", express.static("/home/site/wwwroot", options));

server.use((req, res) => res.sendFile(`${"/home/site/wwwroot"}/index.html`));

server.listen(process.env.PORT);

{% endhighlight %}

A primeira linha configura o servidor para servir conteúdo estático. Isso fará com que os arquivos da nossa aplicação sejam servidos pelo express.

A segunda instrução, encaminha todos os requests para index.html, ponto de entrada da nossa aplicação. Quando a página é renderizada no browser, o router do react toma conta das operações e renderizará a página solicitada.

Agora só resta adaptar o step de publicação no Azure Pipeline, para que tudo funcione:

{% highlight yaml %}
- task: AzureRmWebAppDeployment@4
    inputs:
        ConnectionType: AzureRM
        azureSubscription: '$(azureSubscription)'
        appType: webAppLinux
        WebAppName: '$(serverWebAppName)'
        packageForLinux: '$(System.ArtifactsDirectory)/drop/$(Build.BuildId).zip'
        StartupCommand: 'node index.js'
        ScriptType: 'Inline Script'
        InlineScript: 'npm install'
{% endhighlight %}

{% highlight yaml %}
ScriptType: 'Inline Script'
InlineScript: 'npm install'
{% endhighlight %}

Faz com que o npm install seja executado depois do deploy, e 

{% highlight yaml %}
StartupCommand: 'node index.js'
{% endhighlight %}

Configura index.js como ponto de entrada do servidor.
