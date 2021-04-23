---
layout: post
title: "Configurar VS Code para usar _ como prefixo de campos privados"
date: 2021-04-22 20:00:00 -0300
categories: VS-Code .NET
---
Na minha jornada de migração do Visual Studio para VS Code, uma pormenor que me deixava frustado era a geração de campos privados. A minha preferência sempre foi usar _ como prefixo de campos privados, como maneira de diferenciar campos privados de parametros de método. 

No Visual Studio, sempre confiei no Resharper para implementar essa regra, o que não está disponível no VS Code. 

Por sorte, hoje me enviaram um link para um post do [Steve Ardalis](https://ardalis.com/configure-visual-studio-to-name-private-fields-with-underscore/) em que ele explica como configurar o visual studio para que as propriedades privadas sejam geradas com prefixo _.

Isso não resolveu meu problema, mas aguçou minha curiosidade. E como eu sei que nos repositórios do .NET é usado o mesmo padrão, resolvi ver se no repositório do aspnet core tinha alguma pista. 

Primeira coisa evidente: usam .editorconfig para partilhar as configurações básicas do projeto. (.editorconfig é um arquivo que pode ser usado para partilhar configurações entre os membros da equipe, mesmo que usem ferramentas distintas.)

Criei um arquivo .editorconfig e adicionei as configurações pretendidas

```
[*.{cs,vb}]
dotnet_naming_rule.private_members_with_underscore.symbols  = private_fields
dotnet_naming_rule.private_members_with_underscore.style    = prefix_underscore
dotnet_naming_rule.private_members_with_underscore.severity = suggestion

dotnet_naming_symbols.private_fields.applicable_kinds           = field
dotnet_naming_symbols.private_fields.applicable_accessibilities = private

dotnet_naming_style.prefix_underscore.capitalization = camel_case
dotnet_naming_style.prefix_underscore.required_prefix = _
```

Infelizmente, não funcionou... por algum motivo as configurações não eram aplicadas. 

Próximo passo, foi ver se a existia alguma configuração do ominsharp (extensão usada pelo VS Code para trabalhar com C#) que podesse estar relacionada. E não é que tem? 

![.vscode/settings.json](assets/images/2021/04/omnisharp-use-editor-formatting-settings.png)

Adicionando a linha 
```
"omnisharp.enableEditorConfigSupport": true
```
em .vscode/settings.json,

E com o novo arquivo .editorconfig, agora o refactoring para adicionar um nova propriedade já apresenta o comportamento esperado

![Adicionar campo privado](assets/images/2021/04/refactor-add-private-field.png)

Usando o arquivo .editorsettins e setando as configurações no arquivo .vscode/settings.json, garantimos que toda a equipe partilhará as mesmas configurações, independente de usar VS Code ou Visual Studio.

E já que está com a mão na massa, não deixe de adicionar ao .editorconfig:

```
[*]
indent_style = space
```

Porque amigos não deixam amigos usar TAB!
