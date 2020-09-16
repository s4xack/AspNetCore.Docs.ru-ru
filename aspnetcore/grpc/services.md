---
title: Создание служб и методов gRPC
author: jamesnk
description: Узнайте, как создавать службы и методы gRPC.
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 08/25/2020
no-loc:
- ASP.NET Core Identity
- cookie
- Cookie
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: grpc/services
ms.openlocfilehash: 878792120d69bea9ca6f620a87a7e04da2ec1815
ms.sourcegitcommit: 111b4e451da2e275fb074cde5d8a84b26a81937d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/27/2020
ms.locfileid: "89040844"
---
# <a name="create-grpc-services-and-methods"></a><span data-ttu-id="ceb2c-103">Создание служб и методов gRPC</span><span class="sxs-lookup"><span data-stu-id="ceb2c-103">Create gRPC services and methods</span></span>

<span data-ttu-id="ceb2c-104">Автор: [Джеймс Ньютон-Кинг (James Newton-King)](https://twitter.com/jamesnk)</span><span class="sxs-lookup"><span data-stu-id="ceb2c-104">By [James Newton-King](https://twitter.com/jamesnk)</span></span>

<span data-ttu-id="ceb2c-105">В этом документе объясняется, как создать службы и методы gRPC в C#.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-105">This document explains how to create gRPC services and methods in C#.</span></span> <span data-ttu-id="ceb2c-106">Будут рассмотрены следующие задачи:</span><span class="sxs-lookup"><span data-stu-id="ceb2c-106">Topics include:</span></span>

* <span data-ttu-id="ceb2c-107">Определение служб и методов в файлах *.proto*.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-107">How to define services and methods in *.proto* files.</span></span>
* <span data-ttu-id="ceb2c-108">Код, созданный с помощью инструментов gRPC в C#.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-108">Generated code using gRPC C# tooling.</span></span>
* <span data-ttu-id="ceb2c-109">Реализация служб и методов gRPC.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-109">Implementing gRPC services and methods.</span></span>

## <a name="create-new-grpc-services"></a><span data-ttu-id="ceb2c-110">Создание новых служб gRPC</span><span class="sxs-lookup"><span data-stu-id="ceb2c-110">Create new gRPC services</span></span>

<span data-ttu-id="ceb2c-111">[Службы gRPC на языке C#](xref:grpc/basics) предоставляют подход к разработке API на основе контракта gRPC.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-111">[gRPC services with C#](xref:grpc/basics) introduced gRPC's contract-first approach to API development.</span></span> <span data-ttu-id="ceb2c-112">Службы и сообщения определяются в файлах *.proto*.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-112">Services and messages are defined in *.proto* files.</span></span> <span data-ttu-id="ceb2c-113">Затем средства C# создают код из файлов *.proto*.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-113">C# tooling then generates code from *.proto* files.</span></span> <span data-ttu-id="ceb2c-114">Для ресурсов на стороне сервера абстрактный базовый тип создается для каждой службы вместе с классами для любых сообщений.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-114">For server-side assets, an abstract base type is generated for each service, along with classes for any messages.</span></span>

<span data-ttu-id="ceb2c-115">Следующий файл *.proto*:</span><span class="sxs-lookup"><span data-stu-id="ceb2c-115">The following *.proto* file:</span></span>

* <span data-ttu-id="ceb2c-116">Определяет службу `Greeter`.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-116">Defines a `Greeter` service.</span></span>
* <span data-ttu-id="ceb2c-117">Служба `Greeter` определяет вызов `SayHello`.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-117">The `Greeter` service defines a `SayHello` call.</span></span>
* <span data-ttu-id="ceb2c-118">`SayHello` отправляет сообщение `HelloRequest` и получает сообщение `HelloReply`.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-118">`SayHello` sends a `HelloRequest` message and receives a `HelloReply` message</span></span>

```protobuf
syntax = "proto3";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

<span data-ttu-id="ceb2c-119">Средства C# создают базовый тип C# `GreeterBase`.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-119">C# tooling generates the C# `GreeterBase` base type:</span></span>

```csharp
public abstract partial class GreeterBase
{
    public virtual Task<HelloReply> SayHello(HelloRequest request, ServerCallContext context)
    {
        throw new RpcException(new Status(StatusCode.Unimplemented, ""));
    }
}

public class HelloRequest
{
    public string Name { get; set; }
}

public class HelloReply
{
    public string Message { get; set; }
}
```

<span data-ttu-id="ceb2c-120">По умолчанию созданный `GreeterBase` не выполняет никаких действий.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-120">By default the generated `GreeterBase` doesn't do anything.</span></span> <span data-ttu-id="ceb2c-121">Его виртуальный метод `SayHello` возвращает ошибку `UNIMPLEMENTED` для всех клиентов, которые ее вызывают.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-121">Its virtual `SayHello` method will return an `UNIMPLEMENTED` error to any clients that call it.</span></span> <span data-ttu-id="ceb2c-122">Чтобы службу можно было использовать, приложение должно создать конкретную реализацию `GreeterBase`.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-122">For the service to be useful an app must create a concrete implementation of `GreeterBase`:</span></span>

```csharp
public class GreeterService : GreeterBase
{
    public override Task<HelloReply> UnaryCall(HelloRequest request, ServerCallContext context)
    {
        return Task.FromResult(new HelloRequest { Message = $"Hello {request.Name}" });
    }
}
```

<span data-ttu-id="ceb2c-123">Реализация службы регистрируется в приложении.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-123">The service implementation is registered with the app.</span></span> <span data-ttu-id="ceb2c-124">Если служба размещена ASP.NET Core gRPC, ее следует добавить в конвейер маршрутизации с помощью метода `MapGrpcService`.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-124">If the service is hosted by ASP.NET Core gRPC, it should be added to the routing pipeline with the `MapGrpcService` method.</span></span>

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapGrpcService<GreeterService>();
});
```

<span data-ttu-id="ceb2c-125">Дополнительные сведения см. в разделе <xref:grpc/aspnetcore>.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-125">See <xref:grpc/aspnetcore> for more information.</span></span>

## <a name="implement-grpc-methods"></a><span data-ttu-id="ceb2c-126">Реализация методов gRPC</span><span class="sxs-lookup"><span data-stu-id="ceb2c-126">Implement gRPC methods</span></span>

<span data-ttu-id="ceb2c-127">Служба gRPC может иметь различные типы методов.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-127">A gRPC service can have different types of methods.</span></span> <span data-ttu-id="ceb2c-128">Способ отправки и получения сообщений службой зависит от типа определенного метода.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-128">How messages are sent and received by a service depends on the type of method defined.</span></span> <span data-ttu-id="ceb2c-129">Типы методов gRPC:</span><span class="sxs-lookup"><span data-stu-id="ceb2c-129">The gRPC method types are:</span></span>

* <span data-ttu-id="ceb2c-130">Унарный</span><span class="sxs-lookup"><span data-stu-id="ceb2c-130">Unary</span></span>
* <span data-ttu-id="ceb2c-131">Потоковая передача сервера</span><span class="sxs-lookup"><span data-stu-id="ceb2c-131">Server streaming</span></span>
* <span data-ttu-id="ceb2c-132">Потоковая передача клиента</span><span class="sxs-lookup"><span data-stu-id="ceb2c-132">Client streaming</span></span>
* <span data-ttu-id="ceb2c-133">Двунаправленная потоковая передача</span><span class="sxs-lookup"><span data-stu-id="ceb2c-133">Bi-directional streaming</span></span>

<span data-ttu-id="ceb2c-134">Потоковые вызовы указываются с помощью ключевого слова `stream` в файле *.proto*.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-134">Streaming calls are specified with the `stream` keyword in the *.proto* file.</span></span> <span data-ttu-id="ceb2c-135">`stream` можно поместить в сообщение запроса вызова, ответное сообщение или и в то и в другое.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-135">`stream` can be placed on a call's request message, response message, or both.</span></span>

```protobuf
syntax = "proto3";

service ExampleService {
  // Unary
  rpc UnaryCall (ExampleRequest) returns (ExampleResponse);

  // Server streaming
  rpc StreamingFromServer (ExampleRequest) returns (stream ExampleResponse);

  // Client streaming
  rpc StreamingFromClient (stream ExampleRequest) returns (ExampleResponse);

  // Bi-directional streaming
  rpc StreamingBothWays (stream ExampleRequest) returns (stream ExampleResponse);
}
```

<span data-ttu-id="ceb2c-136">Каждый тип вызова имеет разную сигнатуру метода.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-136">Each call type has a different method signature.</span></span> <span data-ttu-id="ceb2c-137">Переопределение созданных методов из абстрактного базового типа службы в конкретной реализации гарантирует, что используются правильные аргументы и возвращаемый тип.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-137">Overriding generated methods from the abstract base service type in a concrete implementation ensures the correct arguments and return type are used.</span></span>

### <a name="unary-method"></a><span data-ttu-id="ceb2c-138">Унарный метод</span><span class="sxs-lookup"><span data-stu-id="ceb2c-138">Unary method</span></span>

<span data-ttu-id="ceb2c-139">Унарный метод получает сообщение запроса в качестве параметра и возвращает ответ.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-139">A unary method gets the request message as a parameter, and returns the response.</span></span> <span data-ttu-id="ceb2c-140">При возвращении ответа будет выполнен унарный вызов.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-140">A unary call is complete when the response is returned.</span></span>

```csharp
public override Task<ExampleResponse> UnaryCall(ExampleRequest request,
    ServerCallContext context)
{
    var response = new ExampleResponse();
    return Task.FromResult(response);
}
```

<span data-ttu-id="ceb2c-141">Унарные вызовы больше всего похожи на [действия на контроллерах веб-API](xref:web-api/index).</span><span class="sxs-lookup"><span data-stu-id="ceb2c-141">Unary calls are the most similar to [actions on web API controllers](xref:web-api/index).</span></span> <span data-ttu-id="ceb2c-142">Одно важное отличие от gRPC-методов заключается в том, что методы gRPC не могут привязывать части запроса к разным аргументам метода.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-142">One important difference gRPC methods have from actions is gRPC methods are not able to bind parts of a request to different method arguments.</span></span> <span data-ttu-id="ceb2c-143">Методы gRPC всегда имеют один аргумент message для данных входящего запроса.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-143">gRPC methods always have one message argument for the incoming request data.</span></span> <span data-ttu-id="ceb2c-144">Несколько значений по-прежнему можно отправить в службу gRPC, сделав их полями в сообщении запроса.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-144">Multiple values can still be sent to a gRPC service by making them fields on the request message:</span></span>

```protobuf
message ExampleRequest {
    int pageIndex = 1;
    int pageSize = 2;
    bool isDescending = 3;
}
```

### <a name="server-streaming-method"></a><span data-ttu-id="ceb2c-145">Метод потоковой передачи сервера</span><span class="sxs-lookup"><span data-stu-id="ceb2c-145">Server streaming method</span></span>

<span data-ttu-id="ceb2c-146">Метод потоковой передачи сервера получает сообщение запроса в виде параметра.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-146">A server streaming method gets the request message as a parameter.</span></span> <span data-ttu-id="ceb2c-147">Так как несколько сообщений можно передать обратно вызывающему объекту, для отправки ответных сообщений используется `responseStream.WriteAsync`.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-147">Because multiple messages can be streamed back to the caller, `responseStream.WriteAsync` is used to send response messages.</span></span> <span data-ttu-id="ceb2c-148">Вызов потоковой передачи сервера завершается, когда метод возвращает результат.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-148">A server streaming call is complete when the method returns.</span></span>

```csharp
public override async Task StreamingFromServer(ExampleRequest request,
    IServerStreamWriter<ExampleResponse> responseStream, ServerCallContext context)
{
    for (var i = 0; i < 5; i++)
    {
        await responseStream.WriteAsync(new ExampleResponse());
        await Task.Delay(TimeSpan.FromSeconds(1));
    }
}
```

<span data-ttu-id="ceb2c-149">Клиент не может отправить дополнительные сообщения или данные после запуска метода потоковой передачи сервера.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-149">The client has no way to send additional messages or data once the server streaming method has started.</span></span> <span data-ttu-id="ceb2c-150">Некоторые методы потоковой передачи предназначены для непрерывного выполнения.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-150">Some streaming methods are designed to run forever.</span></span> <span data-ttu-id="ceb2c-151">Для методов непрерывной потоковой передачи клиент может отменить вызов, если он больше не нужен.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-151">For continuous streaming methods, a client can cancel the call when it's no longer needed.</span></span> <span data-ttu-id="ceb2c-152">При отмене клиент отправляет сигнал на сервер и вызывается [ServerCallContext.CancellationToken](xref:System.Threading.CancellationToken).</span><span class="sxs-lookup"><span data-stu-id="ceb2c-152">When cancellation happens the client sends a signal to the server and the [ServerCallContext.CancellationToken](xref:System.Threading.CancellationToken) is raised.</span></span> <span data-ttu-id="ceb2c-153">Токен `CancellationToken` должен использоваться на сервере с асинхронными методами, чтобы:</span><span class="sxs-lookup"><span data-stu-id="ceb2c-153">The `CancellationToken` token should be used on the server with async methods so that:</span></span>

* <span data-ttu-id="ceb2c-154">любая асинхронная работа отменялась вместе с вызовом потоковой передачи;</span><span class="sxs-lookup"><span data-stu-id="ceb2c-154">Any asynchronous work is canceled together with the streaming call.</span></span>
* <span data-ttu-id="ceb2c-155">метод быстро завершал свою работу.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-155">The method exits quickly.</span></span>

```csharp
public override async Task StreamingFromServer(ExampleRequest request,
    IServerStreamWriter<ExampleResponse> responseStream, ServerCallContext context)
{
    while (!context.CancellationToken.IsCancellationRequested)
    {
        await responseStream.WriteAsync(new ExampleResponse());
        await Task.Delay(TimeSpan.FromSeconds(1), context.CancellationToken);
    }
}
```

### <a name="client-streaming-method"></a><span data-ttu-id="ceb2c-156">Метод потоковой передачи клиента</span><span class="sxs-lookup"><span data-stu-id="ceb2c-156">Client streaming method</span></span>

<span data-ttu-id="ceb2c-157">Метод потоковой передачи клиента запускается *без* метода, получающего сообщение.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-157">A client streaming method starts *without* the method receiving a message.</span></span> <span data-ttu-id="ceb2c-158">Параметр `requestStream` используется для чтения сообщений от клиента.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-158">The `requestStream` parameter is used to read messages from the client.</span></span> <span data-ttu-id="ceb2c-159">Метод потоковой передачи клиента завершается при возвращении ответного сообщения.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-159">A client streaming call is complete when a response message is returned:</span></span>

```csharp
public override async Task<ExampleResponse> StreamingFromClient(
    IAsyncStreamReader<ExampleRequest> requestStream, ServerCallContext context)
{
    while (await requestStream.MoveNext())
    {
        var message = requestStream.Current;
        // ...
    }
    return new ExampleResponse();
}
```

<span data-ttu-id="ceb2c-160">Если используется C# 8 или более поздней версии, для чтения сообщений можно использовать синтаксис `await foreach`.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-160">When using C# 8 or later, the `await foreach` syntax can be used to read messages.</span></span> <span data-ttu-id="ceb2c-161">Метод расширения `IAsyncStreamReader<T>.ReadAllAsync()` считывает все сообщения из потока запросов.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-161">The `IAsyncStreamReader<T>.ReadAllAsync()` extension method reads all messages from the request stream:</span></span>

```csharp
public override async Task<ExampleResponse> StreamingFromClient(
    IAsyncStreamReader<ExampleRequest> requestStream, ServerCallContext context)
{
    await foreach (var message in requestStream.ReadAllAsync())
    {
        // ...
    }
    return new ExampleResponse();
}
```

### <a name="bi-directional-streaming-method"></a><span data-ttu-id="ceb2c-162">Метод двунаправленной потоковой передачи</span><span class="sxs-lookup"><span data-stu-id="ceb2c-162">Bi-directional streaming method</span></span>

<span data-ttu-id="ceb2c-163">Метод двунаправленной потоковой передачи запускается *без* метода, получающего сообщение.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-163">A bi-directional streaming method starts *without* the method receiving a message.</span></span> <span data-ttu-id="ceb2c-164">Параметр `requestStream` используется для чтения сообщений от клиента.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-164">The `requestStream` parameter is used to read messages from the client.</span></span> <span data-ttu-id="ceb2c-165">Клиент может выбрать отправку сообщений с помощью `responseStream.WriteAsync`.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-165">The method can choose to send messages with `responseStream.WriteAsync`.</span></span> <span data-ttu-id="ceb2c-166">Метод двунаправленной потоковой передачи завершается, когда метод возвращает:</span><span class="sxs-lookup"><span data-stu-id="ceb2c-166">A bi-directional streaming call is complete when the the method returns:</span></span>

```csharp
public override async Task StreamingBothWays(IAsyncStreamReader<ExampleRequest> requestStream,
    IServerStreamWriter<ExampleResponse> responseStream, ServerCallContext context)
{
    await foreach (var message in requestStream.ReadAllAsync())
    {
        await responseStream.WriteAsync(new ExampleResponse());
    }
}
```

<span data-ttu-id="ceb2c-167">Предыдущий код:</span><span class="sxs-lookup"><span data-stu-id="ceb2c-167">The preceding code:</span></span>

* <span data-ttu-id="ceb2c-168">Отправляет ответ для каждого запроса.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-168">Sends a response for each request.</span></span>
* <span data-ttu-id="ceb2c-169">Является базовым сценарием использования двунаправленной потоковой передачи.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-169">Is a basic usage of bi-directional streaming.</span></span>

<span data-ttu-id="ceb2c-170">Существует возможность поддержки более сложных сценариев, таких как чтение запросов и отправка ответов одновременно.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-170">It is possible to support more complex scenarios, such as reading requests and sending responses simultaneously:</span></span>

```csharp
public override async Task StreamingBothWays(IAsyncStreamReader<ExampleRequest> requestStream,
    IServerStreamWriter<ExampleResponse> responseStream, ServerCallContext context)
{
    // Read requests in a background task.
    var readTask = Task.Run(async () =>
    {
        await foreach (var message in requestStream.ReadAllAsync())
        {
            // Process request.
        }
    });
    
    // Send responses until the client signals that it is complete.
    while (!readTask.IsCompleted)
    {
        await responseStream.WriteAsync(new ExampleResponse());
        await Task.Delay(TimeSpan.FromSeconds(1), context.CancellationToken);
    }
}
```

<span data-ttu-id="ceb2c-171">В методе двунаправленной потоковой передачи клиент и служба могут обмениваться сообщениями в любое время.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-171">In a bi-directional streaming method, the client and service can send messages to each other at any time.</span></span> <span data-ttu-id="ceb2c-172">Оптимальная реализация двунаправленного метода зависит от требований.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-172">The best implementation of a bi-directional method varies depending upon requirements.</span></span>

## <a name="access-grpc-request-headers"></a><span data-ttu-id="ceb2c-173">Доступ к заголовкам запроса gRPC</span><span class="sxs-lookup"><span data-stu-id="ceb2c-173">Access gRPC request headers</span></span>

<span data-ttu-id="ceb2c-174">Сообщение запроса — не единственный способ отправки данных клиентом в службу gRPC.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-174">A request message is not the only way for a client to send data to a gRPC service.</span></span> <span data-ttu-id="ceb2c-175">Значения заголовков доступны в службе с помощью `ServerCallContext.RequestHeaders`.</span><span class="sxs-lookup"><span data-stu-id="ceb2c-175">Header values are available in a service using `ServerCallContext.RequestHeaders`.</span></span>

```csharp
public override Task<ExampleResponse> UnaryCall(ExampleRequest request, ServerCallContext context)
{
    var userAgent = context.RequestHeaders.GetValue("user-agent");
    // ...

    return Task.FromResult(new ExampleResponse());
}
```

## <a name="additional-resources"></a><span data-ttu-id="ceb2c-176">Дополнительные ресурсы</span><span class="sxs-lookup"><span data-stu-id="ceb2c-176">Additional resources</span></span>

* <xref:grpc/basics>
* <xref:grpc/client>