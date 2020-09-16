---
title: Лучшие методики повышения производительности gRPC
author: jamesnk
description: Ознакомьтесь с рекомендациями по созданию высокопроизводительных служб gRPC.
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 08/23/2020
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
uid: grpc/performance
ms.openlocfilehash: a0a1a6901e07fb0074ca403870378f267d3d4403
ms.sourcegitcommit: c9b03d8a6a4dcc59e4aacb30a691f349235a74c8
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/02/2020
ms.locfileid: "89379449"
---
# <a name="performance-best-practices-with-grpc"></a><span data-ttu-id="8736c-103">Лучшие методики повышения производительности gRPC</span><span class="sxs-lookup"><span data-stu-id="8736c-103">Performance best practices with gRPC</span></span>

<span data-ttu-id="8736c-104">Автор: [Джеймс Ньютон-Кинг (James Newton-King)](https://twitter.com/jamesnk)</span><span class="sxs-lookup"><span data-stu-id="8736c-104">By [James Newton-King](https://twitter.com/jamesnk)</span></span>

<span data-ttu-id="8736c-105">Система gRPC предназначена для создания высокопроизводительных служб.</span><span class="sxs-lookup"><span data-stu-id="8736c-105">gRPC is designed for high-performance services.</span></span> <span data-ttu-id="8736c-106">В этом документе описывается, как обеспечить максимальную производительность gRPC.</span><span class="sxs-lookup"><span data-stu-id="8736c-106">This document explains how to get the best performance possible from gRPC.</span></span>

## <a name="reuse-grpc-channels"></a><span data-ttu-id="8736c-107">Повторное использование каналов gRPC</span><span class="sxs-lookup"><span data-stu-id="8736c-107">Reuse gRPC channels</span></span>

<span data-ttu-id="8736c-108">При выполнении вызовов gRPC канал gRPC следует использовать повторно.</span><span class="sxs-lookup"><span data-stu-id="8736c-108">A gRPC channel should be reused when making gRPC calls.</span></span> <span data-ttu-id="8736c-109">Повторное использование канала позволяет мультиплексировать вызовы через существующее соединение HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="8736c-109">Reusing a channel allows calls to be multiplexed through an existing HTTP/2 connection.</span></span>

<span data-ttu-id="8736c-110">Если для каждого вызова gRPC создается новый канал, то время, необходимое для его выполнения, может значительно возрасти.</span><span class="sxs-lookup"><span data-stu-id="8736c-110">If a new channel is created for each gRPC call then the amount of time it takes to complete can increase significantly.</span></span> <span data-ttu-id="8736c-111">При каждом вызове потребуется несколько круговых путей по сети между клиентом и сервером для создания нового соединения HTTP/2:</span><span class="sxs-lookup"><span data-stu-id="8736c-111">Each call will require multiple network round-trips between the client and the server to create a new HTTP/2 connection:</span></span>

1. <span data-ttu-id="8736c-112">открытие сокета;</span><span class="sxs-lookup"><span data-stu-id="8736c-112">Opening a socket</span></span>
2. <span data-ttu-id="8736c-113">установка TCP-соединения;</span><span class="sxs-lookup"><span data-stu-id="8736c-113">Establishing TCP connection</span></span>
3. <span data-ttu-id="8736c-114">согласование TLS;</span><span class="sxs-lookup"><span data-stu-id="8736c-114">Negotiating TLS</span></span>
4. <span data-ttu-id="8736c-115">инициация соединения HTTP/2;</span><span class="sxs-lookup"><span data-stu-id="8736c-115">Starting HTTP/2 connection</span></span>
5. <span data-ttu-id="8736c-116">выполнение вызова gRPC.</span><span class="sxs-lookup"><span data-stu-id="8736c-116">Making the gRPC call</span></span>

<span data-ttu-id="8736c-117">Каналы можно безопасно использовать для нескольких вызовов gRPC.</span><span class="sxs-lookup"><span data-stu-id="8736c-117">Channels are safe to share and reuse between gRPC calls:</span></span>

* <span data-ttu-id="8736c-118">Клиенты gRPC создаются с помощью каналов.</span><span class="sxs-lookup"><span data-stu-id="8736c-118">gRPC clients are created with channels.</span></span> <span data-ttu-id="8736c-119">Клиенты gRPC являются облегченными объектами и не нуждаются в кэшировании или повторном использовании.</span><span class="sxs-lookup"><span data-stu-id="8736c-119">gRPC clients are lightweight objects and don't need to be cached or reused.</span></span>
* <span data-ttu-id="8736c-120">Из одного канала можно создать несколько клиентов gRPC, включая различные типы клиентов.</span><span class="sxs-lookup"><span data-stu-id="8736c-120">Multiple gRPC clients can be created from a channel, including different types of clients.</span></span>
* <span data-ttu-id="8736c-121">Канал и клиенты, созданные из канала, могут безопасно использоваться несколькими потоками.</span><span class="sxs-lookup"><span data-stu-id="8736c-121">A channel and clients created from the channel can safely be used by multiple threads.</span></span>
* <span data-ttu-id="8736c-122">Клиенты, созданные из канала, могут выполнять несколько одновременных вызовов.</span><span class="sxs-lookup"><span data-stu-id="8736c-122">Clients created from the channel can make multiple simultaneous calls.</span></span>

<span data-ttu-id="8736c-123">Фабрика клиента gRPC предлагает централизованный способ настройки каналов.</span><span class="sxs-lookup"><span data-stu-id="8736c-123">gRPC client factory offers a centralized way to configure channels.</span></span> <span data-ttu-id="8736c-124">Она автоматически повторно использует базовые каналы.</span><span class="sxs-lookup"><span data-stu-id="8736c-124">It automatically reuses underlying channels.</span></span> <span data-ttu-id="8736c-125">Для получения дополнительной информации см. <xref:grpc/clientfactory>.</span><span class="sxs-lookup"><span data-stu-id="8736c-125">For more information, see <xref:grpc/clientfactory>.</span></span>

## <a name="connection-concurrency"></a><span data-ttu-id="8736c-126">Параллелизм соединений</span><span class="sxs-lookup"><span data-stu-id="8736c-126">Connection concurrency</span></span>

<span data-ttu-id="8736c-127">Обычно существует ограничение на [количество параллельных потоков (активных HTTP-запросов)](https://http2.github.io/http2-spec/#rfc.section.5.1.2) для одного соединения по HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="8736c-127">HTTP/2 connections typically have a limit on the number of [maximum concurrent streams (active HTTP requests)](https://http2.github.io/http2-spec/#rfc.section.5.1.2) on a connection at one time.</span></span> <span data-ttu-id="8736c-128">По умолчанию на большинстве серверов оно составляет 100 параллельных потоков.</span><span class="sxs-lookup"><span data-stu-id="8736c-128">By default, most servers set this limit to 100 concurrent streams.</span></span>

<span data-ttu-id="8736c-129">Канал gRPC использует одно соединение HTTP/2, параллельные вызовы по которому мультиплексируются.</span><span class="sxs-lookup"><span data-stu-id="8736c-129">A gRPC channel uses a single HTTP/2 connection, and concurrent calls are multiplexed on that connection.</span></span> <span data-ttu-id="8736c-130">Когда число активных вызовов достигает предельного числа потоков для соединения, дополнительные вызовы помещаются в очередь в клиенте.</span><span class="sxs-lookup"><span data-stu-id="8736c-130">When the number of active calls reaches the connection stream limit, additional calls are queued in the client.</span></span> <span data-ttu-id="8736c-131">Вызовы в очереди ожидают завершения активных вызовов.</span><span class="sxs-lookup"><span data-stu-id="8736c-131">Queued calls wait for active calls to complete before they are sent.</span></span> <span data-ttu-id="8736c-132">Приложения с высокой нагрузкой или длительными потоковыми вызовами gRPC могут испытывать проблемы с производительностью, вызванные помещением вызовов в очередь из-за этого ограничения.</span><span class="sxs-lookup"><span data-stu-id="8736c-132">Applications with high load, or long running streaming gRPC calls, could see performance issues caused by calls queuing because of this limit.</span></span>

::: moniker range=">= aspnetcore-5.0"

<span data-ttu-id="8736c-133">В .NET 5 появилось свойство `SocketsHttpHandler.EnableMultipleHttp2Connections`.</span><span class="sxs-lookup"><span data-stu-id="8736c-133">.NET 5 introduces the `SocketsHttpHandler.EnableMultipleHttp2Connections` property.</span></span> <span data-ttu-id="8736c-134">Если ему присвоено значение `true`, то при достижении предельного числа параллельных потоков канал создает дополнительные соединения HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="8736c-134">When set to `true`, additional HTTP/2 connections are created by a channel when the concurrent stream limit is reached.</span></span> <span data-ttu-id="8736c-135">При создании `GrpcChannel` его внутренний обработчик `SocketsHttpHandler` автоматически настраивается так, чтобы создавались дополнительные соединения HTTP/2.</span><span class="sxs-lookup"><span data-stu-id="8736c-135">When a `GrpcChannel` is created its internal `SocketsHttpHandler` is automatically configured to create additional HTTP/2 connections.</span></span> <span data-ttu-id="8736c-136">Если приложение настраивает собственный обработчик, рекомендуется присвоить свойству `EnableMultipleHttp2Connections` значение `true`.</span><span class="sxs-lookup"><span data-stu-id="8736c-136">If an app configures its own handler, consider setting `EnableMultipleHttp2Connections` to `true`:</span></span>

```csharp
var channel = GrpcChannel.ForAddress("https://localhost", new GrpcChannelOptions
{
    HttpHandler = new SocketsHttpHandler
    {
        EnableMultipleHttp2Connections = true,

        // ...configure other handler settings
    }
});
```

::: moniker-end

<span data-ttu-id="8736c-137">Для приложений .NET Core 3.1 существует несколько обходных путей.</span><span class="sxs-lookup"><span data-stu-id="8736c-137">There are a couple of workarounds for .NET Core 3.1 apps:</span></span>

* <span data-ttu-id="8736c-138">Создайте отдельные каналы gRPC для частей приложения с высокой нагрузкой.</span><span class="sxs-lookup"><span data-stu-id="8736c-138">Create separate gRPC channels for areas of the app with high load.</span></span> <span data-ttu-id="8736c-139">Например, служба gRPC `Logger` может испытывать высокую нагрузку.</span><span class="sxs-lookup"><span data-stu-id="8736c-139">For example, the `Logger` gRPC service might have a high load.</span></span> <span data-ttu-id="8736c-140">Используйте отдельный канал для создания `LoggerClient` в приложении.</span><span class="sxs-lookup"><span data-stu-id="8736c-140">Use a separate channel to create the `LoggerClient` in the app.</span></span>
* <span data-ttu-id="8736c-141">Используйте пул каналов gRPC, например создайте их список.</span><span class="sxs-lookup"><span data-stu-id="8736c-141">Use a pool of gRPC channels, for example,  create a list of gRPC channels.</span></span> <span data-ttu-id="8736c-142">`Random` используется для выбора канала из списка каждый раз, когда требуется канал gRPC.</span><span class="sxs-lookup"><span data-stu-id="8736c-142">`Random` is used to pick a channel from the list each time a gRPC channel is needed.</span></span> <span data-ttu-id="8736c-143">Использование `Random` позволяет случайным образом распределять вызовы между несколькими соединениями.</span><span class="sxs-lookup"><span data-stu-id="8736c-143">Using `Random` randomly distributes calls over multiple connections.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="8736c-144">Еще один способ решить эту проблему — увеличить максимальное число параллельных потоков на сервере.</span><span class="sxs-lookup"><span data-stu-id="8736c-144">Increasing the maximum concurrent stream limit on the server is another way to solve this problem.</span></span> <span data-ttu-id="8736c-145">В Kestrel оно настраивается с помощью <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.MaxStreamsPerConnection>.</span><span class="sxs-lookup"><span data-stu-id="8736c-145">In Kestrel this is configured with <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Http2Limits.MaxStreamsPerConnection>.</span></span>
>
> <span data-ttu-id="8736c-146">Увеличивать максимальное число параллельных потоков не рекомендуется.</span><span class="sxs-lookup"><span data-stu-id="8736c-146">Increasing the maximum concurrent stream limit is not recommended.</span></span> <span data-ttu-id="8736c-147">Слишком большое число потоков в одном соединении HTTP/2 вызывает новые проблемы с производительностью.</span><span class="sxs-lookup"><span data-stu-id="8736c-147">Too many streams on a single HTTP/2 connection introduces new performance issues:</span></span>
>
> * <span data-ttu-id="8736c-148">Возникает состязание между потоками, пытающимися выполнить запись через соединение.</span><span class="sxs-lookup"><span data-stu-id="8736c-148">Thread contention between streams trying to write to the connection.</span></span>
> * <span data-ttu-id="8736c-149">Потеря пакетов, передаваемых через соединение, приводит к блокировке всех вызовов на уровне TCP.</span><span class="sxs-lookup"><span data-stu-id="8736c-149">Connection packet loss causes all calls to be blocked at the TCP layer.</span></span>

## <a name="load-balancing"></a><span data-ttu-id="8736c-150">Балансировка нагрузки</span><span class="sxs-lookup"><span data-stu-id="8736c-150">Load balancing</span></span>

<span data-ttu-id="8736c-151">Некоторые подсистемы балансировки нагрузки не могут эффективно работать с gRPC.</span><span class="sxs-lookup"><span data-stu-id="8736c-151">Some load balancers don't work effectively with gRPC.</span></span> <span data-ttu-id="8736c-152">Подсистемы балансировки нагрузки L4 (транспортировка) действуют на уровне соединения путем распределения TCP-подключений между конечными точками.</span><span class="sxs-lookup"><span data-stu-id="8736c-152">L4 (transport) load balancers operate at a connection level, by distributing TCP connections across endpoints.</span></span> <span data-ttu-id="8736c-153">Такой способ хорошо подходит для вызовов API балансировки нагрузки, выполняемых с помощью HTTP/1.1.</span><span class="sxs-lookup"><span data-stu-id="8736c-153">This approach works well for loading balancing API calls made with HTTP/1.1.</span></span> <span data-ttu-id="8736c-154">Одновременные вызовы, выполняемые с помощью HTTP/1.1, отправляются по разным соединениям, что позволяет распределять нагрузку вызовов между конечными точками.</span><span class="sxs-lookup"><span data-stu-id="8736c-154">Concurrent calls made with HTTP/1.1 are sent on different connections, allowing calls to be load balanced across endpoints.</span></span>

<span data-ttu-id="8736c-155">Так как подсистемы балансировки нагрузки L4 работают на уровне соединения, они плохо работают с gRPC.</span><span class="sxs-lookup"><span data-stu-id="8736c-155">Because L4 load balancers operate at a connection level, they don't work well with gRPC.</span></span> <span data-ttu-id="8736c-156">gRPC использует HTTP/2, что приводит к мультиплексированию нескольких вызовов в одном TCP-подключении.</span><span class="sxs-lookup"><span data-stu-id="8736c-156">gRPC uses HTTP/2, which multiplexes multiple calls on a single TCP connection.</span></span> <span data-ttu-id="8736c-157">Все вызовы gRPC через это подключение поступают в одну конечную точку.</span><span class="sxs-lookup"><span data-stu-id="8736c-157">All gRPC calls over that connection go to one endpoint.</span></span>

<span data-ttu-id="8736c-158">Существует два варианта эффективного распределения нагрузки gRPC.</span><span class="sxs-lookup"><span data-stu-id="8736c-158">There are two options to effectively load balance gRPC:</span></span>

* <span data-ttu-id="8736c-159">Балансировка нагрузки на стороне клиента</span><span class="sxs-lookup"><span data-stu-id="8736c-159">Client-side load balancing</span></span>
* <span data-ttu-id="8736c-160">Балансировка нагрузки на прокси-сервере L7 (приложения)</span><span class="sxs-lookup"><span data-stu-id="8736c-160">L7 (application) proxy load balancing</span></span>

> [!NOTE]
> <span data-ttu-id="8736c-161">Между конечными точками может распределяться только нагрузка вызовов gRPC.</span><span class="sxs-lookup"><span data-stu-id="8736c-161">Only gRPC calls can be load balanced between endpoints.</span></span> <span data-ttu-id="8736c-162">После установки вызова gRPC потоковой передачи все сообщения, отправленные в этом потоке, поступают в одну конечную точку.</span><span class="sxs-lookup"><span data-stu-id="8736c-162">Once a streaming gRPC call is established, all messages sent over the stream go to one endpoint.</span></span>

### <a name="client-side-load-balancing"></a><span data-ttu-id="8736c-163">Балансировка нагрузки на стороне клиента</span><span class="sxs-lookup"><span data-stu-id="8736c-163">Client-side load balancing</span></span>

<span data-ttu-id="8736c-164">При использовании балансировки нагрузки на стороне клиента клиент осведомлен о конечных точках.</span><span class="sxs-lookup"><span data-stu-id="8736c-164">With client-side load balancing, the client knows about endpoints.</span></span> <span data-ttu-id="8736c-165">При каждом вызове gRPC клиент выбирает другую конечную точку для отправки вызова.</span><span class="sxs-lookup"><span data-stu-id="8736c-165">For each gRPC call it selects a different endpoint to send the call to.</span></span> <span data-ttu-id="8736c-166">Балансировка нагрузки на стороне клиента прекрасно подходит в случаях, когда задержка играет важную роль.</span><span class="sxs-lookup"><span data-stu-id="8736c-166">Client-side load balancing is a good choice when latency is important.</span></span> <span data-ttu-id="8736c-167">Между клиентом и службой нет прокси-сервера, поэтому вызов отправляется в службу напрямую.</span><span class="sxs-lookup"><span data-stu-id="8736c-167">There is no proxy between the client and the service so the call is sent to the service directly.</span></span> <span data-ttu-id="8736c-168">Недостаток балансировки нагрузки на стороне клиента заключается в том, что каждый клиент должен отслеживать доступные конечные точки, которые ему нужно использовать.</span><span class="sxs-lookup"><span data-stu-id="8736c-168">The downside to client-side load balancing is that each client must keep track of available endpoints it should use.</span></span>

<span data-ttu-id="8736c-169">Балансировка нагрузки клиента с резервированием — это метод, где состояние балансировки нагрузки хранится в центральном расположении.</span><span class="sxs-lookup"><span data-stu-id="8736c-169">Lookaside client load balancing is a technique where load balancing state is stored in a central location.</span></span> <span data-ttu-id="8736c-170">Клиенты периодически запрашивают в центральном расположении сведения, которые используются при принятии решений для балансировки нагрузки.</span><span class="sxs-lookup"><span data-stu-id="8736c-170">Clients periodically query the central location for information to use when making load balancing decisions.</span></span>

<span data-ttu-id="8736c-171">`Grpc.Net.Client` пока не поддерживает балансировку нагрузки на стороне клиента.</span><span class="sxs-lookup"><span data-stu-id="8736c-171">`Grpc.Net.Client` currently doesn't support client-side load balancing.</span></span> <span data-ttu-id="8736c-172">Если в .NET требуется балансировка нагрузки на стороне клиента, рекомендуется использовать [Grpc.Core](https://www.nuget.org/packages/Grpc.Core).</span><span class="sxs-lookup"><span data-stu-id="8736c-172">[Grpc.Core](https://www.nuget.org/packages/Grpc.Core) is a good choice if client-side load balancing is required in .NET.</span></span>

### <a name="proxy-load-balancing"></a><span data-ttu-id="8736c-173">Балансировка нагрузки на прокси-сервере</span><span class="sxs-lookup"><span data-stu-id="8736c-173">Proxy load balancing</span></span>

<span data-ttu-id="8736c-174">Прокси-сервер L7 (приложения) работает на более высоком уровне, чем прокси-сервер L4 (транспортировка).</span><span class="sxs-lookup"><span data-stu-id="8736c-174">An L7 (application) proxy works at a higher level than an L4 (transport) proxy.</span></span> <span data-ttu-id="8736c-175">Прокси-серверы L7 поддерживают HTTP/2 и могут распределять вызовы gRPC, мультиплексированные на прокси-сервер через одно подключение HTTP/2, между несколькими конечными точками.</span><span class="sxs-lookup"><span data-stu-id="8736c-175">L7 proxies understand HTTP/2, and are able to distribute gRPC calls multiplexed to the proxy on one HTTP/2 connection across multiple endpoints.</span></span> <span data-ttu-id="8736c-176">Использование прокси-сервера проще, чем балансировка нагрузки на стороне клиента, но может увеличить задержку при вызовах gRPC.</span><span class="sxs-lookup"><span data-stu-id="8736c-176">Using a proxy is simpler than client-side load balancing, but can add extra latency to gRPC calls.</span></span>

<span data-ttu-id="8736c-177">Доступно множество прокси-серверов L7.</span><span class="sxs-lookup"><span data-stu-id="8736c-177">There are many L7 proxies available.</span></span> <span data-ttu-id="8736c-178">Вот некоторые варианты:</span><span class="sxs-lookup"><span data-stu-id="8736c-178">Some options are:</span></span>

* <span data-ttu-id="8736c-179">[Envoy](https://www.envoyproxy.io/) — популярный прокси-сервер с открытым кодом;</span><span class="sxs-lookup"><span data-stu-id="8736c-179">[Envoy](https://www.envoyproxy.io/) - A popular open source proxy.</span></span>
* <span data-ttu-id="8736c-180">[Linkerd](https://linkerd.io/) — сетка Service Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="8736c-180">[Linkerd](https://linkerd.io/) - Service mesh for Kubernetes.</span></span>
* <span data-ttu-id="8736c-181">[YARP: обратный прокси-сервер](https://microsoft.github.io/reverse-proxy/) — предварительная версия прокси-сервера с открытым кодом, написанным на .NET.</span><span class="sxs-lookup"><span data-stu-id="8736c-181">[YARP: A Reverse Proxy](https://microsoft.github.io/reverse-proxy/) - A preview open source proxy written in .NET.</span></span>

::: moniker range=">= aspnetcore-5.0"

## <a name="keep-alive-pings"></a><span data-ttu-id="8736c-182">Пакеты проверки активности</span><span class="sxs-lookup"><span data-stu-id="8736c-182">Keep alive pings</span></span>

<span data-ttu-id="8736c-183">Пакеты проверки активности могут использоваться для поддержания соединений HTTP/2 в активном состоянии в периоды бездействия.</span><span class="sxs-lookup"><span data-stu-id="8736c-183">Keep alive pings can be used to keep HTTP/2 connections alive during periods of inactivity.</span></span> <span data-ttu-id="8736c-184">Готовность соединения HTTP/2 на момент возобновления работы приложения позволяет быстро выполнять первые вызовы gRPC без задержки, вызванной повторным установлением соединения.</span><span class="sxs-lookup"><span data-stu-id="8736c-184">Having an existing HTTP/2 connection ready when an app resumes activity allows for the initial gRPC calls to be made quickly, without a delay caused by the connection being reestablished.</span></span>

<span data-ttu-id="8736c-185">Пакеты проверки активности настраиваются в <xref:System.Net.Http.SocketsHttpHandler>.</span><span class="sxs-lookup"><span data-stu-id="8736c-185">Keep alive pings are configured on <xref:System.Net.Http.SocketsHttpHandler>:</span></span>

```csharp
var handler = new SocketsHttpHandler
{
    PooledConnectionIdleTimeout = Timeout.InfiniteTimeSpan,
    KeepAlivePingDelay = TimeSpan.FromSeconds(60),
    KeepAlivePingTimeout = TimeSpan.FromSeconds(30),
    EnableMultipleHttp2Connections = true
};

var channel = GrpcChannel.ForAddress("https://localhost:5001", new GrpcChannelOptions
{
    HttpHandler = handler
});
```

<span data-ttu-id="8736c-186">Приведенный выше код настраивает канал, который отправляет пакет проверки активности на сервер каждые 60 секунд в периоды бездействия.</span><span class="sxs-lookup"><span data-stu-id="8736c-186">The preceding code configures a channel that sends a keep alive ping to the server every 60 seconds during periods of inactivity.</span></span> <span data-ttu-id="8736c-187">Проверка активности гарантирует, что сервер и все используемые прокси-серверы не закроют соединение из-за бездействия.</span><span class="sxs-lookup"><span data-stu-id="8736c-187">The ping ensures the server and any proxies in use won't close the connection because of inactivity.</span></span>

::: moniker-end

## <a name="streaming"></a><span data-ttu-id="8736c-188">Потоковая передача</span><span class="sxs-lookup"><span data-stu-id="8736c-188">Streaming</span></span>

<span data-ttu-id="8736c-189">В высокопроизводительных сценариях вместо отдельных вызовов gRPC может использоваться двунаправленная потоковая передача gRPC.</span><span class="sxs-lookup"><span data-stu-id="8736c-189">gRPC bidirectional streaming can be used to replace unary gRPC calls in high-performance scenarios.</span></span> <span data-ttu-id="8736c-190">После запуска двунаправленного потока потоковая передача сообщений в обе стороны происходит быстрее, чем отправка сообщений для нескольких отдельных вызовов gRPC.</span><span class="sxs-lookup"><span data-stu-id="8736c-190">Once a bidirectional stream has started, streaming messages back and forth is faster than sending messages with multiple unary gRPC calls.</span></span> <span data-ttu-id="8736c-191">Потоковые сообщения отправляются в виде данных существующего запроса HTTP/2, благодаря чему устраняются издержки на создание запроса HTTP/2 для каждого отдельного вызова.</span><span class="sxs-lookup"><span data-stu-id="8736c-191">Streamed messages are sent as data on an existing HTTP/2 request and eliminates the overhead of creating a new HTTP/2 request for each unary call.</span></span>

<span data-ttu-id="8736c-192">Пример службы:</span><span class="sxs-lookup"><span data-stu-id="8736c-192">Example service:</span></span>

```csharp
public override async Task SayHello(IAsyncStreamReader<HelloRequest> requestStream,
    IServerStreamWriter<HelloReply> responseStream, ServerCallContext context)
{
    await foreach (var request in requestStream.ReadAllAsync())
    {
        var helloReply = new HelloReply { Message = "Hello " + request.Name };

        await responseStream.WriteAsync(helloReply);
    }
}
```

<span data-ttu-id="8736c-193">Пример клиента:</span><span class="sxs-lookup"><span data-stu-id="8736c-193">Example client:</span></span>

```csharp
var client = new Greet.GreeterClient(channel);
using var call = client.SayHello();

Console.WriteLine("Type a name then press enter.");
while (true)
{
    var text = Console.ReadLine();

    // Send and receive messages over the stream
    await call.RequestStream.WriteAsync(new HelloRequest { Name = text });
    await call.ResponseStream.MoveNext();

    Console.WriteLine($"Greeting: {call.ResponseStream.Current.Message}");
}
```

<span data-ttu-id="8736c-194">Замена отдельных вызовов на двунаправленную потоковую передачу для повышения производительности — сложный в реализации метод, который не подходит во многих ситуациях.</span><span class="sxs-lookup"><span data-stu-id="8736c-194">Replacing unary calls with bidirectional streaming for performance reasons is an advanced technique and is not appropriate in many situations.</span></span>

<span data-ttu-id="8736c-195">Использовать потоковые вызовы рекомендуется в указанных ниже случаях.</span><span class="sxs-lookup"><span data-stu-id="8736c-195">Using streaming calls is a good choice when:</span></span>

1. <span data-ttu-id="8736c-196">Требуется высокая пропускная способность или низкая задержка.</span><span class="sxs-lookup"><span data-stu-id="8736c-196">High throughput or low latency is required.</span></span>
2. <span data-ttu-id="8736c-197">gRPC и HTTP/2 были определены как узкие места производительности.</span><span class="sxs-lookup"><span data-stu-id="8736c-197">gRPC and HTTP/2 are identified as a performance bottleneck.</span></span>
3. <span data-ttu-id="8736c-198">Рабочая роль клиента регулярно отправляет или получает сообщения с помощью службы gRPC.</span><span class="sxs-lookup"><span data-stu-id="8736c-198">A worker in the client is sending or receiving regular messages with a gRPC service.</span></span>

<span data-ttu-id="8736c-199">Обратите внимание на дополнительные сложности и ограничения, связанные с использованием потоковых вызовов вместо отдельных.</span><span class="sxs-lookup"><span data-stu-id="8736c-199">Be aware of the additional complexity and limitations of using streaming calls instead of unary:</span></span>

1. <span data-ttu-id="8736c-200">Поток может быть прерван службой или ошибкой соединения.</span><span class="sxs-lookup"><span data-stu-id="8736c-200">A stream can be interrupted by a service or connection error.</span></span> <span data-ttu-id="8736c-201">Для перезапуска потока при возникновении ошибки требуется логика.</span><span class="sxs-lookup"><span data-stu-id="8736c-201">Logic is required to restart stream if there is an error.</span></span>
2. <span data-ttu-id="8736c-202">`RequestStream.WriteAsync` небезопасно использовать в режиме многопоточности.</span><span class="sxs-lookup"><span data-stu-id="8736c-202">`RequestStream.WriteAsync` is not safe for multi-threading.</span></span> <span data-ttu-id="8736c-203">В поток за раз можно записать только одно сообщение.</span><span class="sxs-lookup"><span data-stu-id="8736c-203">Only one message can be written to a stream at a time.</span></span> <span data-ttu-id="8736c-204">Для отправки сообщений из нескольких потоков требуется очередь производителя или потребителя, например <xref:System.Threading.Channels.Channel%601>, для маршалирования сообщений.</span><span class="sxs-lookup"><span data-stu-id="8736c-204">Sending messages from multiple threads over a single stream requires a producer/consumer queue like <xref:System.Threading.Channels.Channel%601> to marshall messages.</span></span>
3. <span data-ttu-id="8736c-205">Метод потоковой передачи gRPC ограничен получением и отправкой сообщений одного типа.</span><span class="sxs-lookup"><span data-stu-id="8736c-205">A gRPC streaming method is limited to receiving one type of message and sending one type of message.</span></span> <span data-ttu-id="8736c-206">Например, `rpc StreamingCall(stream RequestMessage) returns (stream ResponseMessage)` получает `RequestMessage` и отправляет `ResponseMessage`.</span><span class="sxs-lookup"><span data-stu-id="8736c-206">For example, `rpc StreamingCall(stream RequestMessage) returns (stream ResponseMessage)` receives `RequestMessage` and sends `ResponseMessage`.</span></span> <span data-ttu-id="8736c-207">Поддержка неизвестных или условных сообщений в protobuf благодаря `Any` и `oneof` позволяет обойти это ограничение.</span><span class="sxs-lookup"><span data-stu-id="8736c-207">Protobuf's support for unknown or conditional messages using `Any` and `oneof` can work around this limitation.</span></span>