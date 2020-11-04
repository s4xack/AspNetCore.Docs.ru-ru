---
title: 'Изоляция CSS в ASP.NET Core :::no-loc(Blazor):::'
author: daveabrock
description: Узнайте, как изоляция CSS позволяет ограничить область применения CSS к компонентам, что позволяет упростить CSS и избежать конфликтов с другими компонентами или библиотеками.
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/20/2020
no-loc:
- ':::no-loc(ASP.NET Core Identity):::'
- ':::no-loc(cookie):::'
- ':::no-loc(Cookie):::'
- ':::no-loc(Blazor):::'
- ':::no-loc(Blazor Server):::'
- ':::no-loc(Blazor WebAssembly):::'
- ':::no-loc(Identity):::'
- ":::no-loc(Let's Encrypt):::"
- ':::no-loc(Razor):::'
- ':::no-loc(SignalR):::'
uid: blazor/components/css-isolation
ms.openlocfilehash: c154e746c4c88fc919b2c0dddaea5fd585427a82
ms.sourcegitcommit: d84a225ec3381355c343460deed50f2fa5722f60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/22/2020
ms.locfileid: "92431828"
---
# <a name="aspnet-core-no-locblazor-css-isolation"></a><span data-ttu-id="cb25c-103">Изоляция CSS в ASP.NET Core :::no-loc(Blazor):::</span><span class="sxs-lookup"><span data-stu-id="cb25c-103">ASP.NET Core :::no-loc(Blazor)::: CSS isolation</span></span>

