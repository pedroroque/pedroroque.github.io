---
layout: post
title: "Pipeline de deploy react"
date:   2020-02-18 18:00:00 -0300
categories: devops azure pipelines react
---
Hoje veremos como criar um pipeline de publicação de uma single page app, no caso, uma aplicação react.js.

Muitos já conhecem o método clássico para criação de pipelines para build e release, onde montados tdo o pipeline com os wizards do portal do Azure Devops.

Com esse método é relativamente fácil criar tanto pipelines de build ou release, mas não é nada prático quando você tem uma grande quantidade de pipelines para administrar.

Aí que entra a configuração por YAML. Desse jeito, os nossos pipelines são meros arquivos YAML, que podem ser reaproveitados, versionados e fácilmente replicados.

Nesse exemplo, itemos configurar o build de uma aplicação react, com deploy automático para ambiente DEV, e publicação sujeita a aprovação para staging e produção.
