---
author: Jan Skala
title: >-
  NTLM Authentication with HTTP Client
date: '2021-02-04T15:00:00+0200'
categories:
  - .NET
  - C#
  - .NET Core
tags:
  - NTLM
  - HTTP
  - Authentication
  - Dev
excerpt_separator: <!--more-->
---

In rare cases you will face a system which is secured by **NTLM Authentication**. It can even expose a REST API. In this blog post, I will show you how to easily interact with such system using a built in [HttpClient](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httpclient?view=net-5.0). or any 3rd party Http client.
<!--more-->

# The problem
How to correctly authenticate against a RESTful service, which is secured by NTLM.

## NTLM
If you never heard of it, it stands for **NT (New Technology) LAN Manager (NTLM)**. It's a suite of Microsoft security protocols intended to provide authentication, integrity, and confidentiality to users. It is widely deployed, even on new systems, mostly because of compatibility reasons. However even Microsoft does not recommend using it.

## Correct usage of Http Client
It is not a good practice to create a new instance of HttpClient for every request you send. Mostly because an HttpClient is just a wrapper around a set of HTTP requests. The heavy lifting is done by a [HttpMessageHandler](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httpmessagehandler?view=net-5.0). By creating a new HttpClient every time with a default constructor, you are also creating a new instance of the mentioned HttpMessageHandler, This can potentially lead to System.Net.Sockets.SocketException. The best practice is to reuse HttpMessageHandler among multiple HttpClients. Microsoft recommends using [HttpClientFactory](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests) for that.

## NTLM with HttpClientHandler
Including NTLM authentication in HTTP request is pretty simple. One does simply have to set a **Credentials** property of a **HttpClientHandler**.
```cs
new HttpClientHandler
{
    Credentials = new NetworkCredential(options.Username, options.Password, options.Domain)
};
```

# The solution
Now we have to integrate all these parts together. 
## Create a configuration class for loading credentials
```cs
public class NTLMOptions
    {
        public string BaseUrl { get; set; }
        public string Username { get; set; }
        public string Password { get; set; }
        public string Domain { get; set; }
    }
```
## Create strongly typed NTLM Client
```cs
 public class NtlmClient
    {
        private readonly HttpClient _client;

        public NtlmClient(HttpClient client)
        {
            _client = client;
        }

        public Task<string> Foo()
        {
            return _client.GetStringAsync("api/foo/bar");
        }
    }
```

## Setup dependency injection
```cs
// add strongly typed options
services.AddOptions<NTLMOptions>()
        .Bind(Configuration.GetSection("NTLM"));

// configure http client
services
    .AddHttpClient<NtlmClient>((s, client) =>
    {
        var options = s.GetRequiredService<IOptions<NTLMOptions>>().Value;
        client.BaseAddress = new Uri(options.BaseUrl);
    })
    .ConfigurePrimaryHttpMessageHandler((s) =>
    {
        var options = s.GetRequiredService<IOptions<NTLMOptions>>().Value;
        return new HttpClientHandler
        {
            Credentials = new NetworkCredential(options.Username, options.Password, options.Domain)
        };
    });
```

## Use your strongly typed client
```cs
public class MyService
    {
        private readonly NtlmClient _client;

        public MyService(NtlmClient client)
        {
            _client = client;
        }

        public async Task Work()
        {
            var data = await _client.Foo();
            // work with data
        }
    }
```
Simply just request your strongly typed client as a dependency.

# Summary
1. Do not create HttpClient directly, but ask for it from dependency injection instead
2. Configure message handler to use NTLM authentication in dependency injection configuration
3. Profit!

In order to use this approach with a non build in HttpClient, one does simply have to pass the HttpClient into the 3rd party HttpClient's constructor, like in the example below:
```cs
 public class NtlmClient
    {
        private readonly IClient _client;

        public NtlmClient(HttpClient client, IOptions<NTLMOptions> options) =>
            _client = new FluentClient(new Uri(options.Value.BaseUrl), client);
```
