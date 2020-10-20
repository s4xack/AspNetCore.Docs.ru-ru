---
title: Использование ASP.NET Core с HTTP/2 в службах IIS
author: rick-anderson
description: Сведения об использовании возможностей HTTP/2 со службами IIS.
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 01/13/2020
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
uid: host-and-deploy/iis/protocols
ms.openlocfilehash: 1c3748eeead1b8cccdb6112150cf90fb5597b689
ms.sourcegitcommit: e519d95d17443abafba8f712ac168347b15c8b57
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/02/2020
ms.locfileid: "91654351"
---
# <a name="use-aspnet-core-with-http2-on-iis"></a><span data-ttu-id="baba7-103">Использование ASP.NET Core с HTTP/2 в службах IIS</span><span class="sxs-lookup"><span data-stu-id="baba7-103">Use ASP.NET Core with HTTP/2 on IIS</span></span>

<span data-ttu-id="baba7-104">Автор [Джастин Коталик (Justin Kotalik)](https://github.com/jkotalik)</span><span class="sxs-lookup"><span data-stu-id="baba7-104">By [Justin Kotalik](https://github.com/jkotalik)</span></span>

<span data-ttu-id="baba7-105">[HTTP/2](https://httpwg.org/specs/rfc7540.html) поддерживается в ASP.NET Core для следующих сценариев развертывания IIS:</span><span class="sxs-lookup"><span data-stu-id="baba7-105">[HTTP/2](https://httpwg.org/specs/rfc7540.html) is supported with ASP.NET Core in the following IIS deployment scenarios:</span></span>

* <span data-ttu-id="baba7-106">Windows Server 2016 или более поздней версии либо Windows 10 или более поздней версии</span><span class="sxs-lookup"><span data-stu-id="baba7-106">Windows Server 2016 or later / Windows 10 or later</span></span>
* <span data-ttu-id="baba7-107">Службы IIS 10 или более поздней версии</span><span class="sxs-lookup"><span data-stu-id="baba7-107">IIS 10 or later</span></span>
* <span data-ttu-id="baba7-108">Подключение TLS 1.2 или более поздней версии.</span><span class="sxs-lookup"><span data-stu-id="baba7-108">TLS 1.2 or later connection</span></span>
* <span data-ttu-id="baba7-109">Размещение [вне процесса](xref:host-and-deploy/iis/index#out-of-process-hosting-model). Подключения к пограничным серверам, открытых для общего доступа, выполняются по протоколу HTTP/2, а подключения к [серверу Kestrel](xref:fundamentals/servers/kestrel) через обратный прокси-сервер — по HTTP/1.1.</span><span class="sxs-lookup"><span data-stu-id="baba7-109">When [hosting out-of-process](xref:host-and-deploy/iis/index#out-of-process-hosting-model): Public-facing edge server connections use HTTP/2, but the reverse proxy connection to the [Kestrel server](xref:fundamentals/servers/kestrel) uses HTTP/1.1.</span></span>

<span data-ttu-id="baba7-110">При внутрипроцессном развертывании и установленном подключении HTTP/2 [`HttpRequest.Protocol`](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) возвращает `HTTP/2`.</span><span class="sxs-lookup"><span data-stu-id="baba7-110">For an in-process deployment when an HTTP/2 connection is established, [`HttpRequest.Protocol`](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) reports `HTTP/2`.</span></span> <span data-ttu-id="baba7-111">При внепроцессном развертывании и установленном подключении HTTP/2 [`HttpRequest.Protocol`](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) возвращает `HTTP/1.1`.</span><span class="sxs-lookup"><span data-stu-id="baba7-111">For an out-of-process deployment when an HTTP/2 connection is established, [`HttpRequest.Protocol`](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) reports `HTTP/1.1`.</span></span>

<span data-ttu-id="baba7-112">Дополнительные сведения о внутрипроцессной и внепроцессной моделях размещения см. в статье <xref:host-and-deploy/aspnet-core-module>.</span><span class="sxs-lookup"><span data-stu-id="baba7-112">For more information on the in-process and out-of-process hosting models, see <xref:host-and-deploy/aspnet-core-module>.</span></span>

<span data-ttu-id="baba7-113">Протокол HTTP/2 включен по умолчанию для соединений по HTTPS/TLS.</span><span class="sxs-lookup"><span data-stu-id="baba7-113">HTTP/2 is enabled by default for HTTPS/TLS connections.</span></span> <span data-ttu-id="baba7-114">Если не удается установить подключение HTTP/2, применяется резервный вариант HTTP/1.1.</span><span class="sxs-lookup"><span data-stu-id="baba7-114">Connections fall back to HTTP/1.1 if an HTTP/2 connection isn't established.</span></span> <span data-ttu-id="baba7-115">Дополнительные сведения о настройке HTTP/2 для развертываний IIS см. в статье [об HTTP/2 в IIS](/iis/get-started/whats-new-in-iis-10/http2-on-iis).</span><span class="sxs-lookup"><span data-stu-id="baba7-115">For more information on HTTP/2 configuration with IIS deployments, see [HTTP/2 on IIS](/iis/get-started/whats-new-in-iis-10/http2-on-iis).</span></span>

## <a name="advanced-http2-features-to-support-grpc"></a><span data-ttu-id="baba7-116">Расширенные возможности HTTP/2 для поддержки gRPC</span><span class="sxs-lookup"><span data-stu-id="baba7-116">Advanced HTTP/2 features to support gRPC</span></span>

<span data-ttu-id="baba7-117">Дополнительные возможности HTTP/2 в службах IIS поддерживают gRPC, включая трейлеры ответов и отправку кадров сброса.</span><span class="sxs-lookup"><span data-stu-id="baba7-117">Additional HTTP/2 features in IIS support gRPC, including support for response trailers and sending reset frames.</span></span>

<span data-ttu-id="baba7-118">Требования для выполнения gRPC в службах IIS</span><span class="sxs-lookup"><span data-stu-id="baba7-118">Requirements to run gRPC on IIS:</span></span>

* <span data-ttu-id="baba7-119">Внутрипроцессное размещение.</span><span class="sxs-lookup"><span data-stu-id="baba7-119">In-process hosting.</span></span>
* <span data-ttu-id="baba7-120">Сборка 20300.1000 или более поздняя ОС Windows 10.</span><span class="sxs-lookup"><span data-stu-id="baba7-120">Windows 10, OS Build 20300.1000 or later.</span></span> <span data-ttu-id="baba7-121">Могут требоваться сборки программы предварительной оценки Windows.</span><span class="sxs-lookup"><span data-stu-id="baba7-121">May require use of Windows Insider Builds.</span></span>
* <span data-ttu-id="baba7-122">Подключение TLS 1.2 или более поздней версии.</span><span class="sxs-lookup"><span data-stu-id="baba7-122">TLS 1.2 or later connection</span></span>

### <a name="trailers"></a><span data-ttu-id="baba7-123">Трейлеры</span><span class="sxs-lookup"><span data-stu-id="baba7-123">Trailers</span></span>

[!INCLUDE[](~/includes/trailers.md)]

### <a name="reset"></a><span data-ttu-id="baba7-124">Reset</span><span class="sxs-lookup"><span data-stu-id="baba7-124">Reset</span></span>

[!INCLUDE[](~/includes/reset.md)]