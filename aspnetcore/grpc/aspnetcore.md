---
title: 使用 ASP.NET Core 的 gRPC 服务
author: juntaoluo
description: 了解 ASP.NET Core 编写 gRPC services 时的基本概念。
monikerRange: '>= aspnetcore-3.0'
ms.author: johluo
ms.date: 09/03/2019
uid: grpc/aspnetcore
ms.openlocfilehash: 28e6b8589bbe0b6a3723b64736c723c883302571
ms.sourcegitcommit: e6bd2bbe5683e9a7dbbc2f2eab644986e6dc8a87
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/03/2019
ms.locfileid: "70238161"
---
# <a name="grpc-services-with-aspnet-core"></a>使用 ASP.NET Core 的 gRPC 服务

本文档演示如何使用 ASP.NET Core 开始使用 gRPC 服务。

## <a name="prerequisites"></a>系统必备

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.0.md)]

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.0.md)]

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.0.md)]

---

## <a name="get-started-with-grpc-service-in-aspnet-core"></a>开始使用 ASP.NET Core 中的 gRPC 服务

[查看或下载示例代码](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/grpc/grpc-start/sample)（[如何下载](xref:index#how-to-download-a-sample)）。

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

有关如何创建 gRPC 项目的详细说明，请参阅[gRPC services 入门](xref:tutorials/grpc/grpc-start)。

# <a name="visual-studio-code--visual-studio-for-mactabvisual-studio-codevisual-studio-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

在命令行中运行 `dotnet new grpc -o GrpcGreeter`。

---

## <a name="add-grpc-services-to-an-aspnet-core-app"></a>将 gRPC 服务添加到 ASP.NET Core 应用

gRPC 需要[gRPC](https://www.nuget.org/packages/Grpc.AspNetCore)包。

### <a name="configure-grpc"></a>配置 gRPC

在 Startup.cs 中：

* gRPC 是通过`AddGrpc`方法启用的。
* 每个 gRPC 服务通过`MapGrpcService`方法添加到路由管道。

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeter/Startup.cs?name=snippet&highlight=7,24)]

ASP.NET Core 中间件和功能共享路由管道，因此可以将应用配置为提供其他请求处理程序。 其他请求处理程序（如 MVC 控制器）与已配置的 gRPC 服务并行工作。

### <a name="configure-kestrel"></a>配置 Kestrel

Kestrel gRPC 终结点：

* 需要 HTTP/2。
* 应通过 HTTPS 进行保护。

#### <a name="http2"></a>HTTP/2

gRPC 要求 HTTP/2。 gRPC for ASP.NET Core 验证[HttpRequest](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*)为`HTTP/2`。

在大多数现代操作系统上，Kestrel[支持 HTTP/2](xref:fundamentals/servers/kestrel#http2-support) 。 默认情况下，Kestrel 终结点配置为支持 HTTP/1.1 和 HTTP/2 连接。

#### <a name="https"></a>HTTPS

用于 gRPC 的 Kestrel 终结点应使用 HTTPS 进行保护。 在开发中， `https://localhost:5001`当存在 ASP.NET Core 开发证书时，将自动创建 HTTPS 终结点。 不需要配置。

在生产环境中，必须显式配置 HTTPS。 在下面的*appsettings*示例中，提供了一个使用 HTTPS 保护的 HTTP/2 终结点：

```json
{
  "Kestrel": {
    "Endpoints": {
      "HttpsDefaultCert": {
        "Url": "https://localhost:5001",
        "Protocols": "Http2"
      }
    },
    "Certificates": {
      "Default": {
        "Path": "<path to .pfx file>",
        "Password": "<certificate password>"
      }
    }
  }
}
```

或者，可以在*Program.cs*中配置 Kestrel 终结点：

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.ConfigureKestrel(options =>
            {
                // This endpoint will use HTTP/2 and HTTPS on port 5001.
                options.Listen(IPAddress.Any, 5001, listenOptions =>
                {
                    listenOptions.Protocols = HttpProtocols.Http2;
                    listenOptions.UseHttps("<path to .pfx file>", 
                        "<certificate password>");
                });
            });
            webBuilder.UseStartup<Startup>();
        });
```

如果未使用 HTTPS 配置 HTTP/2 终结点，则终结点的[ListenOptions](xref:fundamentals/servers/kestrel#listenoptionsprotocols)必须设置为`HttpProtocols.Http2`。 `HttpProtocols.Http1AndHttp2`无法使用，因为需要使用 HTTPS 来协商 HTTP/2。 如果没有 HTTPS，则端点的所有连接默认为 HTTP/1.1，并且 gRPC 调用失败。

有关启用 HTTP/2 和 HTTPS with Kestrel 的详细信息，请参阅[Kestrel 终结点配置](xref:fundamentals/servers/kestrel#endpoint-configuration)。

> [!NOTE]
> macOS 不支持[传输层安全（TLS）](https://tools.ietf.org/html/rfc5246)ASP.NET Core gRPC。 在 macOS 上成功运行 gRPC 服务需要其他配置。 有关详细信息，请参阅[无法在 macOS 上启用 ASP.NET Core gRPC 应用](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos)。

## <a name="integration-with-aspnet-core-apis"></a>与 ASP.NET Core Api 集成

gRPC 服务对 ASP.NET Core 功能（如[依赖关系注入](xref:fundamentals/dependency-injection)（DI）和[日志记录](xref:fundamentals/logging/index)）具有完全访问权限。 例如，服务实现可以通过构造函数从 DI 容器解析记录器服务：

```csharp
public class GreeterService : Greeter.GreeterBase
{
    public GreeterService(ILogger<GreeterService> logger)
    {
    }
}
```

默认情况下，gRPC 服务实现可以解析具有任意生存期（单独、作用域或暂时性）的其他 DI 服务。

### <a name="resolve-httpcontext-in-grpc-methods"></a>解析 gRPC 方法中的 HttpContext

GRPC API 提供对某些 HTTP/2 消息数据（如方法、主机、标头和尾部）的访问权限。 通过传递给每`ServerCallContext`个 gRPC 方法的参数访问：

[!code-csharp[](~/grpc/aspnetcore/sample/GrcpService/GreeterService.cs?highlight=3-4&name=snippet)]

`ServerCallContext`在所有 ASP.NET api 中都`HttpContext`不提供对的完全访问权限。 扩展方法提供对在 ASP.NET api 中`HttpContext`表示基础 HTTP/2 消息的完全访问权限： `GetHttpContext`

[!code-csharp[](~/grpc/aspnetcore/sample/GrcpService/GreeterService2.cs?highlight=6-7&name=snippet)]

## <a name="additional-resources"></a>其他资源

* <xref:tutorials/grpc/grpc-start>
* <xref:grpc/index>
* <xref:grpc/basics>
* <xref:fundamentals/servers/kestrel>
