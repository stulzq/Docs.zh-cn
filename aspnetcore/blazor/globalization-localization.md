---
title: ASP.NET Core Blazor 全球化和本地化
author: guardrex
description: 了解如何使 Razor 组件能够供位于不同区域、使用不同语言的用户使用。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/12/2020
no-loc:
- Blazor
- SignalR
uid: blazor/globalization-localization
ms.openlocfilehash: aba62fa7b6285c8ba884652694f1ea3e3a66ed18
ms.sourcegitcommit: 9a129f5f3e31cc449742b164d5004894bfca90aa
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/06/2020
ms.locfileid: "78644892"
---
# <a name="aspnet-core-opno-locblazor-globalization-and-localization"></a>ASP.NET Core Blazor 全球化和本地化

作者：[Luke Latham](https://github.com/guardrex) 和 [Daniel Roth](https://github.com/danroth27)

Razor 组件可供位于不同区域、使用不同语言的用户使用。 以下 .NET 全球化和本地化方案可用：

* .NET 资源系统
* 特定于区域性的数字和日期格式

当前支持有限的 ASP.NET Core 本地化方案：

* Blazor 应用中支持 `IStringLocalizer<>`  。
* `IHtmlLocalizer<>`、`IViewLocalizer<>` 和数据注释本地化是 ASP.NET Core MVC 方案，在 Blazor 应用中不受支持  。

有关详细信息，请参阅 <xref:fundamentals/localization>。

## <a name="globalization"></a>全球化

Blazor 的 `@bind` 功能基于用户的当前区域性执行格式并分析值以进行显示。

可从 <xref:System.Globalization.CultureInfo.CurrentCulture?displayProperty=fullName> 属性访问当前区域性。

[CultureInfo.InvariantCulture](xref:System.Globalization.CultureInfo.InvariantCulture) 用于以下字段类型 (`<input type="{TYPE}" />`)：

* `date`
* `number`

上述字段类型：

* 使用其基于浏览器的相应格式规则进行显示。
* 不能包含自由格式的文本。
* 基于浏览器的实现提供用户交互特性。

以下字段类型具有特定的格式要求且当前不受 Blazor 支持，因为所有主流浏览器均不支持它们：

* `datetime-local`
* `month`
* `week`

`@bind` 支持 `@bind:culture` 参数，以提供用于分析值并设置值格式的 <xref:System.Globalization.CultureInfo?displayProperty=fullName>。 使用 `date` 和 `number` 字段类型时，不建议指定区域性。 `date` 和 `number` 具有可提供所需区域性的内置 Blazor 支持。

## <a name="localization"></a>本地化

Blazor 服务器应用使用[本地化中间件](xref:fundamentals/localization#localization-middleware)进行本地化。 中间件为从应用请求资源的用户选择相应的区域性。

可使用以下方法之一设置区域性：

* [Cookie](#cookies)
* [提供用于选择区域性的 UI](#provide-ui-to-choose-the-culture)

有关更多信息和示例，请参见<xref:fundamentals/localization>。

### <a name="configure-the-linker-for-internationalization-opno-locblazor-webassembly"></a>为国际化配置链接器 (Blazor WebAssembly)

默认情况下，Blazor 对于 Blazor WebAssembly 应用的链接器配置会去除国际化信息（显式请求的区域设置除外）。 有关控制链接器行为的详细信息和指南，请参阅 <xref:host-and-deploy/blazor/configure-linker#configure-the-linker-for-internationalization>。

### <a name="cookies"></a>Cookie

本地化区域性 cookie 可以保留用户的区域性。 该 cookie 是通过应用主机页 (Pages/Host.cshtml.cs) 的 `OnGet` 方法创建的  。 本地化中间件会在后续请求上读取 cookie，以设置用户的区域性。 

使用 cookie 可确保 WebSocket 连接可以正确地传播区域性。 如果本地化方案基于 URL 路径或查询字符串，则该方案可能无法与 Websocket 协同使用，因而无法保留区域性。 因此，建议的方式是使用本地化区域性 cookie。

如果在本地化 cookie 中保留了区域性，则可以使用任意方法来分配区域性。 如果该应用已经为服务器端 ASP.NET Core 建立了本地化方案，请继续使用应用的现有本地化基础结构，并在应用方案中设置本地化区域性 cookie。

下面的示例演示如何在可由本地化中间件读取的 cookie 中设置当前区域性。 在 Blazor 服务器应用中创建包含以下内容的 Pages/Host.cshtml.cs 文件  ：

```csharp
public class HostModel : PageModel
{
    public void OnGet()
    {
        HttpContext.Response.Cookies.Append(
            CookieRequestCultureProvider.DefaultCookieName,
            CookieRequestCultureProvider.MakeCookieValue(
                new RequestCulture(
                    CultureInfo.CurrentCulture,
                    CultureInfo.CurrentUICulture)));
    }
}
```

本地化由应用按以下事件顺序进行处理：

1. 浏览器向应用发送初始 HTTP 请求。
1. 本地化中间件分配区域性。
1. _Host.cshtml.cs 中的 `OnGet` 方法将区域性作为响应的一部分保留在 cookie 中  。
1. 浏览器打开 WebSocket 连接以创建交互式 Blazor 服务器会话。
1. 本地化中间件读取 cookie 并分配区域性。
1. Blazor 服务器会话以正确的区域性开始。

### <a name="provide-ui-to-choose-the-culture"></a>提供用于选择区域性的 UI

若要提供支持用户选择区域性的 UI，建议使用基于重定向的方法  。 此过程类似于用户尝试访问安全资源时在 web 应用中发生的情况 &mdash; 用户重定向到登录页，然后重定向回原始资源。 

应用通过重定向到控制器来保留用户的所选区域性。 控制器将用户选择的区域性设置为 cookie，然后将用户重定向回原始 URI。

在服务器上创建一个 HTTP 终结点，以在 cookie 中设置用户的所选区域性，并执行重定向回原始 URI：

```csharp
[Route("[controller]/[action]")]
public class CultureController : Controller
{
    public IActionResult SetCulture(string culture, string redirectUri)
    {
        if (culture != null)
        {
            HttpContext.Response.Cookies.Append(
                CookieRequestCultureProvider.DefaultCookieName,
                CookieRequestCultureProvider.MakeCookieValue(
                    new RequestCulture(culture)));
        }

        return LocalRedirect(redirectUri);
    }
}
```

> [!WARNING]
> 使用 `LocalRedirect` 操作结果以阻止开放式重定向攻击。 有关详细信息，请参阅 <xref:security/preventing-open-redirects>。

以下组件显示了一个示例，说明如何在用户选择区域性时执行初始重定向：

```razor
@inject NavigationManager NavigationManager

<h3>Select your language</h3>

<select @onchange="OnSelected">
    <option>Select...</option>
    <option value="en-US">English</option>
    <option value="fr-FR">Français</option>
</select>

@code {
    private void OnSelected(ChangeEventArgs e)
    {
        var culture = (string)e.Value;
        var uri = new Uri(NavigationManager.Uri())
            .GetComponents(UriComponents.PathAndQuery, UriFormat.Unescaped);
        var query = $"?culture={Uri.EscapeDataString(culture)}&" +
            $"redirectUri={Uri.EscapeDataString(uri)}";

        NavigationManager.NavigateTo("/Culture/SetCulture" + query, forceLoad: true);
    }
}
```

## <a name="additional-resources"></a>其他资源

* <xref:fundamentals/localization>
