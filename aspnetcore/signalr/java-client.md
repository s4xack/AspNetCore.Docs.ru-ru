---
title: SignalRКлиент Java ASP.NET Core
author: mikaelm12
description: Узнайте, как использовать SignalR клиент Java ASP.NET Core.
monikerRange: '>= aspnetcore-2.2'
ms.author: mimengis
ms.custom: mvc
ms.date: 11/12/2019
no-loc:
- appsettings.json
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
uid: signalr/java-client
ms.openlocfilehash: 638333176ae31b088bdf5ebefe97e87bde6c0d32
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/30/2020
ms.locfileid: "93051464"
---
# <a name="aspnet-core-no-locsignalr-java-client"></a>SignalRКлиент Java ASP.NET Core

Авторы: [Микаэль Менгисту](https://twitter.com/MikaelM_12) (Mikael Mengistu)

Клиент Java позволяет подключаться к SignalR серверу ASP.NET Core из кода Java, включая приложения Android. Как и клиент [JavaScript](xref:signalr/javascript-client) и [клиент .NET](xref:signalr/dotnet-client), клиент Java позволяет получать и отправлять сообщения в концентратор в режиме реального времени. Клиент Java доступен в ASP.NET Core 2,2 и более поздних версиях.

Пример консольного приложения Java, упоминаемого в этой статье, использует SignalR клиент Java.

[Просмотреть или скачать образец кода](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/signalr/java-client/sample) ([как скачивать](xref:index#how-to-download-a-sample))

## <a name="install-the-no-locsignalr-java-client-package"></a>Установка SignalR клиентского пакета Java

JAR *-файл 1.0.0* позволяет клиентам подключаться к SignalR концентраторам. Чтобы найти последний номер версии JAR-файла, см. [Результаты поиска Maven](https://search.maven.org/search?q=g:com.microsoft.signalr%20AND%20a:signalr).

При использовании Gradle добавьте следующую строку в `dependencies` раздел файла *Build. Gradle* :

```gradle
implementation 'com.microsoft.signalr:signalr:1.0.0'
```

При использовании Maven добавьте следующие строки в `<dependencies>` элемент файла *pom.xml* :

[!code-xml[pom.xml dependency element](java-client/sample/pom.xml?name=snippet_dependencyElement)]

## <a name="connect-to-a-hub"></a>Подключение к концентратору

Чтобы установить `HubConnection` , `HubConnectionBuilder` следует использовать. URL-адрес центра и уровень журнала можно настроить при создании соединения. Настройте все необходимые параметры, вызвав любой из `HubConnectionBuilder` методов, предшествующих `build` . Запустите соединение с `start` .

[!code-java[Build hub connection](java-client/sample/src/main/java/Chat.java?range=16-17)]

## <a name="call-hub-methods-from-client"></a>Методы концентратора вызовов от клиента

Вызов `send` метода вызывает метод концентратора. Передайте имя метода концентратора и все аргументы, определенные в методе концентратора, в `send` .

[!code-java[send method](java-client/sample/src/main/java/Chat.java?range=28)]

> [!NOTE]
> Вызов методов концентратора из клиента поддерживается только при использовании SignalR службы Azure в режиме *по умолчанию* . Дополнительные сведения см. в разделе [часто задаваемые вопросы (репозиторий GitHub Azure SignalR)](https://github.com/Azure/azure-signalr/blob/dev/docs/faq.md#what-is-the-meaning-of-service-mode-defaultserverlessclassic-how-can-i-choose).

## <a name="call-client-methods-from-hub"></a>Вызов методов клиента из концентратора

Используется `hubConnection.on` для определения методов на клиенте, которые может вызывать концентратор. Определите методы после сборки, но перед запуском соединения.

[!code-java[Define client methods](java-client/sample/src/main/java/Chat.java?range=19-21)]

## <a name="add-logging"></a>Добавление ведения журнала

SignalRКлиент Java использует библиотеку [SLF4J](https://www.slf4j.org/) для ведения журнала. Это высокоуровневый API ведения журнала, который позволяет пользователям библиотеки выбирать собственную реализацию ведения журнала, применяя определенную зависимость от ведения журнала. В следующем фрагменте кода показано, как использовать `java.util.logging` с SignalR клиентом Java.

```gradle
implementation 'org.slf4j:slf4j-jdk14:1.7.25'
```

Если вы не настраиваете ведение журнала в зависимостях, SLF4J загружает средство ведения журнала без операций по умолчанию со следующим предупреждающим сообщением:

```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
```

Это можно игнорировать.

## <a name="android-development-notes"></a>Заметки о разработке для Android

С учетом пакет SDK для Android совместимости для SignalR клиентских функций при указании целевой версии пакет SDK для Android учитывайте следующие элементы:

* SignalRКлиент Java будет работать на API Android уровня 16 и более поздних версий.
* Для подключения через службу Azure потребуется SignalR уровень API Android 20 и более поздней версии, так как [ SignalR службе azure](/azure/azure-signalr/signalr-overview) требуется TLS 1,2 и не поддерживаются комплекты шифров на основе SHA-1. В Android [добавлена поддержка комплектов шифров SHA-256 (и более поздних версий)](https://developer.android.com/reference/javax/net/ssl/SSLSocket) на уровне API 20.

## <a name="configure-bearer-token-authentication"></a>Настройка проверки подлинности токена носителя

В SignalR клиенте Java можно настроить токен носителя, который будет использоваться для проверки подлинности, предоставив "фабрику маркеров доступа" для [хттфубконнектионбуилдер](/java/api/com.microsoft.signalr._http_hub_connection_builder?view=aspnet-signalr-java). Используйте [висакцесстокенфактори](/java/api/com.microsoft.signalr._http_hub_connection_builder.withaccesstokenprovider?view=aspnet-signalr-java#com_microsoft_signalr__http_hub_connection_builder_withAccessTokenProvider_Single_String__) для предоставления [рксжава](https://github.com/ReactiveX/RxJava) [Single \<String> ](https://reactivex.io/documentation/single.html). При вызове [Single. отсрочки](https://reactivex.io/RxJava/javadoc/io/reactivex/Single.html#defer-java.util.concurrent.Callable-)можно написать логику для создания маркеров доступа для клиента.

```java
HubConnection hubConnection = HubConnectionBuilder.create("YOUR HUB URL HERE")
    .withAccessTokenProvider(Single.defer(() -> {
        // Your logic here.
        return Single.just("An Access Token");
    })).build();
```

## <a name="known-limitations"></a>Известные ограничения

::: moniker range=">= aspnetcore-3.0"

* Поддерживается только протокол JSON.
* Транспортировка транспорта и транспорт событий отправки сервера не поддерживаются.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

* Поддерживается только протокол JSON.
* Поддерживается только транспорт WebSockets.
* Потоковая передача еще не поддерживается.

::: moniker-end

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Справочник по API Java](/java/api/com.microsoft.signalr?view=aspnet-signalr-java)
* <xref:signalr/hubs>
* <xref:signalr/javascript-client>
* <xref:signalr/publish-to-azure-web-app>
* [SignalRДокументация, посвященная бессерверной службе Azure](/azure/azure-signalr/signalr-concept-serverless-development-config)
