---
title: 整合測試中 ASP.NET Core
author: guardrex
description: 了解如何整合測試，確保應用程式的元件包括資料庫、 檔案系統和網路基礎結構層級正確運作。
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 05/30/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: test/integration-tests
ms.openlocfilehash: a402c3f5f6a75917eba4058e6cc6926f25b214d4
ms.sourcegitcommit: 726ffab258070b4fe6cf950bf030ce10c0c07bb4
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/04/2018
ms.locfileid: "35217586"
---
# <a name="integration-tests-in-aspnet-core"></a><span data-ttu-id="df49f-103">整合測試中 ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="df49f-103">Integration tests in ASP.NET Core</span></span>

<span data-ttu-id="df49f-104">由[Luke Latham](https://github.com/guardrex)和[Steve Smith](https://ardalis.com/)</span><span class="sxs-lookup"><span data-stu-id="df49f-104">By [Luke Latham](https://github.com/guardrex) and [Steve Smith](https://ardalis.com/)</span></span>

<span data-ttu-id="df49f-105">整合測試，確保應用程式的元件正確運作，其中包含應用程式的支援基礎結構，例如資料庫、 檔案系統和網路層級。</span><span class="sxs-lookup"><span data-stu-id="df49f-105">Integration tests ensure that an app's components function correctly at a level that includes the app's supporting infrastructure, such as the database, file system, and network.</span></span> <span data-ttu-id="df49f-106">ASP.NET Core 支援整合測試與測試 web 主機和記憶體中的測試伺服器使用的單元測試架構。</span><span class="sxs-lookup"><span data-stu-id="df49f-106">ASP.NET Core supports integration tests using a unit test framework with a test web host and an in-memory test server.</span></span>

<span data-ttu-id="df49f-107">本主題假設單元測試的基本了的解。</span><span class="sxs-lookup"><span data-stu-id="df49f-107">This topic assumes a basic understanding of unit tests.</span></span> <span data-ttu-id="df49f-108">如果不熟悉測試概念，請參閱[.NET Core 和標準.NET 中的單元測試](/dotnet/core/testing/)主題和連結的內容。</span><span class="sxs-lookup"><span data-stu-id="df49f-108">If unfamiliar with test concepts, see the [Unit Testing in .NET Core and .NET Standard](/dotnet/core/testing/) topic and its linked content.</span></span>

<span data-ttu-id="df49f-109">[檢視或下載範例程式碼](https://github.com/aspnet/Docs/tree/master/aspnetcore/test/integration-tests/samples) \(英文\) ([如何下載](xref:tutorials/index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="df49f-109">[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/test/integration-tests/samples) ([how to download](xref:tutorials/index#how-to-download-a-sample))</span></span>

<span data-ttu-id="df49f-110">範例應用程式是 Razor 頁面的應用程式，並假設 Razor 頁面的基本知識。</span><span class="sxs-lookup"><span data-stu-id="df49f-110">The sample app is a Razor Pages app and assumes a basic understanding of Razor Pages.</span></span> <span data-ttu-id="df49f-111">如果不熟悉使用 Razor 頁面，請參閱下列主題：</span><span class="sxs-lookup"><span data-stu-id="df49f-111">If unfamiliar with Razor Pages, see the following topics:</span></span>

* [<span data-ttu-id="df49f-112">Razor 頁面簡介</span><span class="sxs-lookup"><span data-stu-id="df49f-112">Introduction to Razor Pages</span></span>](xref:mvc/razor-pages/index)
* [<span data-ttu-id="df49f-113">開始使用 Razor Pages</span><span class="sxs-lookup"><span data-stu-id="df49f-113">Get started with Razor Pages</span></span>](xref:tutorials/razor-pages/razor-pages-start)
* [<span data-ttu-id="df49f-114">Razor 頁面單元測試</span><span class="sxs-lookup"><span data-stu-id="df49f-114">Razor Pages unit tests</span></span>](xref:test/razor-pages-tests)

## <a name="introduction-to-integration-tests"></a><span data-ttu-id="df49f-115">整合測試的簡介</span><span class="sxs-lookup"><span data-stu-id="df49f-115">Introduction to integration tests</span></span>

<span data-ttu-id="df49f-116">整合測試評估應用程式的元件，比在廣泛層級[單元測試](/dotnet/core/testing/)。</span><span class="sxs-lookup"><span data-stu-id="df49f-116">Integration tests evaluate an app's components on a broader level than [unit tests](/dotnet/core/testing/).</span></span> <span data-ttu-id="df49f-117">單元測試用來測試隔離的軟體元件，例如個別類別方法。</span><span class="sxs-lookup"><span data-stu-id="df49f-117">Unit tests are used to test isolated software components, such as individual class methods.</span></span> <span data-ttu-id="df49f-118">整合測試確認兩個或多個應用程式元件一起運作以產生預期的結果，可能會包含每個完整處理要求所需的元件。</span><span class="sxs-lookup"><span data-stu-id="df49f-118">Integration tests confirm that two or more app components work together to produce an expected result, possibly including every component required to fully process a request.</span></span>

<span data-ttu-id="df49f-119">這些較廣泛測試用來測試應用程式的基礎結構與整個架構，通常包括下列元件：</span><span class="sxs-lookup"><span data-stu-id="df49f-119">These broader tests are used to test the app's infrastructure and whole framework, often including the following components:</span></span>

* <span data-ttu-id="df49f-120">資料庫</span><span class="sxs-lookup"><span data-stu-id="df49f-120">Database</span></span>
* <span data-ttu-id="df49f-121">檔案系統</span><span class="sxs-lookup"><span data-stu-id="df49f-121">File system</span></span>
* <span data-ttu-id="df49f-122">網路應用裝置</span><span class="sxs-lookup"><span data-stu-id="df49f-122">Network appliances</span></span>
* <span data-ttu-id="df49f-123">要求-回應管線</span><span class="sxs-lookup"><span data-stu-id="df49f-123">Request-response pipeline</span></span>

<span data-ttu-id="df49f-124">單元測試使用傳遞元件，又稱為*fakes*或*模擬物件*，來取代基礎結構的元件。</span><span class="sxs-lookup"><span data-stu-id="df49f-124">Unit tests use fabricated components, known as *fakes* or *mock objects*, in place of infrastructure components.</span></span>

<span data-ttu-id="df49f-125">相較於單元測試，測試整合：</span><span class="sxs-lookup"><span data-stu-id="df49f-125">In contrast to unit tests, integration tests:</span></span>

* <span data-ttu-id="df49f-126">使用應用程式在生產環境中使用的實際元件。</span><span class="sxs-lookup"><span data-stu-id="df49f-126">Use the actual components that the app uses in production.</span></span>
* <span data-ttu-id="df49f-127">需要更多的程式碼及資料處理。</span><span class="sxs-lookup"><span data-stu-id="df49f-127">Require more code and data processing.</span></span>
* <span data-ttu-id="df49f-128">需要更長時間才能執行。</span><span class="sxs-lookup"><span data-stu-id="df49f-128">Take longer to run.</span></span>

<span data-ttu-id="df49f-129">因此，會限制使用的整合測試，將最重要的基礎結構案例。</span><span class="sxs-lookup"><span data-stu-id="df49f-129">Therefore, limit the use of integration tests to the most important infrastructure scenarios.</span></span> <span data-ttu-id="df49f-130">如果可以使用單元測試或整合測試，測試行為，請選擇 單元測試。</span><span class="sxs-lookup"><span data-stu-id="df49f-130">If a behavior can be tested using either a unit test or an integration test, choose the unit test.</span></span>

> [!TIP]
> <span data-ttu-id="df49f-131">不要撰寫每個可能的排列的資料與檔案存取資料庫和檔案系統的整合測試。</span><span class="sxs-lookup"><span data-stu-id="df49f-131">Don't write integration tests for every possible permutation of data and file access with databases and file systems.</span></span> <span data-ttu-id="df49f-132">不論多少會放在應用程式互動的資料庫和檔案系統時，已取得焦點的讀取、 寫入、 更新和刪除整合測試通常能夠適當測試資料庫和檔案系統元件的集合。</span><span class="sxs-lookup"><span data-stu-id="df49f-132">Regardless of how many places across an app interact with databases and file systems, a focused set of read, write, update, and delete integration tests are usually capable of adequately testing database and file system components.</span></span> <span data-ttu-id="df49f-133">使用單元測試方法的邏輯的常式與這些元件互動的測試。</span><span class="sxs-lookup"><span data-stu-id="df49f-133">Use unit tests for routine tests of method logic that interact with these components.</span></span> <span data-ttu-id="df49f-134">在單元測試中的基礎結構使用 fakes/模擬更快的測試執行中的結果。</span><span class="sxs-lookup"><span data-stu-id="df49f-134">In unit tests, the use of infrastructure fakes/mocks result in faster test execution.</span></span>

> [!NOTE]
> <span data-ttu-id="df49f-135">在討論中的整合測試，測試的專案經常稱為*待測系統*，或簡稱為 「 SUT"。</span><span class="sxs-lookup"><span data-stu-id="df49f-135">In discussions of integration tests, the tested project is frequently called the *system under test*, or "SUT" for short.</span></span>

## <a name="aspnet-core-integration-tests"></a><span data-ttu-id="df49f-136">ASP.NET Core 整合測試</span><span class="sxs-lookup"><span data-stu-id="df49f-136">ASP.NET Core integration tests</span></span>

<span data-ttu-id="df49f-137">整合測試中 ASP.NET Core 需要以下各項：</span><span class="sxs-lookup"><span data-stu-id="df49f-137">Integration tests in ASP.NET Core require the following:</span></span>

* <span data-ttu-id="df49f-138">測試專案用來包含及執行測試。</span><span class="sxs-lookup"><span data-stu-id="df49f-138">A test project is used to contain and execute the tests.</span></span> <span data-ttu-id="df49f-139">測試專案有已測試的 ASP.NET Core 專案，稱為參考*待測系統*(SUT)。</span><span class="sxs-lookup"><span data-stu-id="df49f-139">The test project has a reference to the tested ASP.NET Core project, called the *system under test* (SUT).</span></span> <span data-ttu-id="df49f-140">_"SUT 」 在本主題中用於測試應用程式的參考。_</span><span class="sxs-lookup"><span data-stu-id="df49f-140">_"SUT" is used throughout this topic to refer to the tested app._</span></span>
* <span data-ttu-id="df49f-141">測試專案建立 SUT 的測試 web 主機，並處理要求和回應傳給 SUT 使用測試伺服器的用戶端。</span><span class="sxs-lookup"><span data-stu-id="df49f-141">The test project creates a test web host for the SUT and uses a test server client to handle requests and responses to the SUT.</span></span>
* <span data-ttu-id="df49f-142">測試執行器用來執行測試和報告測試結果。</span><span class="sxs-lookup"><span data-stu-id="df49f-142">A test runner is used to execute the tests and report the test results.</span></span>

<span data-ttu-id="df49f-143">整合測試，請依照下列一連串的事件，其中包含一般*排列*， *Act*，和*Assert*測試步驟：</span><span class="sxs-lookup"><span data-stu-id="df49f-143">Integration tests follow a sequence of events that include the usual *Arrange*, *Act*, and *Assert* test steps:</span></span>

1. <span data-ttu-id="df49f-144">SUT web 主機進行設定。</span><span class="sxs-lookup"><span data-stu-id="df49f-144">The SUT's web host is configured.</span></span>
1. <span data-ttu-id="df49f-145">測試伺服器的用戶端會提交要求給應用程式建立。</span><span class="sxs-lookup"><span data-stu-id="df49f-145">A test server client is created to submit requests to the app.</span></span>
1. <span data-ttu-id="df49f-146">*排列*測試步驟執行： 測試應用程式會準備要求。</span><span class="sxs-lookup"><span data-stu-id="df49f-146">The *Arrange* test step is executed: The test app prepares a request.</span></span>
1. <span data-ttu-id="df49f-147">*Act*測試步驟執行： 用戶端會提交要求並接收回應。</span><span class="sxs-lookup"><span data-stu-id="df49f-147">The *Act* test step is executed: The client submits the request and receives the response.</span></span>
1. <span data-ttu-id="df49f-148">*Assert*測試步驟執行：*實際*回應驗證為*傳遞*或*失敗*根據*預期*回應。</span><span class="sxs-lookup"><span data-stu-id="df49f-148">The *Assert* test step is executed: The *actual* response is validated as a *pass* or *fail* based on an *expected* response.</span></span>
1. <span data-ttu-id="df49f-149">所有測試執行之前，會繼續處理程序。</span><span class="sxs-lookup"><span data-stu-id="df49f-149">The process continues until all of the tests are executed.</span></span>
1. <span data-ttu-id="df49f-150">會報告測試結果。</span><span class="sxs-lookup"><span data-stu-id="df49f-150">The test results are reported.</span></span>

<span data-ttu-id="df49f-151">通常，測試 web 主機已設定以不同的方式比執行測試的應用程式的一般 web 主機。</span><span class="sxs-lookup"><span data-stu-id="df49f-151">Usually, the test web host is configured differently than the app's normal web host for the test runs.</span></span> <span data-ttu-id="df49f-152">例如，不同的資料庫或不同的應用程式設定可能會用來測試。</span><span class="sxs-lookup"><span data-stu-id="df49f-152">For example, a different database or different app settings might be used for the tests.</span></span>

<span data-ttu-id="df49f-153">基礎結構元件，例如測試 web 主機和記憶體中的測試伺服器 ([TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver))、 提供或由[Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing)封裝。</span><span class="sxs-lookup"><span data-stu-id="df49f-153">Infrastructure components, such as the test web host and in-memory test server ([TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver)), are provided or managed by the [Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing) package.</span></span> <span data-ttu-id="df49f-154">使用此封裝來簡化測試建立和執行。</span><span class="sxs-lookup"><span data-stu-id="df49f-154">Use of this package streamlines test creation and execution.</span></span>

<span data-ttu-id="df49f-155">`Microsoft.AspNetCore.Mvc.Testing`封裝處理下列工作：</span><span class="sxs-lookup"><span data-stu-id="df49f-155">The `Microsoft.AspNetCore.Mvc.Testing` package handles the following tasks:</span></span>

* <span data-ttu-id="df49f-156">複製相依性的檔案 (*\*.deps*) 從測試專案中的 SUT *bin*資料夾。</span><span class="sxs-lookup"><span data-stu-id="df49f-156">Copies the dependencies file (*\*.deps*) from the SUT into the test project's *bin* folder.</span></span>
* <span data-ttu-id="df49f-157">設定內容的根為 SUT 專案根目錄上，如此靜態檔案和頁面/檢視表位於測試執行時。</span><span class="sxs-lookup"><span data-stu-id="df49f-157">Sets the content root to the SUT's project root so that static files and pages/views are found when the tests are executed.</span></span>
* <span data-ttu-id="df49f-158">提供[WebApplicationFactory](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1)類別來簡化啟動載入與 SUT `TestServer`。</span><span class="sxs-lookup"><span data-stu-id="df49f-158">Provides the [WebApplicationFactory](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1) class to streamline bootstrapping the SUT with `TestServer`.</span></span>

<span data-ttu-id="df49f-159">[單元測試](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)文件說明如何設定測試專案和測試執行器，以及如何執行測試和建議的方式來測試名稱和測試類別的詳細指示。</span><span class="sxs-lookup"><span data-stu-id="df49f-159">The [unit tests](/dotnet/articles/core/testing/unit-testing-with-dotnet-test) documentation describes how to set up a test project and test runner, along with detailed instructions on how to run tests and recommendations for how to name tests and test classes.</span></span>

> [!NOTE]
> <span data-ttu-id="df49f-160">建立應用程式的測試專案時，分開單元測試整合測試分成不同的專案。</span><span class="sxs-lookup"><span data-stu-id="df49f-160">When creating a test project for an app, separate the unit tests from the integration tests into different projects.</span></span> <span data-ttu-id="df49f-161">這有助於確保基礎結構測試元件是的意外地包括在單元測試。</span><span class="sxs-lookup"><span data-stu-id="df49f-161">This helps ensure that infrastructure testing components aren't accidently included in the unit tests.</span></span> <span data-ttu-id="df49f-162">隔離的單位和整合測試也允許控制哪些資料集的測試會執行。</span><span class="sxs-lookup"><span data-stu-id="df49f-162">Separation of unit and integration tests also allows control over which set of tests are run.</span></span>

<span data-ttu-id="df49f-163">不沒有幾乎任何 Razor 網頁應用程式的測試組態和 MVC 應用程式之間的差異。</span><span class="sxs-lookup"><span data-stu-id="df49f-163">There's virtually no difference between the configuration for tests of Razor Pages apps and MVC apps.</span></span> <span data-ttu-id="df49f-164">唯一的差異是在測試命名方式。</span><span class="sxs-lookup"><span data-stu-id="df49f-164">The only difference is in how the tests are named.</span></span> <span data-ttu-id="df49f-165">Razor 網頁應用程式中測試的頁面端點通常命名為之後的頁面模型類別 (例如，`IndexPageTests`來測試索引頁的元件整合)。</span><span class="sxs-lookup"><span data-stu-id="df49f-165">In a Razor Pages app, tests of page endpoints are usually named after the page model class (for example, `IndexPageTests` to test component integration for the Index page).</span></span> <span data-ttu-id="df49f-166">在 MVC 應用程式中測試通常依控制器類別，而且之後的控制站，其會測試名為 (例如，`HomeControllerTests`測試主控制器元件整合)。</span><span class="sxs-lookup"><span data-stu-id="df49f-166">In an MVC app, tests are usually organized by controller classes and named after the controllers they test (for example, `HomeControllerTests` to test component integration for the Home controller).</span></span>

## <a name="test-app-prerequisites"></a><span data-ttu-id="df49f-167">測試應用程式的必要條件</span><span class="sxs-lookup"><span data-stu-id="df49f-167">Test app prerequisites</span></span>

<span data-ttu-id="df49f-168">測試專案必須：</span><span class="sxs-lookup"><span data-stu-id="df49f-168">The test project must:</span></span>

* <span data-ttu-id="df49f-169">針對封裝參考[Microsoft.AspNetCore.App](https://www.nuget.org/packages/Microsoft.AspNetCore.App/)。</span><span class="sxs-lookup"><span data-stu-id="df49f-169">Have a package reference for [Microsoft.AspNetCore.App](https://www.nuget.org/packages/Microsoft.AspNetCore.App/).</span></span>
* <span data-ttu-id="df49f-170">在專案檔中使用 Web SDK (`<Project Sdk="Microsoft.NET.Sdk.Web">`)。</span><span class="sxs-lookup"><span data-stu-id="df49f-170">Use the Web SDK in the project file (`<Project Sdk="Microsoft.NET.Sdk.Web">`).</span></span>

<span data-ttu-id="df49f-171">中可以看到這些 prerequesities[範例應用程式](https://github.com/aspnet/Docs/tree/master/aspnetcore/test/integration-tests/samples/)。</span><span class="sxs-lookup"><span data-stu-id="df49f-171">These prerequesities can be seen in the [sample app](https://github.com/aspnet/Docs/tree/master/aspnetcore/test/integration-tests/samples/).</span></span> <span data-ttu-id="df49f-172">檢查*tests/RazorPagesProject.Tests/RazorPagesProject.Tests.csproj*檔案。</span><span class="sxs-lookup"><span data-stu-id="df49f-172">Inspect the *tests/RazorPagesProject.Tests/RazorPagesProject.Tests.csproj* file.</span></span>

## <a name="basic-tests-with-the-default-webapplicationfactory"></a><span data-ttu-id="df49f-173">預設值 WebApplicationFactory 基本測試</span><span class="sxs-lookup"><span data-stu-id="df49f-173">Basic tests with the default WebApplicationFactory</span></span>

<span data-ttu-id="df49f-174">[WebApplicationFactory&lt;TEntryPoint&gt; ](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1)用來建立[TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver)整合測試。</span><span class="sxs-lookup"><span data-stu-id="df49f-174">[WebApplicationFactory&lt;TEntryPoint&gt;](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1) is used to create a [TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver) for the integration tests.</span></span> <span data-ttu-id="df49f-175">`TEntryPoint` 通常是 SUT 的進入點類別`Startup`類別。</span><span class="sxs-lookup"><span data-stu-id="df49f-175">`TEntryPoint` is the entry point class of the SUT, usually the `Startup` class.</span></span>

<span data-ttu-id="df49f-176">測試類別會實作*類別裝置*介面 (`IClassFixture`) 來指示此類別包含測試，並且提供跨不同測試類別中的共用的物件執行個體。</span><span class="sxs-lookup"><span data-stu-id="df49f-176">Test classes implement a *class fixture* interface (`IClassFixture`) to indicate the class contains tests and provide shared object instances across the tests in the class.</span></span>

### <a name="basic-test-of-app-endpoints"></a><span data-ttu-id="df49f-177">基本測試應用程式端點</span><span class="sxs-lookup"><span data-stu-id="df49f-177">Basic test of app endpoints</span></span>

<span data-ttu-id="df49f-178">下列測試類別， `BasicTests`，會使用`WebApplicationFactory`啟動 SUT 並提供[HttpClient](/dotnet/api/system.net.http.httpclient)至測試方法， `Get_EndpointsReturnSuccessAndCorrectContentType`。</span><span class="sxs-lookup"><span data-stu-id="df49f-178">The following test class, `BasicTests`, uses the `WebApplicationFactory` to bootstrap the SUT and provide an [HttpClient](/dotnet/api/system.net.http.httpclient) to a test method, `Get_EndpointsReturnSuccessAndCorrectContentType`.</span></span> <span data-ttu-id="df49f-179">這個方法會檢查是否成功的回應狀態碼 （200-299 範圍中的狀態碼） 和`Content-Type`標頭是`text/html; charset=utf-8`數個應用程式頁面。</span><span class="sxs-lookup"><span data-stu-id="df49f-179">The method checks if the response status code is successful (status codes in the range 200-299) and the `Content-Type` header is `text/html; charset=utf-8` for several app pages.</span></span>

<span data-ttu-id="df49f-180">[CreateClient](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.createclient)建立的執行個體`HttpClient`，會自動遵循重新導向，以及處理 cookie。</span><span class="sxs-lookup"><span data-stu-id="df49f-180">[CreateClient](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.createclient) creates an instance of `HttpClient` that automatically follows redirects and handles cookies.</span></span>

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/BasicTests.cs?name=snippet1)]

### <a name="test-a-secure-endpoint"></a><span data-ttu-id="df49f-181">測試的安全端點</span><span class="sxs-lookup"><span data-stu-id="df49f-181">Test a secure endpoint</span></span>

<span data-ttu-id="df49f-182">中的另一個測試`BasicTests`類別會檢查安全端點重新導向未驗證的使用者，應用程式的登入頁面。</span><span class="sxs-lookup"><span data-stu-id="df49f-182">Another test in the `BasicTests` class checks that a secure endpoint redirects an unauthenticated user to the app's Login page.</span></span>

<span data-ttu-id="df49f-183">在 SUT`/SecurePage`頁面使用[AuthorizePage](/dotnet/api/microsoft.extensions.dependencyinjection.pageconventioncollectionextensions.authorizepage)慣例，以套用[AuthorizeFilter](/dotnet/api/microsoft.aspnetcore.mvc.authorization.authorizefilter)至頁面。</span><span class="sxs-lookup"><span data-stu-id="df49f-183">In the SUT, the `/SecurePage` page uses an [AuthorizePage](/dotnet/api/microsoft.extensions.dependencyinjection.pageconventioncollectionextensions.authorizepage) convention to apply an [AuthorizeFilter](/dotnet/api/microsoft.aspnetcore.mvc.authorization.authorizefilter) to the page.</span></span> <span data-ttu-id="df49f-184">如需詳細資訊，請參閱[Razor 頁面授權慣例](xref:security/authorization/razor-pages-authorization#require-authorization-to-access-a-page)。</span><span class="sxs-lookup"><span data-stu-id="df49f-184">For more information, see [Razor Pages authorization conventions](xref:security/authorization/razor-pages-authorization#require-authorization-to-access-a-page).</span></span>

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/src/RazorPagesProject/Startup.cs?name=snippet1)]

<span data-ttu-id="df49f-185">在`Get_SecurePageRequiresAnAuthenticatedUser`測試， [WebApplicationFactoryClientOptions](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions)設為禁止重新導向設定[AllowAutoRedirect](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.allowautoredirect)至`false`:</span><span class="sxs-lookup"><span data-stu-id="df49f-185">In the `Get_SecurePageRequiresAnAuthenticatedUser` test, a [WebApplicationFactoryClientOptions](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions) is set to disallow redirects by setting [AllowAutoRedirect](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.allowautoredirect) to `false`:</span></span>

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/BasicTests.cs?name=snippet2)]

