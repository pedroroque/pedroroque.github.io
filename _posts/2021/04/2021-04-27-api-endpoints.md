---
layout: post
title: API Endpoints - uma (melhor?) alternativa ao tradicional controller
date: 2021-04-27 20:00:00 -0300
categories: .NET REST API
---

Não é segredo que sou fã de carteirinha de .NET Core para desenvolvimento de APIs. É um framework que permite o desenvolvimento de aplicações poderosas, performantes e que, no geral, facilita e encoraja o uso de boas práticas.

Mo entanto, tem um pormenor que sempre me deixa desconfortável: controllers!

Na maior parte dos projetos, os controllers acabam por ser uma coleção de métodos sem qualquer relação direta, a não ser o relacionamento a uma entidade de negócio. Normalmente basta olhar para a quantidade de dependências injetadas no controller para levantar a suspeita que o S de SOLID foi para o espaço. 

Para uma API do tipo:

![Open API](/assets/images/2021/04/ap_swagger.png)

Não é raro ter um controller com um construtor parecido com:

{% highlight csharp %}
[ApiController]
[Route("[controller]")]
public class ProductsController : ControllerBase
{
...        
    public ProductsController(IProductsCrudServices crudService, IProductsSalesReporting salesReporting, IProductsInventoryReports inventoryReports)
    {
        _crudService = crudService;
        _salesReporting = salesReporting;
        _inventoryReports = inventoryReports;
    }
...
}
{% endhighlight %}

Podemos com alguma facilidade imaginar um cenário em que o número de dependências cresce exponencialmente.

Até agora, a minha maneira preferida de resolver o problema é separar as funcionalidades em vários controllers, cada um dependente de um único serviço. Para controlar a geração da definição Open API do Swagger, uso o pacote de anotações do wagger.

{% highlight ssh %}
dotnet add package Swashbuckle.AspNetCore.Annotations
{% endhighlight %}

{% highlight csharp %}
services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "Api", Version = "v1" });
    c.EnableAnnotations();
});
{% endhighlight %}

{% highlight csharp %}
[ApiController]
[Route("products")]
public class ProductsCrudController : ControllerBase
{
    private readonly IProductsCrudServices _crudService;
    public ProductsCrudController(IProductsCrudServices crudService) => _crudService = crudService;

    [HttpGet]
    [SwaggerOperation(Tags = new[] { "products" })]
    public async Task<IActionResult> ListAll() { ... }

    [HttpPost]
    [SwaggerOperation(Tags = new[] { "products" })]
    public async Task<IActionResult> Post(Product product) { ... }
}
{% endhighlight %}

É uma abordagem que funciona mas que depende muito do cuidado com que é implementado. Nada impede que um desenvolvedor desavisado viole a regra.

Outra abordagem que descobri recentemente é utilizando o pacote [APIEndpoints](https://github.com/ardalis/ApiEndpoints) desenvolvido pelo [Steve 'Ardalis' Smith](https://ardalis.com/).

Trata-se de uma biblioteca que facilita a criação de API pela definição de endpoints. A sua utilização possibilita a criação de classes super focadas, que realmente aderem ao SRP (Single Responsability Principle).

{% highlight ssh %}
dotnet add package Ardalis.ApiEndpoints
{% endhighlight %}

Para criar um novo endpoint, basta herdar de BaseEndpoint ou BaseAsyncEndpoint e fazer override de Handle

{% highlight csharp %}
public class TopSalesReportEndpoint : BaseAsyncEndpoint.WithoutRequest.WithResponse<SalesReportResponse>
{
    private readonly IProductsSalesReporting _service;
    public TopSalesReportEndpoint(IProductsSalesReporting service) => _service = service;

    [HttpGet("products/reports/topsales")]
    [SwaggerOperation(Tags = new[] { "products" })]
    public override Task<ActionResult<SalesReportResponse>> HandleAsync(CancellationToken cancellationToken = default)
    {
        ...
    }
}
{% endhighlight %}

Como anteriormente, usamos as anotações do Swagger para gerar a documentação Open API. 

O que mais me agrada nesta solução é a possibilidade de standardizar a criação de APIs, sem a possibilidade de inflar a quantidade de dependências, ficando com uma série de classes pequenas e fáceis de testar.
