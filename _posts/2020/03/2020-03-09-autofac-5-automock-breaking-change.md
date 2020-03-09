---
layout: post
title:  "Breaking Changes em Autofac.Extras.Moq"
date:   2020-03-08 19:42:37 -0300
categories: jekyll
---
Se, tal como eu, você é fã do Autofac e Moq, populares frameworks de, respetivamente, injeção de dependência e mocking para .NET, provavelmente já conhece o Automoq. 
Com Automoq nossos testes tendem a ficar bem mais concisos, sem toda a cerimónia inerente à utilização de um framework como Moq.
Um teste que tradicionalmente seria algo do género:

```c#
[Fact]
public void TestingWithMoq()
{
    var oneMockedService = new Mock<IOneService>();
    oneMockedService.Setup(x => x.DoStuff(It.Is<string>((s) => s.Equals("hello"))))
            .Returns("World");

    var anotherMockedService = new Mock<IAnotherService>();
    anotherMockedService.Setup(x => x.DoAnotherThing(It.IsAny<string>()));

    var app = new AmazingApp(oneMockedService.Object, anotherMockedService.Object);

    var result = app.Execute("hello");
    
    result.ShouldBe("World");
}
```

Com Automoq fica:

```c#
[Fact]
public void TestingWithAutoMoq()
{
    using (var mock = AutoMock.GetLoose()) 
    {
        mock.Mock<IOneService>()
            .Setup(x => x.DoStuff(It.Is<string>((s) => s.Equals("hello"))))
            .Returns("World");

        var result = mock.Create<AmazingApp>().Execute("hello");

        result.ShouldBe("World");
    }
}
```

Na minha modesta opinião, muito melhor. Só tenho que me preocupar em fazer o setup das dependências que realmente me interessam, e para as outras o Automoq fornece uma implementação padrão.
Como nem tudo é 0800, quando temos que testar uma classe em que uma das dependências é uma instância concreta, temos um pouco de trabalho extra. Antes da versão 5, podíamos utilizar mock.Provide para indicar ao framework que pretendemos utilizar uma implementação concreta da nossa dependência:

Nesse caso, IOneService continua sendo um Mock, mas para IAnotherService o framework injetará a implementação AnotherServiceFake.

```c#
[Fact]
public void TestingWithProvide()
{
    using (var mock = AutoMock.GetStrict())
    {
        mock.Mock<IOneService>()
            .Setup(x => x.DoStuff(It.Is<string>((s) => s.Equals("hello"))))
            .Returns("World");

        mock.Provide<IAnotherService>(new AnotherServiceFake());

        var result = mock.Create<AmazingApp>().Execute("hello");

        result.ShouldBe("World");
    }
}
```

Com a versão 5, a extensão Provide já não existe. No seu lugar, podemos fazer a configuração passando uma Action para configurar o container logo na criação do Automock:

```c#
[Fact]
public void TestingWithBuilderAction()
{
    using (var mock = AutoMock.GetStrict(
            builder => builder.RegisterInstance(new AnotherServiceFake()).As<IAnotherService>())
        )
    {
        mock.Mock<IOneService>()
            .Setup(x => x.DoStuff(It.Is<string>((s) => s.Equals("hello"))))
            .Returns("World");

        var result = mock.Create<AmazingApp>().Execute("hello");

        result.ShouldBe("World");
    }
}
```

Esta última opção se torna ainda mais poderosa quando nos apercebemos que o Automoq expõe o ContainerBuilder, e com ele podemos fazer toda a nossa configuração de dependência, incluíndo com a utilização de Modules. Mas isso fica outro post!
