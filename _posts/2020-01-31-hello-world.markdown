---
layout: post
title:  "Bem Vindo"
date:   2020-02-04 19:42:37 -0300
categories: jekyll
---
Depois de muita procrastinação, eis que finalmente meu blog vê a luz do dia!

Depois de muita experimentação me decidi pela utilização de Jekyll para criar o site. Básicamente, Jekyll pega seu conteúdo criado em Markdown e cria um site estático, que pode ser alojado gratuitamente no GitHub.

A grande vantagem do Jekyll é a sua simplicidade e performance. Problemas, só a instalação das dependências do ruby numa máquina windows. Definitivamente, acho que o Yukihiro Matsumoto não tava nem aí pra nós que usamos windows. Se posso deixar alguma dica nesse momento é, caso ainda não usem, dêm uma olhada no chocolatey. O bagulho instala tudo o que possa precisar numa máquina dev.

Depois do ruby instalado, são literalmente 3 comandos para criar um site:

{% highlight bash %}
~$ gem install bundler jekyll
~$ jekyll new my-awesome-site
~$ cd my-awesome-site
{% endhighlight %}

E para rodar seu novo site:
{% highlight bash %}
~/my-awesome-site$ bundle exec jekyll serve
#=> agora abra o browser em http://localhost:4000
{% endhighlight %}

[Jekyll](https://jekyllrb.com/ "Jekyll")

[Chocolatey](https://chocolatey.org// "Chocolatey")
