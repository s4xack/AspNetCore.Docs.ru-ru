---
title: Наследование подраздела и шифрование с проверкой подлинности в ASP.NET Core
author: rick-anderson
description: Сведения о реализации ASP.NET Core наследования подраздела защиты данных и шифрования с проверкой подлинности.
ms.author: riande
ms.date: 10/14/2016
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
uid: security/data-protection/implementation/subkeyderivation
ms.openlocfilehash: efe8ad2f71feda9cbc1693d362e30eff29cbcd74
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060161"
---
# <a name="subkey-derivation-and-authenticated-encryption-in-aspnet-core"></a>Наследование подраздела и шифрование с проверкой подлинности в ASP.NET Core

<a name="data-protection-implementation-subkey-derivation"></a>

Большинство ключей в кольце ключа будет содержать некоторую форму энтропии и будет содержать алгоритмы, указывающие "шифрование режима CBC + проверка HMAC" и "шифрование GCM + проверка". В этих случаях в качестве главного материала для ключа (или км) мы будем обращаться к внедренной энтропии, а для получения ключей, которые будут использоваться для фактических криптографических операций, используется функция формирования ключа.

> [!NOTE]
> Ключи являются абстрактными, и пользовательская реализация может работать не так, как показано ниже. Если ключ предоставляет собственную реализацию `IAuthenticatedEncryptor` вместо использования одной из встроенных фабрик, механизм, описанный в этом разделе, больше не применяется.

<a name="data-protection-implementation-subkey-derivation-aad"></a>

## <a name="additional-authenticated-data-and-subkey-derivation"></a>Дополнительные данные с проверкой подлинности и наследование подраздела

`IAuthenticatedEncryptor`Интерфейс выступает в качестве базового интерфейса для всех операций шифрования, прошедших проверку подлинности. Его `Encrypt` метод принимает два буфера: обычный текст и аддитионалаусентикатеддата (AAD). Поток содержимого в виде обычного текста не изменил вызов `IDataProtector.Protect` , а AAD создается системой и состоит из трех компонентов:

1. 32-разрядный заголовок Magic F0 C9 F0, определяющий эту версию системы защиты данных.

2. Идентификатор 128-разрядного ключа.

3. Строка переменной длины, сформированная из цепочки назначений, создавшей объект `IDataProtector` , выполняющий эту операцию.

Поскольку AAD является уникальным для кортежа всех трех компонентов, мы можем использовать его для получения новых ключей с км вместо того, чтобы использовать каждый из этих криптографических операций. При каждом вызове `IAuthenticatedEncryptor.Encrypt` происходит следующий процесс формирования ключа:

`( K_E, K_H ) = SP800_108_CTR_HMACSHA512(K_M, AAD, contextHeader || keyModifier)`

Здесь мы вызываем директиву NIST SP800-108 ПОДПРОГРАММ в режиме счетчика (см. директиву [NIST SP800-108](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-108.pdf), Sec. 5,1) со следующими параметрами:

* Ключ наследования ключа (KDK) = `K_M`

* PRF = HMACSHA512

* Метка = Аддитионалаусентикатеддата

* context = Контекссеадер | | кэймодифиер

Заголовок контекста имеет переменную длину и, по сути, служит отпечаткой алгоритмов, для которых выполняется наследование `K_E` и `K_H` . Модификатор ключа — это 128-разрядная строка, создаваемая случайным образом для каждого вызова `Encrypt` и служащая для обеспечения перегруженной вероятности, что для данной конкретной операции шифрования проверки подлинности регистр и KH являются уникальными, даже если все остальные входные данные в подпрограмм являются постоянными.

Для операций шифрования в режиме CBC и проверки HMAC `| K_E |` — это длина симметричного блочного ключа шифра, а `| K_H |` — Размер дайджеста подпрограммы HMAC. Для операций шифрования GCM и проверки `| K_H | = 0` .

## <a name="cbc-mode-encryption--hmac-validation"></a>Шифрование в режиме CBC и проверка HMAC

После создания с `K_E` помощью приведенного выше механизма создается случайный вектор инициализации и запускается алгоритм симметричного блочного шифра, шифрования паролей обычный текст. Затем вектор инициализации и зашифрованный текст выполняются с помощью процедуры HMAC, инициализированной ключом `K_H` для создания компьютера Mac. Этот процесс и возвращаемое значение представлены графически ниже.

![Процесс и возврат в режиме CBC](subkeyderivation/_static/cbcprocess.png)

`output:= keyModifier || iv || E_cbc (K_E,iv,data) || HMAC(K_H, iv || E_cbc (K_E,iv,data))`

> [!NOTE]
> `IDataProtector.Protect`Реализация будет [добавлять в начале вывода заголовок Magic и идентификатор ключа](xref:security/data-protection/implementation/authenticated-encryption-details) , прежде чем возвращать его вызывающему объекту. Так как заголовок Magic и идентификатор ключа неявно являются частью [AAD](xref:security/data-protection/implementation/subkeyderivation#data-protection-implementation-subkey-derivation-aad), и так как модификатор ключа помещается в качестве входных данных для подпрограмм, это означает, что каждый байт окончательно возвращенных полезных данных проходит проверку подлинности Mac.

## <a name="galoiscounter-mode-encryption--validation"></a>Шифрование и проверка галоис/режим счетчика

После создания `K_E` с помощью приведенного выше механизма мы создаем случайный 96-разрядный nonce и запустили алгоритм симметричного блочного шифра, чтобы шифрования паролей открытый текст и создали тег 128-разрядной проверки подлинности.

![Процесс и возврат в режиме GCM](subkeyderivation/_static/galoisprocess.png)

`output := keyModifier || nonce || E_gcm (K_E,nonce,data) || authTag`

> [!NOTE]
> Несмотря на то, что GCM изначально поддерживает концепцию AAD, мы все еще передаем AAD только первоначальному ПОДПРОГРАММу, пропуская пустую строку в GCM для своего параметра AAD. Причина этого состоит в двух разделах. Во первых, [для обеспечения гибкости](xref:security/data-protection/implementation/context-headers#data-protection-implementation-context-headers) мы никогда не будем использовать их `K_M` напрямую в качестве ключа шифрования. Кроме того, GCM накладывает очень строгое требование уникальности на входные данные. Вероятность того, что подпрограммы шифрования GCM когда-либо вызваны для двух или более отдельных наборов входных данных с одинаковой парой (Key, nonce), не должна превышать 2 ^ 32. Если исправить проблему `K_E` , мы не можем выполнить более 2 операций шифрования в ^ 32 до выполнения афаул из 2 ^-32 ограничения. Это может показаться очень большим количеством операций, но веб-сервер с большим трафиком может пройти через 4 000 000 000 запросов в течение всего дня, а не в течение обычного времени существования этих ключей. Чтобы обеспечить соответствие требованиям 2 ^-32, мы продолжаем использовать модификатор с 128-битным ключом и 96-bit nonce, который радикально расширяет количество используемых операций для всех указанных `K_M` . Для простоты проектирования мы используем путь кода ПОДПРОГРАММ между операциями CBC и GCM, а так как AAD уже рассматривается в ПОДПРОГРАММ, нет необходимости пересылать его в подпрограмму GCM.
