---
author: Jan Skala
title: >-
  Best way to consume REST APIs in .NET
slug: >-
  best-way-to-consume-rest-apis-in-dotnet
date: '2020-05-01 09:00:00'
categories:
  - .NET
  - C#
  - .NET Core
tags:
  - OpenAPI
  - Swagger
excerpt_separator: <!--more-->
---
Have you ever found a well documented API? Yeah, me neither. But seriously, have you ever wanted to consume an API documented with Swagger or OpenAPI? You can do that even without making your repo dirty with generated code.
<!--more-->

What if I told you, that there is a way of adding an OpenAPI url inside your csproj and then every time you build your project the necessary code will be generated and not included inside your git repository. But first, we need to install some dotnet tools.

## OpenAPI tools
The OpenAPI.NET SDK contains a useful object model for OpenAPI documents in .NET along with common serializers to extract raw OpenAPI JSON and YAML documents from the model. We are interested in a particular one: 
**Microsoft.dotnet-openapi** a command line tool to add an OpenAPI service reference.
### Installation
To install this tool simply run:
`dotnet tool install --global Microsoft.dotnet-openapi --version 3.1.1`
### Usage
1. Navigate to the folder with your csproj
2. Run `dotnet openapi add url https://petstore.swagger.io/v2/swagger.json --output-file PetStore.json`
3. Now your csproj should be modified like this:
```xml
<ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="12.0.2" />
    <PackageReference Include="NSwag.ApiDescription.Client" Version="13.0.5" />
  </ItemGroup>
  <ItemGroup>
    <OpenApiReference Include="PetStore.json" SourceUrl="https://petstore.swagger.io/v2/swagger.json" />
  </ItemGroup>
```
4. Run `dotnet build`
5. Inside *obj* folder we can see that a file *PetStoreClient.cs* has been generated, it shares the name with our json file *PetStore.json*
6. Now consume your client in your code
```cs
var client = new PetStoreClient(new HttpClient());
var inventory = await client.GetInventoryAsync();
foreach (var (key, value) in inventory.Select(x => (x.Key, x.Value)))
{
    Console.WriteLine($"Key: {key} Value: {value}");
}
```

Yes! It was that easy. For the best performance, consider using [IHttpClientFactory](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.ihttpclientfactory?view=dotnet-plat-ext-3.1) for spawning your HttpClient. Also, as you can notice, the authorization is not generated unfortunately, so you will have to add authorization headers to the HttpClient by yourself.
