---
title: Защита приложений ASP.NET Core Blazor Server
author: guardrex
description: Сведения о защите приложений Blazor Server как приложений ASP.NET Core.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/06/2020
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
uid: blazor/security/server/index
ms.openlocfilehash: 108fb3a8a24295cad43fd8c83303abd95a7ecd33
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/30/2020
ms.locfileid: "93055481"
---
# <a name="secure-aspnet-core-no-locblazor-server-apps"></a>Защита приложений ASP.NET Core Blazor Server

Автор [Люк Латэм](https://github.com/guardrex) (Luke Latham)

Приложения Blazor Server настраиваются для обеспечения безопасности так же, как приложения ASP.NET Core. Дополнительные сведения см. в статьях в документе <xref:security/index>. Разделы в этом обзоре посвящены Blazor Server.

## <a name="no-locblazor-server-project-template"></a>Шаблон проекта Blazor Server

Шаблон проекта Blazor Server можно настроить для проверки подлинности при создании проекта.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Следуйте инструкциям по работе с Visual Studio (<xref:blazor/tooling>), чтобы создать проект Blazor Server с механизмом проверки подлинности.

Выбрав шаблон **Приложение Blazor Server** в диалоговом окне **Создание веб-приложения ASP.NET Core** , щелкните **Изменить** в разделе **Проверка подлинности** .

Откроется диалоговое окно с тем же набором механизмов аутентификации, которые доступны для других проектов ASP.NET Core.

* **Без аутентификации** .
* **Учетные записи отдельных пользователей** . Учетные записи пользователей могут храниться:
  * Внутри приложения с помощью системы [Identity](xref:security/authentication/identity) в ASP.NET Core.
  * в [Azure AD B2C](xref:security/authentication/azure-ad-b2c);
* **рабочие или учебные учетные записи** .
* **Проверка подлинности Windows**

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Следуйте инструкциям по работе с Visual Studio Code (<xref:blazor/tooling>), чтобы создать проект Blazor Server с механизмом проверки подлинности.

```dotnetcli
dotnet new blazorserver -o {APP NAME} -au {AUTHENTICATION}
```

Допустимые значения аутентификации (`{AUTHENTICATION}`) перечислены в следующей таблице.

| Механизм аутентификации | Описание |
| ------------------------ | ----------- |
| `None` (по умолчанию)         | без аутентификации; |
| `Individual`             | Пользователи, сохраненные в приложении с помощью ASP.NET Core Identity |
| `IndividualB2C`          | Пользователи, сохраненные в [Azure AD B2C](xref:security/authentication/azure-ad-b2c). |
| `SingleOrg`              | Корпоративная аутентификация для отдельного клиента |
| `MultiOrg`               | Корпоративная аутентификация для нескольких клиентов |
| `Windows`                | Проверка подлинности Windows |

С помощью параметра `-o|--output` команда использует значение, указанное для заполнителя `{APP NAME}`, чтобы:

* Создайте папку для проекта.
* Дайте проекту имя.

См. сведения о команде [`dotnet new`](/dotnet/core/tools/dotnet-new) в руководстве по .NET Core.

# <a name="visual-studio-for-mac"></a>[Visual Studio для Mac](#tab/visual-studio-mac)

1. Следуйте инструкциям по работе с Visual Studio для Mac (<xref:blazor/tooling>).

1. На шаге **Настройка нового приложения Blazor Server** выберите **Индивидуальная проверка подлинности (в приложении)** из раскрывающегося списка **Проверка подлинности** .

1. Приложение будет создано для отдельных пользователей, сохраненных в приложении с помощью ASP.NET Core Identity.

# <a name="net-core-cli"></a>[Интерфейс командной строки .NET Core](#tab/netcore-cli/)

Создайте новый проект Blazor Server с механизмом проверки подлинности, используя следующую команду в командной оболочке:

```dotnetcli
dotnet new blazorserver -o {APP NAME} -au {AUTHENTICATION}
```

Допустимые значения аутентификации (`{AUTHENTICATION}`) перечислены в следующей таблице.

| Механизм аутентификации | Описание |
| ------------------------ | ----------- |
| `None` (по умолчанию)         | без аутентификации; |
| `Individual`             | Пользователи, сохраненные в приложении с помощью ASP.NET Core Identity |
| `IndividualB2C`          | Пользователи, сохраненные в [Azure AD B2C](xref:security/authentication/azure-ad-b2c). |
| `SingleOrg`              | Корпоративная аутентификация для отдельного клиента |
| `MultiOrg`               | Корпоративная аутентификация для нескольких клиентов |
| `Windows`                | Проверка подлинности Windows |

С помощью параметра `-o|--output` команда использует значение, указанное для заполнителя `{APP NAME}`, чтобы:

* Создайте папку для проекта.
* Дайте проекту имя.

Дополнительные сведения

* Сведения о команде [`dotnet new`](/dotnet/core/tools/dotnet-new) см. в руководстве по .NET Core.
* Выполните команду help для шаблона Blazor Server (`blazorserver`) в командной оболочке:

  ```dotnetcli
  dotnet new blazorserver --help
  ```

---

## <a name="scaffold-no-locidentity"></a>Формирование шаблонов Identity

Сформируйте шаблоны Identity для проекта Blazor Server:

* [Без существующей авторизации](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-blazor-server-project-without-existing-authorization).
* [С авторизацией](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-blazor-server-project-with-authorization).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Краткое руководство. Добавление входа через Майкрософт в веб-приложение ASP.NET Core](/azure/active-directory/develop/quickstart-v2-aspnet-core-webapp)
* [Краткое руководство. Защита веб-API ASP.NET Core с помощью платформы удостоверений Майкрософт](/azure/active-directory/develop/quickstart-v2-aspnet-core-web-api)