<span data-ttu-id="cb25c-104">Автор: [Дейв Брок](https://twitter.com/daveabrock) (Dave Brock)</span><span class="sxs-lookup"><span data-stu-id="cb25c-104">By [Dave Brock](https://twitter.com/daveabrock)</span></span>

<span data-ttu-id="cb25c-105">Изоляция CSS упрощает использование CSS в приложении, предотвращая зависимости от глобальных стилей, и помогает избежать конфликтов стилей между компонентами и библиотеками.</span><span class="sxs-lookup"><span data-stu-id="cb25c-105">CSS isolation simplifies an app's CSS footprint by preventing dependencies on global styles and helps to avoid styling conflicts among components and libraries.</span></span>

## <a name="enable-css-isolation"></a><span data-ttu-id="cb25c-106">Включение изоляции CSS</span><span class="sxs-lookup"><span data-stu-id="cb25c-106">Enable CSS isolation</span></span> 

<span data-ttu-id="cb25c-107">Чтобы определить стили, относящиеся к компоненту, создайте файл `razor.css`, соответствующий имени файла `.razor` для компонента.</span><span class="sxs-lookup"><span data-stu-id="cb25c-107">To define component-specific styles, create a `razor.css` file matching the name of the `.razor` file for the component.</span></span> <span data-ttu-id="cb25c-108">Этот файл `razor.css` будет *файлом CSS с областью действия*.</span><span class="sxs-lookup"><span data-stu-id="cb25c-108">This `razor.css` file is a *scoped CSS file*.</span></span> 

<span data-ttu-id="cb25c-109">Для компонента `MyComponent`, имеющего файл `MyComponent.razor`, создайте файл с именем `MyComponent.razor.css` рядом с компонентом.</span><span class="sxs-lookup"><span data-stu-id="cb25c-109">For a `MyComponent` component that has a `MyComponent.razor` file, create a file alongside the component called `MyComponent.razor.css`.</span></span> <span data-ttu-id="cb25c-110">Часть `MyComponent` в имени файла `razor.css` **не** учитывает регистр.</span><span class="sxs-lookup"><span data-stu-id="cb25c-110">The `MyComponent` value in the `razor.css` filename is **not** case-sensitive.</span></span>

<span data-ttu-id="cb25c-111">Например, чтобы добавить изоляцию CSS в компонент `Counter` в шаблоне по умолчанию проекта :::no-loc(Blazor):::, добавьте новый файл с именем `Counter.razor.css` в место расположения файла `Counter.razor`, а затем добавьте следующий код CSS:</span><span class="sxs-lookup"><span data-stu-id="cb25c-111">For example to add CSS isolation to the `Counter` component in the default :::no-loc(Blazor)::: project template, add a new file named `Counter.razor.css` alongside the `Counter.razor` file, then add the following CSS:</span></span>

```css
h1 { 
    color: brown;
    font-family: Tahoma, Geneva, Verdana, sans-serif;
}
```

<span data-ttu-id="cb25c-112">Стили, определенные в файле `Counter.razor.css`, применяются только к отображаемым выходным данным компонента `Counter`.</span><span class="sxs-lookup"><span data-stu-id="cb25c-112">The styles defined in `Counter.razor.css` are only applied to the rendered output of the `Counter` component.</span></span> <span data-ttu-id="cb25c-113">Все объявления CSS `h1`, определенные в других местах приложения, не будут конфликтовать со стилями `Counter`.</span><span class="sxs-lookup"><span data-stu-id="cb25c-113">Any `h1` CSS declarations defined elsewhere in the app don't conflict with `Counter` styles.</span></span>

> [!NOTE]
> <span data-ttu-id="cb25c-114">Чтобы обеспечить изоляцию стиля при возникновении объединений, блоки :::no-loc(Razor)::: `@import` не поддерживаются в файлах CSS с заданной областью.</span><span class="sxs-lookup"><span data-stu-id="cb25c-114">In order to guarantee style isolation when bundling occurs, `@import` :::no-loc(Razor)::: blocks aren't supported with scoped CSS files.</span></span>

## <a name="css-isolation-bundling"></a><span data-ttu-id="cb25c-115">Объединение изоляций CSS</span><span class="sxs-lookup"><span data-stu-id="cb25c-115">CSS isolation bundling</span></span>

<span data-ttu-id="cb25c-116">Изоляция CSS выполняется во время сборки.</span><span class="sxs-lookup"><span data-stu-id="cb25c-116">CSS isolation occurs at build time.</span></span> <span data-ttu-id="cb25c-117">Во время этого процесса :::no-loc(Blazor)::: переписывает селекторы CSS в соответствии с разметкой, отображаемой компонентом.</span><span class="sxs-lookup"><span data-stu-id="cb25c-117">During this process, :::no-loc(Blazor)::: rewrites CSS selectors to match markup rendered by the component.</span></span> <span data-ttu-id="cb25c-118">Эти переписанные стили CSS объединяются и создаются как статические ресурсы в `{PROJECT NAME}.styles.css`, где заполнитель `{PROJECT NAME}` является названием пакета или продукта, на который указывает ссылка.</span><span class="sxs-lookup"><span data-stu-id="cb25c-118">These rewritten CSS styles are bundled and produced as a static asset at `{PROJECT NAME}.styles.css`, where the placeholder `{PROJECT NAME}` is the referenced package or product name.</span></span>

<span data-ttu-id="cb25c-119">По умолчанию эти статические файлы ищутся от корневого пути приложения.</span><span class="sxs-lookup"><span data-stu-id="cb25c-119">These static files are referenced from the root path of the app by default.</span></span> <span data-ttu-id="cb25c-120">В приложении ссылайтесь на объединенный файл, проверив ссылку внутри тега `<head>` созданного HTML:</span><span class="sxs-lookup"><span data-stu-id="cb25c-120">In the app, reference the bundled file by inspecting the reference inside the `<head>` tag of the generated HTML:</span></span>

```html
<link href="MyProjectName.styles.css" rel="stylesheet">
```

<span data-ttu-id="cb25c-121">В объединенном файле каждый компонент связан с идентификатором области.</span><span class="sxs-lookup"><span data-stu-id="cb25c-121">Within the bundled file, each component is associated with a scope identifier.</span></span> <span data-ttu-id="cb25c-122">Для каждого компонента, имеющего стиль, атрибут HTML добавляется в формате `b-<10-character-string>`.</span><span class="sxs-lookup"><span data-stu-id="cb25c-122">For each styled component, an HTML attribute is appended with the format `b-<10-character-string>`.</span></span> <span data-ttu-id="cb25c-123">Идентификатор уникален и отличается для каждого приложения.</span><span class="sxs-lookup"><span data-stu-id="cb25c-123">The identifier is unique and different for each app.</span></span> <span data-ttu-id="cb25c-124">В отображаемом компоненте `Counter` :::no-loc(Blazor)::: добавляет идентификатор области к элементу `h1`:</span><span class="sxs-lookup"><span data-stu-id="cb25c-124">In the rendered `Counter` component, :::no-loc(Blazor)::: appends a scope identifier to the `h1` element:</span></span>

```html
<h1 b-3xxtam6d07>
```

<span data-ttu-id="cb25c-125">Файл `MyProjectName.styles.css` использует идентификатор области для группирования объявления стиля с его компонентом.</span><span class="sxs-lookup"><span data-stu-id="cb25c-125">The `MyProjectName.styles.css` file uses the scope identifier to group a style declaration with its component.</span></span> <span data-ttu-id="cb25c-126">В следующем примере представлен стиль для предыдущего элемента `<h1>`:</span><span class="sxs-lookup"><span data-stu-id="cb25c-126">The following example provides the style for the preceding `<h1>` element:</span></span>

```css
/* /Pages/Counter.razor.rz.scp.css */
h1[b-3xxtam6d07] {
    color: brown;
}
```

<span data-ttu-id="cb25c-127">Во время сборки создается пакет проектов с использованием соглашения `{STATIC WEB ASSETS BASE PATH}/MyProject.lib.scp.css`, где заполнитель `{STATIC WEB ASSETS BASE PATH}` является базовым путем к статическим веб-ресурсам.</span><span class="sxs-lookup"><span data-stu-id="cb25c-127">At build time, a project bundle is created with the convention `{STATIC WEB ASSETS BASE PATH}/MyProject.lib.scp.css`, where the placeholder `{STATIC WEB ASSETS BASE PATH}` is the static web assets base path.</span></span>

<span data-ttu-id="cb25c-128">Если используются другие проекты, такие как пакеты NuGet или [библиотеки классов :::no-loc(Razor):::](xref:blazor/components/class-libraries), то объединенный файл:</span><span class="sxs-lookup"><span data-stu-id="cb25c-128">If other projects are utilized, such as NuGet packages or [:::no-loc(Razor)::: class libraries](xref:blazor/components/class-libraries), the bundled file:</span></span>

* <span data-ttu-id="cb25c-129">ссылается на стили с помощью импорта CSS;</span><span class="sxs-lookup"><span data-stu-id="cb25c-129">References the styles using CSS imports.</span></span>
* <span data-ttu-id="cb25c-130">не публикуется как статический веб-ресурс приложения, использующего стили.</span><span class="sxs-lookup"><span data-stu-id="cb25c-130">Isn't published as a static web asset of the app that consumes the styles.</span></span>

## <a name="child-component-support"></a><span data-ttu-id="cb25c-131">Поддержка дочерних компонентов</span><span class="sxs-lookup"><span data-stu-id="cb25c-131">Child component support</span></span>

<span data-ttu-id="cb25c-132">По умолчанию изоляция CSS применяется только к компоненту, привязанному с помощью формата `{COMPONENT NAME}.razor.css`, где заполнитель `{COMPONENT NAME}` обычно является именем компонента.</span><span class="sxs-lookup"><span data-stu-id="cb25c-132">By default, CSS isolation only applies to the component you associate with the format `{COMPONENT NAME}.razor.css`, where the placeholder `{COMPONENT NAME}` is usually the component name.</span></span> <span data-ttu-id="cb25c-133">Чтобы применить изменения к дочернему компоненту, используйте объединение `::deep` для всех элементов-потомков в файле `razor.css` родительского компонента.</span><span class="sxs-lookup"><span data-stu-id="cb25c-133">To apply changes to a child component, use the `::deep` combinator to any descendant elements in the parent component's `razor.css` file.</span></span> <span data-ttu-id="cb25c-134">Объединение `::deep` выбирает элементы, которые являются *потомками* идентификатора области, созданного элементом.</span><span class="sxs-lookup"><span data-stu-id="cb25c-134">The `::deep` combinator selects elements that are *descendants* of an element's generated scope identifier.</span></span> 

<span data-ttu-id="cb25c-135">В следующем примере показан родительский компонент с именем `Parent` и дочерний компонент с именем `Child`.</span><span class="sxs-lookup"><span data-stu-id="cb25c-135">The following example shows a parent component called `Parent` with a child component called `Child`.</span></span>

<span data-ttu-id="cb25c-136">`Parent.razor`:</span><span class="sxs-lookup"><span data-stu-id="cb25c-136">`Parent.razor`:</span></span>

```razor
@page "/parent"

<div>
    <h1>Parent component</h1>

    <Child />
</div>
```

<span data-ttu-id="cb25c-137">`Child.razor`:</span><span class="sxs-lookup"><span data-stu-id="cb25c-137">`Child.razor`:</span></span>

```razor
<h1>Child Component</h1>
```

<span data-ttu-id="cb25c-138">Измените объявление `h1` в `Parent.razor.css`, добавив объединение `::deep` для указания того, что объявление стиля `h1` должно применяться к родительскому компоненту и его дочерним элементам:</span><span class="sxs-lookup"><span data-stu-id="cb25c-138">Update the `h1` declaration in `Parent.razor.css` with the `::deep` combinator to signify the `h1` style declaration must apply to the parent component and its children:</span></span>

```css
::deep h1 { 
    color: red;
}
```

<span data-ttu-id="cb25c-139">Стиль `h1` теперь применяется к компонентам `Parent` и `Child` без необходимости создания отдельного CSS-файла с областью действия для дочернего компонента.</span><span class="sxs-lookup"><span data-stu-id="cb25c-139">The `h1` style now applies to the `Parent` and `Child` components without the need to create a separate scoped CSS file for the child component.</span></span>

> [!NOTE]
> <span data-ttu-id="cb25c-140">Объединение `::deep` работает только с элементами-потомками.</span><span class="sxs-lookup"><span data-stu-id="cb25c-140">The `::deep` combinator only works with descendant elements.</span></span> <span data-ttu-id="cb25c-141">Следующая структура HTML применяет стили `h1` к компонентам, как и ожидалось:</span><span class="sxs-lookup"><span data-stu-id="cb25c-141">The following HTML structure applies the `h1` styles to components as expected:</span></span>
> 
> ```razor
> <div>
>     <h1>Parent</h1>
>
>     <Child />
> </div>
> ```
>
> <span data-ttu-id="cb25c-142">В этом сценарии ASP.NET Core применяет идентификатор области родительского компонента к элементу `div`, поэтому браузеру известно, что он наследует стили от родительского компонента.</span><span class="sxs-lookup"><span data-stu-id="cb25c-142">In this scenario, ASP.NET Core applies the parent component's scope identifier to the `div` element, so the browser knows to inherit styles from the parent component.</span></span>
>
> <span data-ttu-id="cb25c-143">Однако исключение элемента `div` удаляет отношение потомков, и стиль **не** применяется к дочернему компоненту:</span><span class="sxs-lookup"><span data-stu-id="cb25c-143">However, excluding the `div` element removes the descendant relationship, and the style is **not** applied to the child component:</span></span>
>
> ```razor
> <h1>Parent</h1>
>
> <Child />
> ```

## <a name="css-preprocessor-support"></a><span data-ttu-id="cb25c-144">Поддержка препроцессоров CSS</span><span class="sxs-lookup"><span data-stu-id="cb25c-144">CSS preprocessor support</span></span>

<span data-ttu-id="cb25c-145">Препроцессоры CSS полезны для улучшения разработки CSS с помощью таких функций, как переменные, вложенность, модули, примеси и наследование.</span><span class="sxs-lookup"><span data-stu-id="cb25c-145">CSS preprocessors are useful for improving CSS development by utilizing features such as variables, nesting, modules, mixins, and inheritance.</span></span> <span data-ttu-id="cb25c-146">Хотя изоляция CSS изначально не поддерживает препроцессоры CSS, такие как Sass и Less, интеграция препроцессоров CSS происходит прозрачно, поскольку компиляция препроцессора выполняется до того, как :::no-loc(Blazor)::: перезаписывает селекторы CSS в процессе сборки.</span><span class="sxs-lookup"><span data-stu-id="cb25c-146">While CSS isolation doesn't natively support CSS preprocessors such as Sass or Less, integrating CSS preprocessors is seamless as long as preprocessor compilation occurs before :::no-loc(Blazor)::: rewrites the CSS selectors during the build process.</span></span> <span data-ttu-id="cb25c-147">С помощью Visual Studio, например, настройте существующую компиляцию препроцессора в качестве задачи **перед сборкой** в диспетчере выполнения задач Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="cb25c-147">Using Visual Studio for example, configure existing preprocessor compilation as a **Before Build** task in the Visual Studio Task Runner Explorer.</span></span>

<span data-ttu-id="cb25c-148">Многие сторонние пакеты NuGet, такие как [Delegate.SassBuilder](https://www.nuget.org/packages/Delegate.SassBuilder), могут компилировать файлы SASS и SCSS в начале процесса сборки перед выполнением изоляции CSS, и дополнительная настройка не требуется.</span><span class="sxs-lookup"><span data-stu-id="cb25c-148">Many third-party NuGet packages, such as [Delegate.SassBuilder](https://www.nuget.org/packages/Delegate.SassBuilder), can compile SASS/SCSS files at the beginning of the build process before CSS isolation occurs, and no additional additional configuration is required.</span></span>

## <a name="css-isolation-configuration"></a><span data-ttu-id="cb25c-149">Конфигурация изоляции CSS</span><span class="sxs-lookup"><span data-stu-id="cb25c-149">CSS isolation configuration</span></span>

<span data-ttu-id="cb25c-150">Изоляция CSS может работать без дополнительных настроек, но она предоставляет возможности конфигурации для некоторых сложных сценариев, например при наличии зависимостей от существующих инструментов или рабочих процессов.</span><span class="sxs-lookup"><span data-stu-id="cb25c-150">CSS isolation is designed to work out-of-the-box but provides configuration for some advanced scenarios, such as when there are dependencies on existing tools or workflows.</span></span>

### <a name="customize-scope-identifier-format"></a><span data-ttu-id="cb25c-151">Настройка формата идентификатора области</span><span class="sxs-lookup"><span data-stu-id="cb25c-151">Customize scope identifier format</span></span>

<span data-ttu-id="cb25c-152">По умолчанию для идентификаторов областей используется формат `b-<10-character-string>`.</span><span class="sxs-lookup"><span data-stu-id="cb25c-152">By default, scope identifiers use the format `b-<10-character-string>`.</span></span> <span data-ttu-id="cb25c-153">Чтобы настроить формат идентификатора области, измените шаблон в файле проекта:</span><span class="sxs-lookup"><span data-stu-id="cb25c-153">To customize the scope identifier format, update the project file to a desired pattern:</span></span>

```xml
<ItemGroup>
    <None Update="MyComponent.razor.css" CssScope="my-custom-scope-identifier" />
</ItemGroup>
```

<span data-ttu-id="cb25c-154">В предыдущем примере CSS, сформированный для `MyComponent.:::no-loc(Razor):::.css`, изменяет свой идентификатор области с `b-<10-character-string>` на `my-custom-scope-identifier`.</span><span class="sxs-lookup"><span data-stu-id="cb25c-154">In the preceding example, the CSS generated for `MyComponent.:::no-loc(Razor):::.css` changes its scope identifier from `b-<10-character-string>` to `my-custom-scope-identifier`.</span></span>

### <a name="change-base-path-for-static-web-assets"></a><span data-ttu-id="cb25c-155">Изменение базового пути для статических веб-ресурсов</span><span class="sxs-lookup"><span data-stu-id="cb25c-155">Change base path for static web assets</span></span>

<span data-ttu-id="cb25c-156">Файл `scoped.styles.css` создается в корне приложения.</span><span class="sxs-lookup"><span data-stu-id="cb25c-156">The `scoped.styles.css` file is generated at the root of the app.</span></span> <span data-ttu-id="cb25c-157">Чтобы изменить путь по умолчанию, используйте свойство `StaticWebAssetBasePath` в файле проекта.</span><span class="sxs-lookup"><span data-stu-id="cb25c-157">In the project file, use the `StaticWebAssetBasePath` property to change the default path.</span></span> <span data-ttu-id="cb25c-158">В следующем примере файл `scoped.styles.css` и остальные ресурсы приложения размещаются по пути `_content`:</span><span class="sxs-lookup"><span data-stu-id="cb25c-158">The following example places the `scoped.styles.css` file, and the rest of the app's assets, at the `_content` path:</span></span>

```xml
<PropertyGroup>
  <StaticWebAssetBasePath>_content/$(PackageId)</StaticWebAssetBasePath>
</PropertyGroup>
```

### <a name="disable-automatic-bundling"></a><span data-ttu-id="cb25c-159">Отключение автоматического объединения</span><span class="sxs-lookup"><span data-stu-id="cb25c-159">Disable automatic bundling</span></span>

<span data-ttu-id="cb25c-160">Чтобы отказаться от того, как :::no-loc(Blazor)::: публикует и загружает файлы с заданной областью во время выполнения, используйте свойство `DisableScopedCssBundling`.</span><span class="sxs-lookup"><span data-stu-id="cb25c-160">To opt out of how :::no-loc(Blazor)::: publishes and loads scoped files at runtime, use the `DisableScopedCssBundling` property.</span></span> <span data-ttu-id="cb25c-161">При использовании этого свойства за получение изолированных файлов CSS из каталога `obj` и их публикацию и загрузку во время выполнения отвечают другие средства или процессы:</span><span class="sxs-lookup"><span data-stu-id="cb25c-161">When using this property, it means other tools or processes are responsible for taking the isolated CSS files from the `obj` directory and publishing and loading them at runtime:</span></span>

```xml
<PropertyGroup>
  <DisableScopedCssBundling>true</DisableScopedCssBundling>
</PropertyGroup>
```