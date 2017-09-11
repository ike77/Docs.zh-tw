---
title: "快取中 ASP.NET Core 中的介軟體的回應"
author: guardrex
description: "設定及使用回應快取中介軟體中 ASP.NET Core 應用程式。"
keywords: "ASP.NET Core 回應快取，快取，ResponseCache，ResponseCaching，快取控制、 VaryByQueryKeys、 中介軟體"
ms.author: riande
manager: wpickett
ms.date: 08/22/2017
ms.topic: article
ms.assetid: f9267eab-2762-42ac-1638-4a25d2c9d67c
ms.prod: asp.net-core
uid: performance/caching/middleware
ms.openlocfilehash: 7790f38dda61eabd3cbbc6088ad455c07289b739
ms.sourcegitcommit: 70089de5bfd8ecd161261aa95faf07a4e1534cf8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2017
---
# <a name="response-caching-middleware-in-aspnet-core"></a><span data-ttu-id="20c0f-104">快取中 ASP.NET Core 中的介軟體的回應</span><span class="sxs-lookup"><span data-stu-id="20c0f-104">Response Caching Middleware in ASP.NET Core</span></span>

<span data-ttu-id="20c0f-105">由[Luke Latham](https://github.com/GuardRex)和[John Luo](https://github.com/JunTaoLuo)</span><span class="sxs-lookup"><span data-stu-id="20c0f-105">By [Luke Latham](https://github.com/GuardRex) and [John Luo](https://github.com/JunTaoLuo)</span></span>

[<span data-ttu-id="20c0f-106">檢視或下載範例程式碼</span><span class="sxs-lookup"><span data-stu-id="20c0f-106">View or download sample code</span></span>](https://github.com/aspnet/Docs/tree/master/aspnetcore/performance/caching/middleware/samples)

<span data-ttu-id="20c0f-107">本文件提供有關如何設定 ASP.NET Core 應用程式中的回應快取中介軟體的詳細資料。</span><span class="sxs-lookup"><span data-stu-id="20c0f-107">This document provides details on how to configure the Response Caching Middleware in ASP.NET Core apps.</span></span> <span data-ttu-id="20c0f-108">中介軟體會決定當回應的快取、 儲存區回應，以及做回應從快取。</span><span class="sxs-lookup"><span data-stu-id="20c0f-108">The middleware determines when responses are cacheable, stores responses, and serves responses from cache.</span></span> <span data-ttu-id="20c0f-109">如需簡介 HTTP 快取和`ResponseCache`屬性，請參閱[快取回應](response.md)。</span><span class="sxs-lookup"><span data-stu-id="20c0f-109">For an introduction to HTTP caching and the `ResponseCache` attribute, see [Response Caching](response.md).</span></span>

## <a name="package"></a><span data-ttu-id="20c0f-110">Package</span><span class="sxs-lookup"><span data-stu-id="20c0f-110">Package</span></span>
<span data-ttu-id="20c0f-111">若要在專案中包含中介軟體，將參考加入[ `Microsoft.AspNetCore.ResponseCaching` ](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCaching/)封裝，或使用[ `Microsoft.AspNetCore.All` ](https://www.nuget.org/packages/Microsoft.AspNetCore.All/)封裝。</span><span class="sxs-lookup"><span data-stu-id="20c0f-111">To include the middleware in a project, add a reference to the [`Microsoft.AspNetCore.ResponseCaching`](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCaching/) package or use the [`Microsoft.AspNetCore.All`](https://www.nuget.org/packages/Microsoft.AspNetCore.All/) package.</span></span>

## <a name="configuration"></a><span data-ttu-id="20c0f-112">組態</span><span class="sxs-lookup"><span data-stu-id="20c0f-112">Configuration</span></span>
<span data-ttu-id="20c0f-113">在`ConfigureServices`，將中介軟體新增至服務集合。</span><span class="sxs-lookup"><span data-stu-id="20c0f-113">In `ConfigureServices`, add the middleware to the service collection.</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="20c0f-114">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="20c0f-114">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

<span data-ttu-id="20c0f-115">[!code-csharp[Main](middleware/samples/2.x/Program.cs?name=snippet1&highlight=4)]</span><span class="sxs-lookup"><span data-stu-id="20c0f-115">[!code-csharp[Main](middleware/samples/2.x/Program.cs?name=snippet1&highlight=4)]</span></span>

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="20c0f-116">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="20c0f-116">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

<span data-ttu-id="20c0f-117">[!code-csharp[Main](middleware/samples/1.x/Startup.cs?name=snippet1&highlight=3)]</span><span class="sxs-lookup"><span data-stu-id="20c0f-117">[!code-csharp[Main](middleware/samples/1.x/Startup.cs?name=snippet1&highlight=3)]</span></span>

---

<span data-ttu-id="20c0f-118">設定應用程式使用中的介軟體`UseResponseCaching`擴充方法，將中介軟體新增至處理管線的要求。</span><span class="sxs-lookup"><span data-stu-id="20c0f-118">Configure the app to use the middleware with the `UseResponseCaching` extension method, which adds the middleware to the request processing pipeline.</span></span> <span data-ttu-id="20c0f-119">範例應用程式將[ `Cache-Control` ](https://tools.ietf.org/html/rfc7234#section-5.2)標頭至回應的快取的回應會快取多達 10 秒。</span><span class="sxs-lookup"><span data-stu-id="20c0f-119">The sample app adds a [`Cache-Control`](https://tools.ietf.org/html/rfc7234#section-5.2) header to the response that caches cacheable responses for up to 10 seconds.</span></span> <span data-ttu-id="20c0f-120">此範例傳送[ `Vary` ](https://tools.ietf.org/html/rfc7231#section-7.1.4)標頭設定的中介軟體提供快取回的應才[ `Accept-Encoding` ](https://tools.ietf.org/html/rfc7231#section-5.3.4)後續要求的標頭與相符的原始要求。</span><span class="sxs-lookup"><span data-stu-id="20c0f-120">The sample sends a [`Vary`](https://tools.ietf.org/html/rfc7231#section-7.1.4) header to configure the middleware to serve a cached response only if the [`Accept-Encoding`](https://tools.ietf.org/html/rfc7231#section-5.3.4) header of subsequent requests matches that of the original request.</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="20c0f-121">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="20c0f-121">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

<span data-ttu-id="20c0f-122">[!code-csharp[Main](middleware/samples/2.x/Program.cs?name=snippet1&highlight=8)]</span><span class="sxs-lookup"><span data-stu-id="20c0f-122">[!code-csharp[Main](middleware/samples/2.x/Program.cs?name=snippet1&highlight=8)]</span></span>

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="20c0f-123">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="20c0f-123">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

<span data-ttu-id="20c0f-124">[!code-csharp[Main](middleware/samples/1.x/Startup.cs?name=snippet2&highlight=3)]</span><span class="sxs-lookup"><span data-stu-id="20c0f-124">[!code-csharp[Main](middleware/samples/1.x/Startup.cs?name=snippet2&highlight=3)]</span></span>

---

<span data-ttu-id="20c0f-125">回應快取中介軟體只會快取 200 （確定） 伺服器的回應。</span><span class="sxs-lookup"><span data-stu-id="20c0f-125">The Response Caching Middleware only caches 200 (OK) server responses.</span></span> <span data-ttu-id="20c0f-126">任何其他回應，包括[錯誤網頁](xref:fundamentals/error-handling)中, 介軟體會被忽略。</span><span class="sxs-lookup"><span data-stu-id="20c0f-126">Any other responses, including [error pages](xref:fundamentals/error-handling), are ignored by the middleware.</span></span>

> [!WARNING]
> <span data-ttu-id="20c0f-127">回應包含內容的已驗證的用戶端必須標示為不可快取，以防止從儲存和處理這些回應的中介軟體。</span><span class="sxs-lookup"><span data-stu-id="20c0f-127">Responses containing content for authenticated clients must be marked as not cacheable to prevent the middleware from storing and serving those responses.</span></span> <span data-ttu-id="20c0f-128">請參閱[快取的條件](#conditions-for-caching)如需詳細資訊中, 介軟體如何判斷是否可快取的回應。</span><span class="sxs-lookup"><span data-stu-id="20c0f-128">See [Conditions for caching](#conditions-for-caching) for details on how the middleware determines if a response is cacheable.</span></span>

## <a name="options"></a><span data-ttu-id="20c0f-129">選項</span><span class="sxs-lookup"><span data-stu-id="20c0f-129">Options</span></span>
<span data-ttu-id="20c0f-130">中介軟體提供三個選項來控制回應快取。</span><span class="sxs-lookup"><span data-stu-id="20c0f-130">The middleware offers three options for controlling response caching.</span></span>

| <span data-ttu-id="20c0f-131">選項</span><span class="sxs-lookup"><span data-stu-id="20c0f-131">Option</span></span>                | <span data-ttu-id="20c0f-132">預設值</span><span class="sxs-lookup"><span data-stu-id="20c0f-132">Default Value</span></span> |
| --------------------- | ------------- |
| <span data-ttu-id="20c0f-133">UseCaseSensitivePaths</span><span class="sxs-lookup"><span data-stu-id="20c0f-133">UseCaseSensitivePaths</span></span> | <span data-ttu-id="20c0f-134">決定是否區分大小寫的路徑上快取的回應。</span><span class="sxs-lookup"><span data-stu-id="20c0f-134">Determines if responses are cached on case-sensitive paths.</span></span></p><p><span data-ttu-id="20c0f-135">預設值是 `false`。</span><span class="sxs-lookup"><span data-stu-id="20c0f-135">The default value is `false`.</span></span> |
| <span data-ttu-id="20c0f-136">MaximumBodySize</span><span class="sxs-lookup"><span data-stu-id="20c0f-136">MaximumBodySize</span></span>       | <span data-ttu-id="20c0f-137">回應主體，以位元組為單位的最大的可快取大小。</span><span class="sxs-lookup"><span data-stu-id="20c0f-137">The largest cacheable size for the response body in bytes.</span></span></p><span data-ttu-id="20c0f-138">預設值是`64 * 1024 * 1024`(64 MB)。</span><span class="sxs-lookup"><span data-stu-id="20c0f-138">The default value is `64 * 1024 * 1024` (64 MB).</span></span> |
| <span data-ttu-id="20c0f-139">SizeLimit</span><span class="sxs-lookup"><span data-stu-id="20c0f-139">SizeLimit</span></span>             | <span data-ttu-id="20c0f-140">回應快取中介軟體，以位元組為單位的大小限制。</span><span class="sxs-lookup"><span data-stu-id="20c0f-140">The size limit for the response cache middleware in bytes.</span></span> <span data-ttu-id="20c0f-141">預設值是`100 * 1024 * 1024`(100 MB)。</span><span class="sxs-lookup"><span data-stu-id="20c0f-141">The default value is `100 * 1024 * 1024` (100 MB).</span></span> |

<span data-ttu-id="20c0f-142">下列範例會設定來快取回應小於或等於 1024 個位元組，使用區分大小寫的路徑，將儲存至回應的中介軟體`/page1`和`/Page1`分開。</span><span class="sxs-lookup"><span data-stu-id="20c0f-142">The following example configures the middleware to cache responses smaller than or equal to 1,024 bytes using case-sensitive paths, storing the responses to `/page1` and `/Page1` separately.</span></span>

```csharp
services.AddResponseCaching(options =>
{
    options.UseCaseSensitivePaths = true;
    options.MaximumBodySize = 1024;
});
```

## <a name="varybyquerykeys"></a><span data-ttu-id="20c0f-143">VaryByQueryKeys</span><span class="sxs-lookup"><span data-stu-id="20c0f-143">VaryByQueryKeys</span></span>
<span data-ttu-id="20c0f-144">當使用 MVC、`ResponseCache`屬性會指定所需的設定適當的標頭，為快取回應的參數。</span><span class="sxs-lookup"><span data-stu-id="20c0f-144">When using MVC, the `ResponseCache` attribute specifies the parameters necessary for setting appropriate headers for response caching.</span></span> <span data-ttu-id="20c0f-145">唯一的參數`ResponseCache`絕對需要中, 介軟體的屬性是`VaryByQueryKeys`，這並不對應到實際的 HTTP 標頭。</span><span class="sxs-lookup"><span data-stu-id="20c0f-145">The only parameter of the `ResponseCache` attribute that strictly requires the middleware is `VaryByQueryKeys`, which doesn't correspond to an actual HTTP header.</span></span> <span data-ttu-id="20c0f-146">如需詳細資訊，請參閱[ResponseCache 屬性](response.md#responsecache-attribute)。</span><span class="sxs-lookup"><span data-stu-id="20c0f-146">For more information, see [ResponseCache Attribute](response.md#responsecache-attribute).</span></span>

<span data-ttu-id="20c0f-147">當不使用 MVC，您可以變更回應快取`VaryByQueryKeys`功能。</span><span class="sxs-lookup"><span data-stu-id="20c0f-147">When not using MVC, you can vary response caching with the `VaryByQueryKeys` feature.</span></span> <span data-ttu-id="20c0f-148">使用`ResponseCachingFeature`直接從`IFeatureCollection`的`HttpContext`:</span><span class="sxs-lookup"><span data-stu-id="20c0f-148">Use the `ResponseCachingFeature` directly from the `IFeatureCollection` of the `HttpContext`:</span></span>

```csharp
var responseCachingFeature = context.HttpContext.Features.Get<IResponseCachingFeature>();
if (responseCachingFeature != null)
{
    responseCachingFeature.VaryByQueryKeys = new[] { "MyKey" };
}
```

## <a name="http-headers-used-by-response-caching-middleware"></a><span data-ttu-id="20c0f-149">回應快取中介軟體所使用的 HTTP 標頭</span><span class="sxs-lookup"><span data-stu-id="20c0f-149">HTTP headers used by Response Caching Middleware</span></span>
<span data-ttu-id="20c0f-150">中介軟體的快取的回應設定透過 HTTP 標頭。</span><span class="sxs-lookup"><span data-stu-id="20c0f-150">Response caching by the middleware is configured via HTTP headers.</span></span> <span data-ttu-id="20c0f-151">以下列出相關的標頭與它們如何影響快取的注意事項。</span><span class="sxs-lookup"><span data-stu-id="20c0f-151">The relevant headers are listed below with notes on how they affect caching.</span></span>

| <span data-ttu-id="20c0f-152">頁首</span><span class="sxs-lookup"><span data-stu-id="20c0f-152">Header</span></span> | <span data-ttu-id="20c0f-153">詳細資料</span><span class="sxs-lookup"><span data-stu-id="20c0f-153">Details</span></span> |
| ------ | ------- |
| <span data-ttu-id="20c0f-154">授權</span><span class="sxs-lookup"><span data-stu-id="20c0f-154">Authorization</span></span> | <span data-ttu-id="20c0f-155">如果標頭存在，回應不是快取。</span><span class="sxs-lookup"><span data-stu-id="20c0f-155">The response isn't cached if the header exists.</span></span> |
| <span data-ttu-id="20c0f-156">快取控制</span><span class="sxs-lookup"><span data-stu-id="20c0f-156">Cache-Control</span></span> | <span data-ttu-id="20c0f-157">中介軟體只會考慮快取回應標記為`public`快取的指示詞。</span><span class="sxs-lookup"><span data-stu-id="20c0f-157">The middleware only considers caching responses marked with the `public` cache directive.</span></span> <span data-ttu-id="20c0f-158">您可以控制快取包含下列參數：</span><span class="sxs-lookup"><span data-stu-id="20c0f-158">You can control caching with the following parameters:</span></span><ul><li><span data-ttu-id="20c0f-159">保留時間上限</span><span class="sxs-lookup"><span data-stu-id="20c0f-159">max-age</span></span></li><li><span data-ttu-id="20c0f-160">最大過時 &#8224;</span><span class="sxs-lookup"><span data-stu-id="20c0f-160">max-stale&#8224;</span></span></li><li><span data-ttu-id="20c0f-161">min 全新</span><span class="sxs-lookup"><span data-stu-id="20c0f-161">min-fresh</span></span></li><li><span data-ttu-id="20c0f-162">必須指令</span><span class="sxs-lookup"><span data-stu-id="20c0f-162">must-revalidate</span></span></li><li><span data-ttu-id="20c0f-163">無快取</span><span class="sxs-lookup"><span data-stu-id="20c0f-163">no-cache</span></span></li><li><span data-ttu-id="20c0f-164">無存放區</span><span class="sxs-lookup"><span data-stu-id="20c0f-164">no-store</span></span></li><li><span data-ttu-id="20c0f-165">僅限 if-快取</span><span class="sxs-lookup"><span data-stu-id="20c0f-165">only-if-cached</span></span></li><li><span data-ttu-id="20c0f-166">private</span><span class="sxs-lookup"><span data-stu-id="20c0f-166">private</span></span></li><li><span data-ttu-id="20c0f-167">public</span><span class="sxs-lookup"><span data-stu-id="20c0f-167">public</span></span></li><li><span data-ttu-id="20c0f-168">s maxage</span><span class="sxs-lookup"><span data-stu-id="20c0f-168">s-maxage</span></span></li><li><span data-ttu-id="20c0f-169">proxy 指令 &#8225;</span><span class="sxs-lookup"><span data-stu-id="20c0f-169">proxy-revalidate&#8225;</span></span></li></ul><span data-ttu-id="20c0f-170">&#8224; 若要指定無限制到`max-stale`中, 介軟體會採取任何動作。</span><span class="sxs-lookup"><span data-stu-id="20c0f-170">&#8224;If no limit is specified to `max-stale`, the middleware takes no action.</span></span><br><span data-ttu-id="20c0f-171">&#8225;`proxy-revalidate`具有相同的效果`must-revalidate`。</span><span class="sxs-lookup"><span data-stu-id="20c0f-171">&#8225;`proxy-revalidate` has the same effect as `must-revalidate`.</span></span><br><br><span data-ttu-id="20c0f-172">如需詳細資訊，請參閱[RFC 7231： 要求的快取控制指示詞](https://tools.ietf.org/html/rfc7234#section-5.2.1)。</span><span class="sxs-lookup"><span data-stu-id="20c0f-172">For more information, see [RFC 7231: Request Cache-Control Directives](https://tools.ietf.org/html/rfc7234#section-5.2.1).</span></span> |
| <span data-ttu-id="20c0f-173">Pragma</span><span class="sxs-lookup"><span data-stu-id="20c0f-173">Pragma</span></span> | <span data-ttu-id="20c0f-174">A`Pragma: no-cache`要求標頭會產生相同的效果`Cache-Control: no-cache`。</span><span class="sxs-lookup"><span data-stu-id="20c0f-174">A `Pragma: no-cache` header in the request produces the same effect as `Cache-Control: no-cache`.</span></span> <span data-ttu-id="20c0f-175">此標頭中的相關指示詞會覆寫`Cache-Control`標頭，如果有的話。</span><span class="sxs-lookup"><span data-stu-id="20c0f-175">This header is overridden by the relevant directives in the `Cache-Control` header, if present.</span></span> <span data-ttu-id="20c0f-176">與 HTTP/1.0 回溯相容性考量。</span><span class="sxs-lookup"><span data-stu-id="20c0f-176">Considered for backward compatibility with HTTP/1.0.</span></span> |
| <span data-ttu-id="20c0f-177">設定 Cookie</span><span class="sxs-lookup"><span data-stu-id="20c0f-177">Set-Cookie</span></span> | <span data-ttu-id="20c0f-178">如果標頭存在，回應不是快取。</span><span class="sxs-lookup"><span data-stu-id="20c0f-178">The response isn't cached if the header exists.</span></span> |
| <span data-ttu-id="20c0f-179">而有所不同</span><span class="sxs-lookup"><span data-stu-id="20c0f-179">Vary</span></span> | <span data-ttu-id="20c0f-180">`Vary`標頭由另一個標頭用來變更快取的回應。</span><span class="sxs-lookup"><span data-stu-id="20c0f-180">The `Vary` header is used to vary the cached response by another header.</span></span> <span data-ttu-id="20c0f-181">比方說，您可以快取回應編碼包含`Vary: Accept-Encoding`快取回應之要求標頭的標頭`Accept-Encoding: gzip`和`Accept-Encoding: text/plain`分開。</span><span class="sxs-lookup"><span data-stu-id="20c0f-181">For example, you can cache responses by encoding by including the `Vary: Accept-Encoding` header, which caches responses for requests with headers `Accept-Encoding: gzip` and `Accept-Encoding: text/plain` separately.</span></span> <span data-ttu-id="20c0f-182">回應標頭值是`*`絕對不會儲存。</span><span class="sxs-lookup"><span data-stu-id="20c0f-182">A response with a header value of `*` is never stored.</span></span> |
| <span data-ttu-id="20c0f-183">到期</span><span class="sxs-lookup"><span data-stu-id="20c0f-183">Expires</span></span> | <span data-ttu-id="20c0f-184">此標頭視為過時的回應不是存放或擷取除非覆寫其他`Cache-Control`標頭。</span><span class="sxs-lookup"><span data-stu-id="20c0f-184">A response deemed stale by this header isn't stored or retrieved unless overridden by other `Cache-Control` headers.</span></span> |
| <span data-ttu-id="20c0f-185">如果 If-none-Match</span><span class="sxs-lookup"><span data-stu-id="20c0f-185">If-None-Match</span></span> | <span data-ttu-id="20c0f-186">如果此值不是從快取提供完整的回應`*`和`ETag`的回應不符合任何提供的值。</span><span class="sxs-lookup"><span data-stu-id="20c0f-186">The full response is served from cache if the value isn't `*` and the `ETag` of the response doesn't match any of the values provided.</span></span> <span data-ttu-id="20c0f-187">否則，提供 304 （未修改） 的回應。</span><span class="sxs-lookup"><span data-stu-id="20c0f-187">Otherwise, a 304 (Not Modified) response is served.</span></span> |
| <span data-ttu-id="20c0f-188">如果修改自</span><span class="sxs-lookup"><span data-stu-id="20c0f-188">If-Modified-Since</span></span> | <span data-ttu-id="20c0f-189">如果`If-None-Match`標頭不存在，如果快取的回應日期比所提供的值從快取提供完整的回應。</span><span class="sxs-lookup"><span data-stu-id="20c0f-189">If the `If-None-Match` header isn't present, a full response is served from cache if the cached response date is newer than the value provided.</span></span> <span data-ttu-id="20c0f-190">否則，提供 304 （未修改） 的回應。</span><span class="sxs-lookup"><span data-stu-id="20c0f-190">Otherwise, a 304 (Not Modified) response is served.</span></span> |
| <span data-ttu-id="20c0f-191">日期</span><span class="sxs-lookup"><span data-stu-id="20c0f-191">Date</span></span> | <span data-ttu-id="20c0f-192">當服務從快取，`Date`若未提供原始回應上設定標頭中介軟體。</span><span class="sxs-lookup"><span data-stu-id="20c0f-192">When serving from cache, the `Date` header is set by the middleware if it wasn't provided on the original response.</span></span> |
| <span data-ttu-id="20c0f-193">內容長度</span><span class="sxs-lookup"><span data-stu-id="20c0f-193">Content-Length</span></span> | <span data-ttu-id="20c0f-194">當服務從快取，`Content-Length`若未提供原始回應上設定標頭中介軟體。</span><span class="sxs-lookup"><span data-stu-id="20c0f-194">When serving from cache, the `Content-Length` header is set by the middleware if it wasn't provided on the original response.</span></span> |
| <span data-ttu-id="20c0f-195">存留期</span><span class="sxs-lookup"><span data-stu-id="20c0f-195">Age</span></span> | <span data-ttu-id="20c0f-196">`Age`原始回應中傳送的標頭會被忽略。</span><span class="sxs-lookup"><span data-stu-id="20c0f-196">The `Age` header sent in the original response is ignored.</span></span> <span data-ttu-id="20c0f-197">當服務快取的回應中, 介軟體會計算新值。</span><span class="sxs-lookup"><span data-stu-id="20c0f-197">The middleware computes a new value when serving a cached response.</span></span> |

## <a name="troubleshooting"></a><span data-ttu-id="20c0f-198">疑難排解</span><span class="sxs-lookup"><span data-stu-id="20c0f-198">Troubleshooting</span></span>
<span data-ttu-id="20c0f-199">如果快取行為是未如預期般，，確認回應的快取，並且可以從快取由檢查要求的連入標頭和回應的傳出標頭。</span><span class="sxs-lookup"><span data-stu-id="20c0f-199">If caching behavior isn't as you expect, confirm that responses are cacheable and capable of being served from the cache by examining the request's incoming headers and the response's outgoing headers.</span></span> <span data-ttu-id="20c0f-200">啟用[記錄](xref:fundamentals/logging)有助於偵錯時。</span><span class="sxs-lookup"><span data-stu-id="20c0f-200">Enabling [logging](xref:fundamentals/logging) can help when debugging.</span></span> <span data-ttu-id="20c0f-201">快取行為，並回應從快取的擷取時此中介軟體記錄。</span><span class="sxs-lookup"><span data-stu-id="20c0f-201">The middleware logs caching behavior and when a response is retrieved from cache.</span></span>

<span data-ttu-id="20c0f-202">當測試及疑難排解快取行為，在瀏覽器可能設定要求標頭之影響快取不想要的方式。</span><span class="sxs-lookup"><span data-stu-id="20c0f-202">When testing and troubleshooting caching behavior, a browser may set request headers that affect caching in undesirable ways.</span></span> <span data-ttu-id="20c0f-203">例如，可能會設定瀏覽器`Cache-Control`標頭`no-cache`時重新整理頁面。</span><span class="sxs-lookup"><span data-stu-id="20c0f-203">For example, a browser may set the `Cache-Control` header to `no-cache` when you refresh the page.</span></span> <span data-ttu-id="20c0f-204">下列工具可以明確地將要求標頭，以及慣用發佈點進行測試快取：</span><span class="sxs-lookup"><span data-stu-id="20c0f-204">The following tools can explicitly set request headers, and are preferred for testing caching:</span></span>

* [<span data-ttu-id="20c0f-205">Fiddler</span><span class="sxs-lookup"><span data-stu-id="20c0f-205">Fiddler</span></span>](http://www.telerik.com/fiddler)
* [<span data-ttu-id="20c0f-206">Firebug 這類</span><span class="sxs-lookup"><span data-stu-id="20c0f-206">Firebug</span></span>](http://getfirebug.com/)
* [<span data-ttu-id="20c0f-207">郵差</span><span class="sxs-lookup"><span data-stu-id="20c0f-207">Postman</span></span>](https://www.getpostman.com/)

### <a name="conditions-for-caching"></a><span data-ttu-id="20c0f-208">快取的條件</span><span class="sxs-lookup"><span data-stu-id="20c0f-208">Conditions for caching</span></span>
* <span data-ttu-id="20c0f-209">要求必須導致伺服器 200 （確定） 回應。</span><span class="sxs-lookup"><span data-stu-id="20c0f-209">The request must result in a 200 (OK) response from the server.</span></span>
* <span data-ttu-id="20c0f-210">要求方法必須是 GET 或 HEAD。</span><span class="sxs-lookup"><span data-stu-id="20c0f-210">The request method must be GET or HEAD.</span></span>
* <span data-ttu-id="20c0f-211">終端機中的介軟體，例如靜態檔案中介軟體，必須處理前回應快取中介軟體的回應。</span><span class="sxs-lookup"><span data-stu-id="20c0f-211">Terminal middleware, such as Static File Middleware, must not process the response prior to the Response Caching Middleware.</span></span>
* <span data-ttu-id="20c0f-212">`Authorization`標頭不得存在。</span><span class="sxs-lookup"><span data-stu-id="20c0f-212">The `Authorization` header must not be present.</span></span>
* <span data-ttu-id="20c0f-213">`Cache-Control`標頭參數必須是有效，而且必須標示為回應`public`且未標記為`private`。</span><span class="sxs-lookup"><span data-stu-id="20c0f-213">`Cache-Control` header parameters must be valid, and the response must be marked `public` and not marked `private`.</span></span>
* <span data-ttu-id="20c0f-214">`Pragma: no-cache`標頭/值不能存在如果`Cache-Control`標頭不是呈現為`Cache-Control`標頭會覆寫`Pragma`標頭時出現。</span><span class="sxs-lookup"><span data-stu-id="20c0f-214">The `Pragma: no-cache` header/value must not be present if the `Cache-Control` header isn't present, as the `Cache-Control` header overrides the `Pragma` header when present.</span></span>
* <span data-ttu-id="20c0f-215">`Set-Cookie`標頭不得存在。</span><span class="sxs-lookup"><span data-stu-id="20c0f-215">The `Set-Cookie` header must not be present.</span></span>
* <span data-ttu-id="20c0f-216">`Vary`標頭參數必須是有效且不等於`*`。</span><span class="sxs-lookup"><span data-stu-id="20c0f-216">`Vary` header parameters must be valid and not equal to `*`.</span></span>
* <span data-ttu-id="20c0f-217">`Content-Length`標頭值 (如果設定) 必須符合回應主體的大小。</span><span class="sxs-lookup"><span data-stu-id="20c0f-217">The `Content-Length` header value (if set) must match the size of the response body.</span></span>
* <span data-ttu-id="20c0f-218">`HttpSendFileFeature`不會使用。</span><span class="sxs-lookup"><span data-stu-id="20c0f-218">The `HttpSendFileFeature` isn't used.</span></span>
* <span data-ttu-id="20c0f-219">回應不是所指定過時`Expires`標頭和`max-age`和`s-maxage`快取指示詞。</span><span class="sxs-lookup"><span data-stu-id="20c0f-219">The response must not be stale as specified by the `Expires` header and the `max-age` and `s-maxage` cache directives.</span></span>
* <span data-ttu-id="20c0f-220">回應緩衝處理是否成功，並回應的大小會小於所設定，或預設`SizeLimit`。</span><span class="sxs-lookup"><span data-stu-id="20c0f-220">Response buffering is successful, and the size of the response is smaller than the configured or default `SizeLimit`.</span></span>
* <span data-ttu-id="20c0f-221">回應必須是根據可快取[RFC 7234](https://tools.ietf.org/html/rfc7234)規格。</span><span class="sxs-lookup"><span data-stu-id="20c0f-221">The response must be cacheable according to the [RFC 7234](https://tools.ietf.org/html/rfc7234) specifications.</span></span> <span data-ttu-id="20c0f-222">例如，`no-store`指示詞不能存在於要求或回應標頭欄位。</span><span class="sxs-lookup"><span data-stu-id="20c0f-222">For example, the `no-store` directive must not exist in request or response header fields.</span></span> <span data-ttu-id="20c0f-223">請參閱*區段 3： 儲存在快取的回應*的[RFC 7234](https://tools.ietf.org/html/rfc7234)如需詳細資訊。</span><span class="sxs-lookup"><span data-stu-id="20c0f-223">See *Section 3: Storing Responses in Caches* of [RFC 7234](https://tools.ietf.org/html/rfc7234) for details.</span></span>

> [!NOTE]
> <span data-ttu-id="20c0f-224">Antiforgery 系統是否有產生安全性權杖，以防止跨站台要求偽造 (CSRF) 攻擊集`Cache-Control`和`Pragma`標頭`no-cache`以便回應未快取。</span><span class="sxs-lookup"><span data-stu-id="20c0f-224">The Antiforgery system for generating secure tokens to prevent Cross-Site Request Forgery (CSRF) attacks sets the `Cache-Control` and `Pragma` headers to `no-cache` so that responses aren't cached.</span></span>

## <a name="additional-resources"></a><span data-ttu-id="20c0f-225">其他資源</span><span class="sxs-lookup"><span data-stu-id="20c0f-225">Additional resources</span></span>

* [<span data-ttu-id="20c0f-226">應用程式啟動</span><span class="sxs-lookup"><span data-stu-id="20c0f-226">Application Startup</span></span>](xref:fundamentals/startup)
* [<span data-ttu-id="20c0f-227">中介軟體</span><span class="sxs-lookup"><span data-stu-id="20c0f-227">Middleware</span></span>](xref:fundamentals/middleware)