<span data-ttu-id="df49f-186">不允許用戶端，請遵循重新導向，您可以進行下列檢查：</span><span class="sxs-lookup"><span data-stu-id="df49f-186">By disallowing the client to follow the redirect, the following checks can be made:</span></span>

* <span data-ttu-id="df49f-187">SUT 所傳回的狀態碼可以檢查針對預期[HttpStatusCode.Redirect](/dotnet/api/system.net.httpstatuscode)結果，不最終狀態的程式碼之後重新導向至登入頁面，方式是[HttpStatusCode.OK](/dotnet/api/system.net.httpstatuscode).</span><span class="sxs-lookup"><span data-stu-id="df49f-187">The status code returned by the SUT can be checked against the expected [HttpStatusCode.Redirect](/dotnet/api/system.net.httpstatuscode) result, not the final status code after the redirect to the Login page, which would be [HttpStatusCode.OK](/dotnet/api/system.net.httpstatuscode).</span></span>
* <span data-ttu-id="df49f-188">`Location`回應標頭中的標頭值會檢查以確認它開頭`http://localhost/Identity/Account/Login`，不是最後登入頁面的回應，其中`Location`標頭不會出現。</span><span class="sxs-lookup"><span data-stu-id="df49f-188">The `Location` header value in the response headers is checked to confirm that it starts with `http://localhost/Identity/Account/Login`, not the final Login page response, where the `Location` header wouldn't be present.</span></span>

