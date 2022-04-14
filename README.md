[![build-and-publish Workflow Status](https://github.com/warrenbrasil/HeaderPropagation/actions/workflows/build-and-publish.yml/badge.svg?branch=master)](https://github.com/warrenbrasil/HeaderPropagation/actions/workflows/build-and-publish.yml?branch=master)

|                     Package                    |                       Github Packages                      |
|:----------------------------------------------:|:----------------------------------------------------------:|
| [HeaderPropagation](https://github.com/warrenbrasil/HeaderPropagation/packages/1365822) | ![Github](https://img.shields.io/badge/github-v4.0.0-blue) |

## About HeaderPropagation

[ASP.NET Core middleware](https://github.com/dotnet/aspnetcore/tree/main/src/Middleware/HeaderPropagation) to propagate HTTP headers from the incoming request to the outgoing HTTP Client requests written in netstandard2.0.
All code is licensed under the Apache License, Version 2.0 and copyrighted by the [.NET Foundation](https://dotnetfoundation.org/).

## Motivation
It is a common use case which deserves to be included in ASP.NET Core.
Its main use case is probably to track distributed transaction which requires the ability to pass through a transaction identifier as well as generating a new one when not present.

The current ASP.NET Core policy doesn't allow to backport new features to already shipped releases, so this package has been created as [recommended](https://github.com/aspnet/AspNetCore/pull/7921#issuecomment-479717164), so it can be used today on projects that want/need to target netstandard2.0.

## Usage

In `Startup.Configure` enable the middleware:

```csharp
app.UseHeaderPropagation();
```

In `Startup.ConfigureServices` add the required services, eventually specifying a configuration action:

```csharp
services.AddHeaderPropagation(o =>
{
    // propagate the header if present
    o.Headers.Add("User-Agent");

    // if still missing, set it with a value factory
    o.Headers.Add("User-Agent", context => "Mozilla/5.0 (trust me, I'm really Mozilla!)");

    // propagate the header if present, using a different name in the outbound request
    o.Headers.Add("Accept-Language", "Lang");
});
```

If you are using the `HttpClientFactory`, add the `DelegatingHandler` to the client configuration using the `AddHeaderPropagation` extension method.

```csharp
services.AddHttpClient<GitHubClient>(c =>
{
    c.BaseAddress = new Uri("https://api.github.com/");
    c.DefaultRequestHeaders.Add("Accept", "application/vnd.github.v3+json");

}).AddHeaderPropagation();
```

Or propagate only a specific header, also redefining the name to use

```csharp
services.AddHttpClient("example", c =>
{
    c.BaseAddress = new Uri("https://api.github.com/");
    c.DefaultRequestHeaders.Add("Accept", "application/vnd.github.v3+json");

}).AddHeaderPropagation(o =>
{
    o.Headers.Add("User-Agent", "Source");
});
```

See [samples/HeaderPropagationSample](samples/HeaderPropagationSample).

## Behaviour

`HeaderPropagationOptions` contains a dictionary where the key represent the name of the header to consume from the incoming request.

Each entry define the behaviour to propagate that header as follows:

- `InboundHeaderName` is the name of the header to be captured.
- `CapturedHeaderName` determines the name of the header to be used by default for the outbound http requests. If not specified, defaults to `InboundHeaderName`.
- When present, the `ValueFilter` delegate will be evaluated once per request to provide the transformed
header value. The delegate will be called regardless of whether a header with the name corresponding to `InboundHeaderName` is present in the request. It should return `StringValues.Empty` to not add the header.
- If multiple configurations for the same header are present, the first which returns a value wins.

Please note the factory is called only once per incoming request and the same value will be used by all the
outbound calls.

`HeaderPropagationMessageHandlerOptions` allows to customize the behaviour per clients, where each entry define the behaviour as follows:

- `CapturedHeaderName` is the name of the header to be used to lookup the headers captured.
- `OutboundHeaderName` is the name of the header to be added to the outgoing http requests. If not specified, defaults to `CapturedHeaderName`.

# Acknowledgements

You can find the [list of contributions](https://github.com/aspnet/AspNetCore/commits/master/src/Middleware/HeaderPropagation) in the original repository.
