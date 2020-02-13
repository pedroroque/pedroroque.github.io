---
layout: post
title:  "Publicando um pacote nuget no Azure Artifacts, num feed com escopo de projeto"
date:   2020-02-13 18:55:37 -0300
categories: devops azure pipelines
---

Há muito tempo que é possível criar e comnpartilhar feeds de pacotes (Nuget e não só) no Azure Artifacts.

Se antigamente era uma opção relativamente cara, com as alterações efetuadas no preçário dos serviços durante o ano passado, a utilização do serviço passou a um 'no brainer' total. 2 GB gratuitos dá pra guardar muito pacote!

Vem isso a respeito de uma situação que enfrentei há pouco tempo e que, por deficiência da documentação atual, me fez perder uma hora só para configurar um pipeline de build / deploy de pacotes nuget.

No caso, a publicação seria feita num feed com escopo de projeto, ao invés de escopo de organização.

Como a configuração por YAML, gostando ou não, virou standard, o meu step de publicação ficou uma coisa do género:

{% highlight yaml %}
- task: NuGetCommand@2
    displayName: 'Push Release Package'
    inputs:
    command: 'push'
    packagesToPush: '$(Pipeline.Workspace)/packages/release/*.nupkg'
    nuGetFeedType: 'internal'
    publishVstsFeed: 'MYFEED'
{% endhighlight %}

Devia funcionar, não?

Claro que não. Isso só funciona se MYFEED for definido ao nível da organização.

Se o feed tiver escopo de projeto, o valor de _'publishVstsFeed'_ tem que assumir o formato **PROJECT/FEED**

{% highlight yaml %}
- task: NuGetCommand@2
    displayName: 'Push Release Package'
    inputs:
    command: 'push'
    packagesToPush: '$(Pipeline.Workspace)/packages/release/*.nupkg'
    nuGetFeedType: 'internal'
    publishVstsFeed: 'PROJECT/MYFEED'
{% endhighlight %}