<span data-ttu-id="df49f-189">如需有關`WebApplicationFactoryClientOptions`，請參閱[用戶端選項](#client-options)> 一節。</span><span class="sxs-lookup"><span data-stu-id="df49f-189">For more information on `WebApplicationFactoryClientOptions`, see the [Client options](#client-options) section.</span></span>

## <a name="customize-webapplicationfactory"></a><span data-ttu-id="df49f-190">自訂 WebApplicationFactory</span><span class="sxs-lookup"><span data-stu-id="df49f-190">Customize WebApplicationFactory</span></span>

<span data-ttu-id="df49f-191">Web 主機組態可以建立獨立的測試類別繼承自`WebApplicationFactory`建立一或多個自訂處理站：</span><span class="sxs-lookup"><span data-stu-id="df49f-191">Web host configuration can be created independently of the test classes by inheriting from `WebApplicationFactory` to create one or more custom factories:</span></span>

1. <span data-ttu-id="df49f-192">繼承自`WebApplicationFactory`並覆寫[ConfigureWebHost](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.configurewebhost)。</span><span class="sxs-lookup"><span data-stu-id="df49f-192">Inherit from `WebApplicationFactory` and override [ConfigureWebHost](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.configurewebhost).</span></span> <span data-ttu-id="df49f-193">[IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder)允許的服務集合設定[ConfigureServices](/dotnet/api/microsoft.aspnetcore.hosting.istartup.configureservices):</span><span class="sxs-lookup"><span data-stu-id="df49f-193">The [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder) allows the configuration of the service collection with [ConfigureServices](/dotnet/api/microsoft.aspnetcore.hosting.istartup.configureservices):</span></span>

   [!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/CustomWebApplicationFactory.cs?name=snippet1)]

   <span data-ttu-id="df49f-194">資料庫在植入[範例應用程式](https://github.com/aspnet/Docs/tree/master/aspnetcore/test/integration-tests/samples)由執行`InitializeDbForTests`方法。</span><span class="sxs-lookup"><span data-stu-id="df49f-194">Database seeding in the [sample app](https://github.com/aspnet/Docs/tree/master/aspnetcore/test/integration-tests/samples) is performed by the `InitializeDbForTests` method.</span></span> <span data-ttu-id="df49f-195">中所述的方法是[整合測試的範例： 測試應用程式的組織](#test-app-organization)> 一節。</span><span class="sxs-lookup"><span data-stu-id="df49f-195">The method is described in the [Integration tests sample: Test app organization](#test-app-organization) section.</span></span>

2. <span data-ttu-id="df49f-196">使用自訂`CustomWebApplicationFactory`測試類別中。</span><span class="sxs-lookup"><span data-stu-id="df49f-196">Use the custom `CustomWebApplicationFactory` in test classes.</span></span> <span data-ttu-id="df49f-197">下列範例會使用中的處理站`IndexPageTests`類別：</span><span class="sxs-lookup"><span data-stu-id="df49f-197">The following example uses the factory in the `IndexPageTests` class:</span></span>

   [!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet1)]

   <span data-ttu-id="df49f-198">範例應用程式的用戶端設定以防止`HttpClient`從下列重新導向。</span><span class="sxs-lookup"><span data-stu-id="df49f-198">The sample app's client is configured to prevent the `HttpClient` from following redirects.</span></span> <span data-ttu-id="df49f-199">中所述[測試安全端點](#test-a-secure-endpoint) 區段中，這可讓測試以檢查應用程式的第一個回應的結果。</span><span class="sxs-lookup"><span data-stu-id="df49f-199">As explained in the [Test a secure endpoint](#test-a-secure-endpoint) section, this permits tests to check the result of the app's first response.</span></span> <span data-ttu-id="df49f-200">第一個回應是在許多測試，以重新導向`Location`標頭。</span><span class="sxs-lookup"><span data-stu-id="df49f-200">The first response is a redirect in many of these tests with a `Location` header.</span></span>

3. <span data-ttu-id="df49f-201">一般測試會使用`HttpClient`和 helper 方法來處理要求和回應：</span><span class="sxs-lookup"><span data-stu-id="df49f-201">A typical test uses the `HttpClient` and helper methods to process the request and the response:</span></span>

   [!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet2)]

<span data-ttu-id="df49f-202">任何 SUT 的 POST 要求都必須滿足的應用程式會自動成為 antiforgery 核取[資料保護 antiforgery 系統](xref:security/data-protection/introduction)。</span><span class="sxs-lookup"><span data-stu-id="df49f-202">Any POST request to the SUT must satisfy the antiforgery check that's automatically made by the app's [data protection antiforgery system](xref:security/data-protection/introduction).</span></span> <span data-ttu-id="df49f-203">若要排列測試的 POST 要求，必須測試應用程式：</span><span class="sxs-lookup"><span data-stu-id="df49f-203">In order to arrange for a test's POST request, the test app must:</span></span>

1. <span data-ttu-id="df49f-204">提出要求的頁面。</span><span class="sxs-lookup"><span data-stu-id="df49f-204">Make a request for the page.</span></span>
1. <span data-ttu-id="df49f-205">剖析 antiforgery cookie 和來自回應的要求驗證語彙基元。</span><span class="sxs-lookup"><span data-stu-id="df49f-205">Parse the antiforgery cookie and request validation token from the response.</span></span>
1. <span data-ttu-id="df49f-206">在位置進行 antiforgery cookie 和要求驗證的 POST 要求語彙基元。</span><span class="sxs-lookup"><span data-stu-id="df49f-206">Make the POST request with the antiforgery cookie and request validation token in place.</span></span>

<span data-ttu-id="df49f-207">`SendAsync` Helper 擴充方法 (*Helpers/HttpClientExtensions.cs*) 和`GetDocumentAsync`helper 方法 (*Helpers/HtmlHelpers.cs*) 中[的範例應用程式](https://github.com/aspnet/Docs/tree/master/aspnetcore/test/integration-tests/samples/)使用[AngleSharp](https://anglesharp.github.io/)剖析器來處理 antiforgery 核取，使用下列方法：</span><span class="sxs-lookup"><span data-stu-id="df49f-207">The `SendAsync` helper extension methods (*Helpers/HttpClientExtensions.cs*) and the `GetDocumentAsync` helper method (*Helpers/HtmlHelpers.cs*) in the [sample app](https://github.com/aspnet/Docs/tree/master/aspnetcore/test/integration-tests/samples/) use the [AngleSharp](https://anglesharp.github.io/) parser to handle the antiforgery check with the following methods:</span></span>

* <span data-ttu-id="df49f-208">`GetDocumentAsync` &ndash; 接收[HttpResponseMessage](/dotnet/api/system.net.http.httpresponsemessage)並傳回`IHtmlDocument`。</span><span class="sxs-lookup"><span data-stu-id="df49f-208">`GetDocumentAsync` &ndash; Receives the [HttpResponseMessage](/dotnet/api/system.net.http.httpresponsemessage) and returns an `IHtmlDocument`.</span></span> <span data-ttu-id="df49f-209">`GetDocumentAsync` 使用準備處理站*虛擬回應*根據原始`HttpResponseMessage`。</span><span class="sxs-lookup"><span data-stu-id="df49f-209">`GetDocumentAsync` uses a factory that prepares a *virtual response* based on the original `HttpResponseMessage`.</span></span> <span data-ttu-id="df49f-210">如需詳細資訊，請參閱[AngleSharp 文件](https://github.com/AngleSharp/AngleSharp#documentation)。</span><span class="sxs-lookup"><span data-stu-id="df49f-210">For more information, see the [AngleSharp documentation](https://github.com/AngleSharp/AngleSharp#documentation).</span></span>
* <span data-ttu-id="df49f-211">`SendAsync` 擴充方法`HttpClient`撰寫[HttpRequestMessage](/dotnet/api/system.net.http.httprequestmessage)呼叫[SendAsync(HttpRequestMessage)](/dotnet/api/system.net.http.httpclient.sendasync#System_Net_Http_HttpClient_SendAsync_System_Net_Http_HttpRequestMessage_)提交要求給 SUT。</span><span class="sxs-lookup"><span data-stu-id="df49f-211">`SendAsync` extension methods for the `HttpClient` compose an [HttpRequestMessage](/dotnet/api/system.net.http.httprequestmessage) and call [SendAsync(HttpRequestMessage)](/dotnet/api/system.net.http.httpclient.sendasync#System_Net_Http_HttpClient_SendAsync_System_Net_Http_HttpRequestMessage_) to submit requests to the SUT.</span></span> <span data-ttu-id="df49f-212">多載`SendAsync`接受 HTML 表單 (`IHtmlFormElement`) 和下列：</span><span class="sxs-lookup"><span data-stu-id="df49f-212">Overloads for `SendAsync` accept the HTML form (`IHtmlFormElement`) and the following:</span></span>
  - <span data-ttu-id="df49f-213">送出表單的按鈕 (`IHtmlElement`)</span><span class="sxs-lookup"><span data-stu-id="df49f-213">Submit button of the form (`IHtmlElement`)</span></span>
  - <span data-ttu-id="df49f-214">表單值集合 (`IEnumerable<KeyValuePair<string, string>>`)</span><span class="sxs-lookup"><span data-stu-id="df49f-214">Form values collection (`IEnumerable<KeyValuePair<string, string>>`)</span></span>
  - <span data-ttu-id="df49f-215">送出按鈕 (`IHtmlElement`) 和表單值 (`IEnumerable<KeyValuePair<string, string>>`)</span><span class="sxs-lookup"><span data-stu-id="df49f-215">Submit button (`IHtmlElement`) and form values (`IEnumerable<KeyValuePair<string, string>>`)</span></span>

> [!NOTE]
> <span data-ttu-id="df49f-216">[AngleSharp](https://anglesharp.github.io/)第三方剖析用於示範用途，本主題中的範例應用程式的程式庫。</span><span class="sxs-lookup"><span data-stu-id="df49f-216">[AngleSharp](https://anglesharp.github.io/) is a third-party parsing library used for demonstration purposes in this topic and the sample app.</span></span> <span data-ttu-id="df49f-217">AngleSharp 不支援或所需的整合測試的 ASP.NET Core 應用程式。</span><span class="sxs-lookup"><span data-stu-id="df49f-217">AngleSharp isn't supported or required for integration testing of ASP.NET Core apps.</span></span> <span data-ttu-id="df49f-218">其他剖析器可以使用，例如[Html 靈活度組件 (HAP)](http://html-agility-pack.net/)。</span><span class="sxs-lookup"><span data-stu-id="df49f-218">Other parsers can be used, such as the [Html Agility Pack (HAP)](http://html-agility-pack.net/).</span></span> <span data-ttu-id="df49f-219">另一個方法是撰寫程式碼來直接處理 antiforgery 系統的要求驗證語彙基元和 antiforgery cookie。</span><span class="sxs-lookup"><span data-stu-id="df49f-219">Another approach is to write code to handle the antiforgery system's request verification token and antiforgery cookie directly.</span></span>

## <a name="customize-the-client-with-withwebhostbuilder"></a><span data-ttu-id="df49f-220">自訂 WithWebHostBuilder 與用戶端</span><span class="sxs-lookup"><span data-stu-id="df49f-220">Customize the client with WithWebHostBuilder</span></span>

<span data-ttu-id="df49f-221">在測試方法中，需要額外的設定時[WithWebHostBuilder](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.withwebhostbuilder)新建`WebApplicationFactory`與[IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder) ，進一步自訂組態。</span><span class="sxs-lookup"><span data-stu-id="df49f-221">When additional configuration is required within a test method, [WithWebHostBuilder](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.withwebhostbuilder) creates a new `WebApplicationFactory` with an [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder) that is further customized by configuration.</span></span>

<span data-ttu-id="df49f-222">`Post_DeleteMessageHandler_ReturnsRedirectToRoot`測試方法的[範例應用程式](https://github.com/aspnet/Docs/tree/master/aspnetcore/test/integration-tests/samples)示範如何使用`WithWebHostBuilder`。</span><span class="sxs-lookup"><span data-stu-id="df49f-222">The `Post_DeleteMessageHandler_ReturnsRedirectToRoot` test method of the [sample app](https://github.com/aspnet/Docs/tree/master/aspnetcore/test/integration-tests/samples) demonstrates the use of `WithWebHostBuilder`.</span></span> <span data-ttu-id="df49f-223">此測試會在資料庫中執行記錄刪除，透過觸發 SUT 中的送出表單。</span><span class="sxs-lookup"><span data-stu-id="df49f-223">This test performs a record delete in the database by triggering a form submission in the SUT.</span></span>

<span data-ttu-id="df49f-224">因為另一個測試中`IndexPageTests`類別執行的作業，刪除所有資料庫中記錄和可能執行之前`Post_DeleteMessageHandler_ReturnsRedirectToRoot`方法時，資料庫在這個測試方法，以確保有一筆記錄的刪除 SUT 植入。</span><span class="sxs-lookup"><span data-stu-id="df49f-224">Because another test in the `IndexPageTests` class performs an operation that deletes all of the records in the database and may run before the `Post_DeleteMessageHandler_ReturnsRedirectToRoot` method, the database is seeded in this test method to ensure that a record is present for the SUT to delete.</span></span> <span data-ttu-id="df49f-225">選取`deleteBtn1`按鈕`messages`SUT 的要求中模擬 SUT 中的表單時：</span><span class="sxs-lookup"><span data-stu-id="df49f-225">Selecting the `deleteBtn1` button of the `messages` form in the SUT is simulated in the request to the SUT:</span></span>

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet3)]

## <a name="client-options"></a><span data-ttu-id="df49f-226">用戶端選項</span><span class="sxs-lookup"><span data-stu-id="df49f-226">Client options</span></span>

<span data-ttu-id="df49f-227">下表顯示預設[WebApplicationFactoryClientOptions](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions)可用時建立`HttpClient`執行個體。</span><span class="sxs-lookup"><span data-stu-id="df49f-227">The following table shows the default [WebApplicationFactoryClientOptions](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions) available when creating `HttpClient` instances.</span></span>

| <span data-ttu-id="df49f-228">選項</span><span class="sxs-lookup"><span data-stu-id="df49f-228">Option</span></span> | <span data-ttu-id="df49f-229">描述</span><span class="sxs-lookup"><span data-stu-id="df49f-229">Description</span></span> | <span data-ttu-id="df49f-230">預設</span><span class="sxs-lookup"><span data-stu-id="df49f-230">Default</span></span> |
| ------ | ----------- | ------- |
| [<span data-ttu-id="df49f-231">AllowAutoRedirect</span><span class="sxs-lookup"><span data-stu-id="df49f-231">AllowAutoRedirect</span></span>](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.allowautoredirect) | <span data-ttu-id="df49f-232">取得或設定是否`HttpClient`執行個體應該自動追蹤重新導向回應。</span><span class="sxs-lookup"><span data-stu-id="df49f-232">Gets or sets whether or not `HttpClient` instances should automatically follow redirect responses.</span></span> | `true` |
| [<span data-ttu-id="df49f-233">BaseAddress</span><span class="sxs-lookup"><span data-stu-id="df49f-233">BaseAddress</span></span>](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.baseaddress) | <span data-ttu-id="df49f-234">取得或設定基底地址`HttpClient`執行個體。</span><span class="sxs-lookup"><span data-stu-id="df49f-234">Gets or sets the base address of `HttpClient` instances.</span></span> | `http://localhost` |
| [<span data-ttu-id="df49f-235">HandleCookies</span><span class="sxs-lookup"><span data-stu-id="df49f-235">HandleCookies</span></span>](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.handlecookies) | <span data-ttu-id="df49f-236">取得或設定是否`HttpClient`執行個體應該處理 cookie。</span><span class="sxs-lookup"><span data-stu-id="df49f-236">Gets or sets whether `HttpClient` instances should handle cookies.</span></span> | `true` |
| [<span data-ttu-id="df49f-237">MaxAutomaticRedirections</span><span class="sxs-lookup"><span data-stu-id="df49f-237">MaxAutomaticRedirections</span></span>](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.maxautomaticredirections) | <span data-ttu-id="df49f-238">取得或設定最大重新導向回應數目`HttpClient`應遵循的執行個體。</span><span class="sxs-lookup"><span data-stu-id="df49f-238">Gets or sets the maximum number of redirect responses that `HttpClient` instances should follow.</span></span> | <span data-ttu-id="df49f-239">7</span><span class="sxs-lookup"><span data-stu-id="df49f-239">7</span></span> |

<span data-ttu-id="df49f-240">建立`WebApplicationFactoryClientOptions`類別，並將它傳遞給[CreateClient](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.createclient)方法 （預設值以在程式碼範例所示）：</span><span class="sxs-lookup"><span data-stu-id="df49f-240">Create the `WebApplicationFactoryClientOptions` class and pass it to the [CreateClient](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.createclient) method (default values are shown in the code example):</span></span>

```csharp
// Default client option values are shown
var clientOptions = new WebApplicationFactoryClientOptions();
clientOptions.AllowAutoRedirect = true;
clientOptions.BaseAddress = new Uri("http://localhost");
clientOptions.HandleCookies = true;
clientOptions.MaxAutomaticRedirections = 7;

_client = _factory.CreateClient(clientOptions);
```

## <a name="how-the-test-infrastructure-infers-the-app-content-root-path"></a><span data-ttu-id="df49f-241">如何測試基礎結構會推斷的應用程式內容的根路徑</span><span class="sxs-lookup"><span data-stu-id="df49f-241">How the test infrastructure infers the app content root path</span></span>

<span data-ttu-id="df49f-242">`WebApplicationFactory`建構函式會藉由搜尋推斷的應用程式內容的根路徑[WebApplicationFactoryContentRootAttribute](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactorycontentrootattribute)包含具有索引鍵等於整合測試的組件上`TEntryPoint`的組件`System.Reflection.Assembly.FullName`.</span><span class="sxs-lookup"><span data-stu-id="df49f-242">The `WebApplicationFactory` constructor infers the app content root path by searching for a [WebApplicationFactoryContentRootAttribute](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactorycontentrootattribute) on the assembly containing the integration tests with a key equal to the `TEntryPoint` assembly `System.Reflection.Assembly.FullName`.</span></span> <span data-ttu-id="df49f-243">萬一找不到具有正確的索引鍵的屬性，`WebApplicationFactory`會回復為搜尋的方案檔 (*\*.sln*)，並將附加`TEntryPoint`的方案目錄的組件名稱。</span><span class="sxs-lookup"><span data-stu-id="df49f-243">In case an attribute with the correct key isn't found, `WebApplicationFactory` falls back to searching for a solution file (*\*.sln*) and appends the `TEntryPoint` assembly name to the solution directory.</span></span> <span data-ttu-id="df49f-244">應用程式根目錄 （內容的根路徑） 用來探索檢視和內容檔案。</span><span class="sxs-lookup"><span data-stu-id="df49f-244">The app root directory (the content root path) is used to discover views and content files.</span></span>

<span data-ttu-id="df49f-245">在大部分情況下，它就不需要明確地將應用程式內容的根目錄中，設定如下的搜尋邏輯通常在執行階段尋找正確內容的根。</span><span class="sxs-lookup"><span data-stu-id="df49f-245">In most cases, it isn't necessary to explicitly set the app content root, as the search logic usually finds the correct content root at runtime.</span></span> <span data-ttu-id="df49f-246">在找到內容的根目錄不是特殊案例中使用內建的搜尋演算法，內容可以在明確或使用自訂邏輯，指定根應用程式。</span><span class="sxs-lookup"><span data-stu-id="df49f-246">In special scenarios where the content root isn't found using the built-in search algorithm, the app content root can be specified explicitly or by using custom logic.</span></span> <span data-ttu-id="df49f-247">若要設定應用程式內容的根目錄中的情況下，呼叫`UseSolutionRelativeContentRoot`擴充方法，從[Microsoft.AspNetCore.TestHost](https://www.nuget.org/packages/Microsoft.AspNetCore.TestHost)封裝。</span><span class="sxs-lookup"><span data-stu-id="df49f-247">To set the app content root in those scenarios, call the `UseSolutionRelativeContentRoot` extension method from the [Microsoft.AspNetCore.TestHost](https://www.nuget.org/packages/Microsoft.AspNetCore.TestHost) package.</span></span> <span data-ttu-id="df49f-248">提供方案的相對路徑和選用的方案檔案名稱或 glob 模式 (預設 = `*.sln`)。</span><span class="sxs-lookup"><span data-stu-id="df49f-248">Supply the solution's relative path and optional solution file name or glob pattern (default = `*.sln`).</span></span>

<span data-ttu-id="df49f-249">呼叫[UseSolutionRelativeContentRoot](/dotnet/api/microsoft.aspnetcore.testhost.webhostbuilderextensions.usesolutionrelativecontentroot)擴充方法使用*一個*的方法包括：</span><span class="sxs-lookup"><span data-stu-id="df49f-249">Call the [UseSolutionRelativeContentRoot](/dotnet/api/microsoft.aspnetcore.testhost.webhostbuilderextensions.usesolutionrelativecontentroot) extension method using *ONE* of the following approaches:</span></span>

* <span data-ttu-id="df49f-250">設定與測試類別時`WebApplicationFactory`，提供的自訂組態[IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder):</span><span class="sxs-lookup"><span data-stu-id="df49f-250">When configuring test classes with `WebApplicationFactory`, provide a custom configuration with the [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder):</span></span>

   ```csharp
   public IndexPageTests(
       WebApplicationFactory<RazorPagesProject.Startup> factory)
   {
       var _factory = factory.WithWebHostBuilder(builder =>
       {
           builder.UseSolutionRelativeContentRoot("<SOLUTION-RELATIVE-PATH>");

           ...
       });
   }
   ```

* <span data-ttu-id="df49f-251">當利用自訂設定的測試類別時`WebApplicationFactory`，繼承自`WebApplicationFactory`並覆寫[ConfigureWebHost](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.configurewebhost):</span><span class="sxs-lookup"><span data-stu-id="df49f-251">When configuring test classes with a custom `WebApplicationFactory`, inherit from `WebApplicationFactory` and override [ConfigureWebHost](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.configurewebhost):</span></span>

   ```csharp
   public class CustomWebApplicationFactory<TStartup>
       : WebApplicationFactory<RazorPagesProject.Startup>
   {
       protected override void ConfigureWebHost(IWebHostBuilder builder)
       {
           builder.ConfigureServices(services =>
           {
               builder.UseSolutionRelativeContentRoot("<SOLUTION-RELATIVE-PATH>");

               ...
           });
       }
   }
   ```

## <a name="disable-shadow-copying"></a><span data-ttu-id="df49f-252">停用陰影複製</span><span class="sxs-lookup"><span data-stu-id="df49f-252">Disable shadow copying</span></span>

<span data-ttu-id="df49f-253">陰影複製會導致在不同的輸出資料夾的資料夾中執行測試。</span><span class="sxs-lookup"><span data-stu-id="df49f-253">Shadow copying causes the tests to execute in a different folder than the output folder.</span></span> <span data-ttu-id="df49f-254">測試中才能正常運作，陰影複製必須停用。</span><span class="sxs-lookup"><span data-stu-id="df49f-254">For tests to work properly, shadow copying must be disabled.</span></span> <span data-ttu-id="df49f-255">[範例應用程式](https://github.com/aspnet/Docs/tree/master/aspnetcore/test/integration-tests/samples)使用 xUnit，並停用所包括的 xUnit 陰影複製*xunit.runner.json*具有正確的組態設定檔。</span><span class="sxs-lookup"><span data-stu-id="df49f-255">The [sample app](https://github.com/aspnet/Docs/tree/master/aspnetcore/test/integration-tests/samples) uses xUnit and disables shadow copying for xUnit by including an *xunit.runner.json* file with the correct configuration setting.</span></span> <span data-ttu-id="df49f-256">如需詳細資訊，請參閱[使用 JSON 設定 xUnit.net](https://xunit.github.io/docs/configuring-with-json.html)。</span><span class="sxs-lookup"><span data-stu-id="df49f-256">For more information, see [Configuring xUnit.net with JSON](https://xunit.github.io/docs/configuring-with-json.html).</span></span>

<span data-ttu-id="df49f-257">新增*xunit.runner.json*具有下列內容在測試專案的根檔案：</span><span class="sxs-lookup"><span data-stu-id="df49f-257">Add the *xunit.runner.json* file to root of the test project with the following content:</span></span>

```json
{
  "shadowCopy": false
}
```

## <a name="integration-tests-sample"></a><span data-ttu-id="df49f-258">整合測試範例</span><span class="sxs-lookup"><span data-stu-id="df49f-258">Integration tests sample</span></span>

<span data-ttu-id="df49f-259">[範例應用程式](https://github.com/aspnet/Docs/tree/master/aspnetcore/test/integration-tests/samples)是兩個應用程式所組成：</span><span class="sxs-lookup"><span data-stu-id="df49f-259">The [sample app](https://github.com/aspnet/Docs/tree/master/aspnetcore/test/integration-tests/samples) is composed of two apps:</span></span>

| <span data-ttu-id="df49f-260">應用程式</span><span class="sxs-lookup"><span data-stu-id="df49f-260">App</span></span> | <span data-ttu-id="df49f-261">專案資料夾</span><span class="sxs-lookup"><span data-stu-id="df49f-261">Project folder</span></span> | <span data-ttu-id="df49f-262">描述</span><span class="sxs-lookup"><span data-stu-id="df49f-262">Description</span></span> |
| --- | -------------- | ----------- |
| <span data-ttu-id="df49f-263">訊息應用程式 (SUT)</span><span class="sxs-lookup"><span data-stu-id="df49f-263">Message app (the SUT)</span></span> | <span data-ttu-id="df49f-264">*src/RazorPagesProject*</span><span class="sxs-lookup"><span data-stu-id="df49f-264">*src/RazorPagesProject*</span></span> | <span data-ttu-id="df49f-265">允許使用者加入、 刪除其中一個，刪除所有項目，以及分析訊息。</span><span class="sxs-lookup"><span data-stu-id="df49f-265">Allows a user to add, delete one, delete all, and analyze messages.</span></span> |
| <span data-ttu-id="df49f-266">測試應用程式</span><span class="sxs-lookup"><span data-stu-id="df49f-266">Test app</span></span> | <span data-ttu-id="df49f-267">*tests/RazorPagesProject.Tests*</span><span class="sxs-lookup"><span data-stu-id="df49f-267">*tests/RazorPagesProject.Tests*</span></span> | <span data-ttu-id="df49f-268">用來整合測試 SUT。</span><span class="sxs-lookup"><span data-stu-id="df49f-268">Used to integration test the SUT.</span></span> |

<span data-ttu-id="df49f-269">可以執行測試，例如使用內建測試功能的 IDE， [Visual Studio](https://www.visualstudio.com/vs/)。</span><span class="sxs-lookup"><span data-stu-id="df49f-269">The tests can be run using the built-in test features of an IDE, such as [Visual Studio](https://www.visualstudio.com/vs/).</span></span> <span data-ttu-id="df49f-270">如果使用[Visual Studio Code](https://code.visualstudio.com/)或命令列中，執行下列命令，在命令提示字元中*tests/RazorPagesProject.Tests*資料夾：</span><span class="sxs-lookup"><span data-stu-id="df49f-270">If using [Visual Studio Code](https://code.visualstudio.com/) or the command line, execute the following command at a command prompt in the *tests/RazorPagesProject.Tests* folder:</span></span>

```console
dotnet test
```

### <a name="message-app-sut-organization"></a><span data-ttu-id="df49f-271">訊息應用程式 (SUT) 組織</span><span class="sxs-lookup"><span data-stu-id="df49f-271">Message app (SUT) organization</span></span>

<span data-ttu-id="df49f-272">SUT 是 Razor 頁面訊息系統具有下列特性：</span><span class="sxs-lookup"><span data-stu-id="df49f-272">The SUT is a Razor Pages message system with the following characteristics:</span></span>

* <span data-ttu-id="df49f-273">索引頁的應用程式 (*Pages/Index.cshtml*和*Pages/Index.cshtml.cs*) 提供 UI 和頁面來控制的新增、 刪除和訊息 （每個訊息的平均字） 的分析模型方法.</span><span class="sxs-lookup"><span data-stu-id="df49f-273">The Index page of the app (*Pages/Index.cshtml* and *Pages/Index.cshtml.cs*) provides a UI and page model methods to control the addition, deletion, and analysis of messages (average words per message).</span></span>
* <span data-ttu-id="df49f-274">所描述的訊息`Message`類別 (*Data/Message.cs*) 使用兩個屬性： `Id` （金鑰） 和`Text`（訊息）。</span><span class="sxs-lookup"><span data-stu-id="df49f-274">A message is described by the `Message` class (*Data/Message.cs*) with two properties: `Id` (key) and `Text` (message).</span></span> <span data-ttu-id="df49f-275">`Text`屬性所需，且限制為 200 個字元。</span><span class="sxs-lookup"><span data-stu-id="df49f-275">The `Text` property is required and limited to 200 characters.</span></span>
* <span data-ttu-id="df49f-276">訊息會儲存使用[Entity Framework 的記憶體中資料庫](/ef/core/providers/in-memory/)&#8224;。</span><span class="sxs-lookup"><span data-stu-id="df49f-276">Messages are stored using [Entity Framework's in-memory database](/ef/core/providers/in-memory/)&#8224;.</span></span>
* <span data-ttu-id="df49f-277">應用程式在其資料庫內容類別，包含資料存取層 (DAL) `AppDbContext` (*Data/AppDbContext.cs*)。</span><span class="sxs-lookup"><span data-stu-id="df49f-277">The app contains a data access layer (DAL) in its database context class, `AppDbContext` (*Data/AppDbContext.cs*).</span></span>
* <span data-ttu-id="df49f-278">如果資料庫是空的應用程式啟動時，訊息存放區會使用三個訊息中初始化。</span><span class="sxs-lookup"><span data-stu-id="df49f-278">If the database is empty on app startup, the message store is initialized with three messages.</span></span>
* <span data-ttu-id="df49f-279">應用程式包含`/SecurePage`，只能存取已驗證使用者。</span><span class="sxs-lookup"><span data-stu-id="df49f-279">The app includes a `/SecurePage` that can only be accessed by an authenticated user.</span></span>

<span data-ttu-id="df49f-280">&#8224;EF 主題[測試與 InMemory](/ef/core/miscellaneous/testing/in-memory)，說明如何使用記憶體中資料庫 MSTest 的測試。</span><span class="sxs-lookup"><span data-stu-id="df49f-280">&#8224;The EF topic, [Test with InMemory](/ef/core/miscellaneous/testing/in-memory), explains how to use an in-memory database for tests with MSTest.</span></span> <span data-ttu-id="df49f-281">本主題使用[xUnit](https://xunit.github.io/)測試架構。</span><span class="sxs-lookup"><span data-stu-id="df49f-281">This topic uses the [xUnit](https://xunit.github.io/) test framework.</span></span> <span data-ttu-id="df49f-282">測試概念與測試實作跨不同測試架構很類似的但不是完全相同。</span><span class="sxs-lookup"><span data-stu-id="df49f-282">Test concepts and test implementations across different test frameworks are similar but not identical.</span></span>

<span data-ttu-id="df49f-283">雖然此應用程式不會使用[儲存機制模式](http://martinfowler.com/eaaCatalog/repository.html)並不是有效的範例[工作單位 (UoW) 模式](https://martinfowler.com/eaaCatalog/unitOfWork.html)，Razor 頁面支援這些模式開發。</span><span class="sxs-lookup"><span data-stu-id="df49f-283">Although the app doesn't use the [repository pattern](http://martinfowler.com/eaaCatalog/repository.html) and isn't an effective example of the [Unit of Work (UoW) pattern](https://martinfowler.com/eaaCatalog/unitOfWork.html), Razor Pages supports these patterns of development.</span></span> <span data-ttu-id="df49f-284">如需詳細資訊，請參閱[設計基礎結構的持續性層級](/dotnet/standard/microservices-architecture/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)， [ASP.NET MVC 應用程式中實作的儲存機制和工作單元模式](/aspnet/mvc/overview/older-versions/getting-started-with-ef-5-using-mvc-4/implementing-the-repository-and-unit-of-work-patterns-in-an-asp-net-mvc-application)，和[測試控制器邏輯](/aspnet/core/mvc/controllers/testing)（此範例會實作儲存機制模式）。</span><span class="sxs-lookup"><span data-stu-id="df49f-284">For more information, see [Designing the infrastructure persistence layer](/dotnet/standard/microservices-architecture/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design), [Implementing the Repository and Unit of Work Patterns in an ASP.NET MVC Application](/aspnet/mvc/overview/older-versions/getting-started-with-ef-5-using-mvc-4/implementing-the-repository-and-unit-of-work-patterns-in-an-asp-net-mvc-application), and [Test controller logic](/aspnet/core/mvc/controllers/testing) (the sample implements the repository pattern).</span></span>

### <a name="test-app-organization"></a><span data-ttu-id="df49f-285">測試應用程式的組織</span><span class="sxs-lookup"><span data-stu-id="df49f-285">Test app organization</span></span>

<span data-ttu-id="df49f-286">測試應用程式是主控台應用程式內*tests/RazorPagesProject.Tests*資料夾。</span><span class="sxs-lookup"><span data-stu-id="df49f-286">The test app is a console app inside the *tests/RazorPagesProject.Tests* folder.</span></span>

| <span data-ttu-id="df49f-287">測試應用程式的資料夾</span><span class="sxs-lookup"><span data-stu-id="df49f-287">Test app folder</span></span> | <span data-ttu-id="df49f-288">描述</span><span class="sxs-lookup"><span data-stu-id="df49f-288">Description</span></span> |
| --------------- | ----------- |
| <span data-ttu-id="df49f-289">*BasicTests*</span><span class="sxs-lookup"><span data-stu-id="df49f-289">*BasicTests*</span></span> | <span data-ttu-id="df49f-290">*BasicTests.cs*包含路由、 存取安全網頁由未經驗證的使用者，並取得 GitHub 使用者設定檔和檢查設定檔的使用者登入的測試方法。</span><span class="sxs-lookup"><span data-stu-id="df49f-290">*BasicTests.cs* contains test methods for routing, accessing a secure page by an unauthenticated user, and obtaining a GitHub user profile and checking the profile's user login.</span></span> |
| <span data-ttu-id="df49f-291">*IntegrationTests*</span><span class="sxs-lookup"><span data-stu-id="df49f-291">*IntegrationTests*</span></span> | <span data-ttu-id="df49f-292">*IndexPageTests.cs*包含使用自訂的索引頁的整合測試`WebApplicationFactory`類別。</span><span class="sxs-lookup"><span data-stu-id="df49f-292">*IndexPageTests.cs* contains the integration tests for the Index page using custom `WebApplicationFactory` class.</span></span> |
| <span data-ttu-id="df49f-293">*Helper/公用程式*</span><span class="sxs-lookup"><span data-stu-id="df49f-293">*Helpers/Utilities*</span></span> | <ul><li><span data-ttu-id="df49f-294">*Utilities.cs*包含`InitializeDbForTests`方法用來植入的測試資料的資料庫。</span><span class="sxs-lookup"><span data-stu-id="df49f-294">*Utilities.cs* contains the `InitializeDbForTests` method used to seed the database with test data.</span></span></li><li><span data-ttu-id="df49f-295">*HtmlHelpers.cs*提供方法來傳回 AngleSharp`IHtmlDocument`以供測試方法。</span><span class="sxs-lookup"><span data-stu-id="df49f-295">*HtmlHelpers.cs* provides a method to return an AngleSharp `IHtmlDocument` for use by the test methods.</span></span></li><li><span data-ttu-id="df49f-296">*HttpClientExtensions.cs*提供的多載`SendAsync`提交要求給 SUT。</span><span class="sxs-lookup"><span data-stu-id="df49f-296">*HttpClientExtensions.cs* provide overloads for `SendAsync` to submit requests to the SUT.</span></span></li></ul> |

<span data-ttu-id="df49f-297">測試架構是[xUnit](https://xunit.github.io/)。</span><span class="sxs-lookup"><span data-stu-id="df49f-297">The test framework is [xUnit](https://xunit.github.io/).</span></span> <span data-ttu-id="df49f-298">搜尋整合測試都會使用[Microsoft.AspNetCore.TestHost](/dotnet/api/microsoft.aspnetcore.testhost)，其中包括[TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver)。</span><span class="sxs-lookup"><span data-stu-id="df49f-298">Integration tests are conducted using the [Microsoft.AspNetCore.TestHost](/dotnet/api/microsoft.aspnetcore.testhost), which includes the [TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver).</span></span> <span data-ttu-id="df49f-299">因為[Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing)封裝用來設定測試主應用程式和測試伺服器`TestHost`和`TestServer`套件不需要直接將封裝中測試應用程式的專案檔的參考或開發人員在測試應用程式的組態。</span><span class="sxs-lookup"><span data-stu-id="df49f-299">Because the [Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing) package is used to configure the test host and test server, the `TestHost` and `TestServer` packages don't require direct package references in the test app's project file or developer configuration in the test app.</span></span>

<span data-ttu-id="df49f-300">**植入資料庫中的測試**</span><span class="sxs-lookup"><span data-stu-id="df49f-300">**Seeding the database for testing**</span></span>

<span data-ttu-id="df49f-301">整合測試通常需要在測試執行前的資料庫中的小型資料集。</span><span class="sxs-lookup"><span data-stu-id="df49f-301">Integration tests usually require a small dataset in the database prior to the test execution.</span></span> <span data-ttu-id="df49f-302">比方說，刪除測試會呼叫為資料庫記錄的刪除，所以資料庫必須至少一個記錄刪除要求，才能成功。</span><span class="sxs-lookup"><span data-stu-id="df49f-302">For example, a delete test calls for a database record deletion, so the database must have at least one record for the delete request to succeed.</span></span>

<span data-ttu-id="df49f-303">範例應用程式植入的資料庫中的三個訊息*Utilities.cs*它們執行時，可以使用測試：</span><span class="sxs-lookup"><span data-stu-id="df49f-303">The sample app seeds the database with three messages in *Utilities.cs* that tests can use when they execute:</span></span>

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/Helpers/Utilities.cs?name=snippet1)]

## <a name="additional-resources"></a><span data-ttu-id="df49f-304">其他資源</span><span class="sxs-lookup"><span data-stu-id="df49f-304">Additional resources</span></span>

* [<span data-ttu-id="df49f-305">單元測試</span><span class="sxs-lookup"><span data-stu-id="df49f-305">Unit tests</span></span>](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)
* [<span data-ttu-id="df49f-306">Razor 頁面單元測試</span><span class="sxs-lookup"><span data-stu-id="df49f-306">Razor Pages unit tests</span></span>](xref:test/razor-pages-tests)
* [<span data-ttu-id="df49f-307">中介軟體</span><span class="sxs-lookup"><span data-stu-id="df49f-307">Middleware</span></span>](xref:fundamentals/middleware/index)
* [<span data-ttu-id="df49f-308">測試控制器</span><span class="sxs-lookup"><span data-stu-id="df49f-308">Test controllers</span></span>](xref:mvc/controllers/testing)