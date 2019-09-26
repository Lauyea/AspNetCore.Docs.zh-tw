---
title: 保護 ASP.NET Core Blazor 伺服器應用程式
author: guardrex
description: 瞭解如何減緩 Blazor 伺服器應用程式的安全性威脅。
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 09/23/2019
uid: security/blazor/server
ms.openlocfilehash: 706f504738d9c6e5af3c368c382424f2e206bcbf
ms.sourcegitcommit: 79eeb17604b536e8f34641d1e6b697fb9a2ee21f
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2019
ms.locfileid: "71211718"
---
# <a name="secure-aspnet-core-blazor-server-apps"></a><span data-ttu-id="a595a-103">保護 ASP.NET Core Blazor 伺服器應用程式</span><span class="sxs-lookup"><span data-stu-id="a595a-103">Secure ASP.NET Core Blazor Server apps</span></span>

<span data-ttu-id="a595a-104">By [Javier Calvarro Nelson](https://github.com/javiercn)</span><span class="sxs-lookup"><span data-stu-id="a595a-104">By [Javier Calvarro Nelson](https://github.com/javiercn)</span></span>

<span data-ttu-id="a595a-105">Blazor 伺服器應用程式會採用具*狀態*的資料處理模型，其中伺服器和用戶端會維護長期的關聯性。</span><span class="sxs-lookup"><span data-stu-id="a595a-105">Blazor Server apps adopt a *stateful* data processing model, where the server and client maintain a long-lived relationship.</span></span> <span data-ttu-id="a595a-106">持續性狀態是由[電路](xref:blazor/state-management)維護，其可跨越也可能很長的連接。</span><span class="sxs-lookup"><span data-stu-id="a595a-106">The persistent state is maintained by a [circuit](xref:blazor/state-management), which can span connections that are also potentially long-lived.</span></span>

<span data-ttu-id="a595a-107">當使用者造訪 Blazor 伺服器網站時，伺服器會在伺服器的記憶體中建立線路。</span><span class="sxs-lookup"><span data-stu-id="a595a-107">When a user visits a Blazor Server site, the server creates a circuit in the server's memory.</span></span> <span data-ttu-id="a595a-108">線路會向瀏覽器指出要呈現哪些內容並回應事件，例如當使用者選取 UI 中的按鈕時。</span><span class="sxs-lookup"><span data-stu-id="a595a-108">The circuit indicates to the browser what content to render and responds to events, such as when the user selects a button in the UI.</span></span> <span data-ttu-id="a595a-109">若要執行這些動作，線路會在使用者的瀏覽器中叫用 JavaScript 函數，並在伺服器上叫用 .NET 方法。</span><span class="sxs-lookup"><span data-stu-id="a595a-109">To perform these actions, a circuit invokes JavaScript functions in the user's browser and .NET methods on the server.</span></span> <span data-ttu-id="a595a-110">這個雙向以 JavaScript 為基礎的互動稱為[javascript interop （JS interop）](xref:blazor/javascript-interop)。</span><span class="sxs-lookup"><span data-stu-id="a595a-110">This two-way JavaScript-based interaction is referred to as [JavaScript interop (JS interop)](xref:blazor/javascript-interop).</span></span>

<span data-ttu-id="a595a-111">由於 JS interop 是透過網際網路執行，而用戶端使用遠端瀏覽器，因此 Blazor 伺服器應用程式共用大部分的 web 應用程式安全性考慮。</span><span class="sxs-lookup"><span data-stu-id="a595a-111">Because JS interop occurs over the Internet and the client uses a remote browser, Blazor Server apps share most web app security concerns.</span></span> <span data-ttu-id="a595a-112">本主題說明 Blazor 伺服器應用程式的常見威脅，並提供著重于網際網路面向應用程式的威脅防護指導方針。</span><span class="sxs-lookup"><span data-stu-id="a595a-112">This topic describes common threats to Blazor Server apps and provides threat mitigation guidance focused on Internet-facing apps.</span></span>

<span data-ttu-id="a595a-113">在受限制的環境（例如公司網路或內部網路內部）中，以下是一些緩和措施指引：</span><span class="sxs-lookup"><span data-stu-id="a595a-113">In constrained environments, such as inside corporate networks or intranets, some of the mitigation guidance either:</span></span>

* <span data-ttu-id="a595a-114">不適用於受限的環境。</span><span class="sxs-lookup"><span data-stu-id="a595a-114">Doesn't apply in the constrained environment.</span></span>
* <span data-ttu-id="a595a-115">不值得實行成本，因為在受限的環境中，安全性風險很低。</span><span class="sxs-lookup"><span data-stu-id="a595a-115">Isn't worth the cost to implement because the security risk is low in a constrained environment.</span></span>

## <a name="resource-exhaustion"></a><span data-ttu-id="a595a-116">資源耗盡</span><span class="sxs-lookup"><span data-stu-id="a595a-116">Resource exhaustion</span></span>

<span data-ttu-id="a595a-117">當用戶端與伺服器互動，並導致伺服器耗用過多的資源時，就會發生資源耗盡的情況。</span><span class="sxs-lookup"><span data-stu-id="a595a-117">Resource exhaustion can occur when a client interacts with the server and causes the server to consume excessive resources.</span></span> <span data-ttu-id="a595a-118">過多的資源耗用量主要會影響：</span><span class="sxs-lookup"><span data-stu-id="a595a-118">Excessive resource consumption primarily affects:</span></span>

* [<span data-ttu-id="a595a-119">CPU</span><span class="sxs-lookup"><span data-stu-id="a595a-119">CPU</span></span>](#cpu)
* [<span data-ttu-id="a595a-120">記憶體</span><span class="sxs-lookup"><span data-stu-id="a595a-120">Memory</span></span>](#memory)
* [<span data-ttu-id="a595a-121">用戶端連接</span><span class="sxs-lookup"><span data-stu-id="a595a-121">Client connections</span></span>](#client-connections)

<span data-ttu-id="a595a-122">阻絕服務（DoS）攻擊通常會設法耗盡應用程式或伺服器的資源。</span><span class="sxs-lookup"><span data-stu-id="a595a-122">Denial of service (DoS) attacks usually seek to exhaust an app or server's resources.</span></span> <span data-ttu-id="a595a-123">不過，資源耗盡不一定是系統遭受攻擊的結果。</span><span class="sxs-lookup"><span data-stu-id="a595a-123">However, resource exhaustion isn't necessarily the result of an attack on the system.</span></span> <span data-ttu-id="a595a-124">例如，有限的資源可能會因為高使用者需求而耗盡。</span><span class="sxs-lookup"><span data-stu-id="a595a-124">For example, finite resources can be exhausted due to high user demand.</span></span> <span data-ttu-id="a595a-125">[拒絕服務（dos）攻擊](#denial-of-service-dos-attacks)一節會進一步涵蓋 DoS。</span><span class="sxs-lookup"><span data-stu-id="a595a-125">DoS is covered further in the [Denial of service (DoS) attacks](#denial-of-service-dos-attacks) section.</span></span>

<span data-ttu-id="a595a-126">Blazor framework 外部的資源（例如資料庫和檔案控制代碼，用來讀取和寫入檔案）可能也會遇到資源耗盡的情況。</span><span class="sxs-lookup"><span data-stu-id="a595a-126">Resources external to the Blazor framework, such as databases and file handles (used to read and write files), may also experience resource exhaustion.</span></span> <span data-ttu-id="a595a-127">如需詳細資訊，請參閱<xref:performance/performance-best-practices>。</span><span class="sxs-lookup"><span data-stu-id="a595a-127">For more information, see <xref:performance/performance-best-practices>.</span></span>

### <a name="cpu"></a><span data-ttu-id="a595a-128">CPU</span><span class="sxs-lookup"><span data-stu-id="a595a-128">CPU</span></span>

<span data-ttu-id="a595a-129">當一或多個用戶端強制服務器執行密集的 CPU 工作時，可能會發生 CPU 耗盡。</span><span class="sxs-lookup"><span data-stu-id="a595a-129">CPU exhaustion can occur when one or more clients force the server to perform intensive CPU work.</span></span>

<span data-ttu-id="a595a-130">例如，假設有一個 Blazor 伺服器應用程式會計算*Fibonnacci 編號*。</span><span class="sxs-lookup"><span data-stu-id="a595a-130">For example, consider a Blazor Server app that calculates a *Fibonnacci number*.</span></span> <span data-ttu-id="a595a-131">Fibonnacci 號碼是從 Fibonnacci 序列產生的，其中序列中的每個數位都是上述兩個數字的總和。</span><span class="sxs-lookup"><span data-stu-id="a595a-131">A Fibonnacci number is produced from a Fibonnacci sequence, where each number in the sequence is the sum of the two preceding numbers.</span></span> <span data-ttu-id="a595a-132">到達答案所需的工作量取決於順序的長度和初始值的大小。</span><span class="sxs-lookup"><span data-stu-id="a595a-132">The amount of work required to reach the answer depends on the length of the sequence and the size of the initial value.</span></span> <span data-ttu-id="a595a-133">如果應用程式不會對用戶端的要求加上限制，則需要大量 CPU 的計算可能會佔據 CPU 的時間，並降低其他工作的效能。</span><span class="sxs-lookup"><span data-stu-id="a595a-133">If the app doesn't place limits on a client's request, the CPU-intensive calculations may dominate the CPU's time and diminish the performance of other tasks.</span></span> <span data-ttu-id="a595a-134">過多的資源耗用量是影響可用性的安全性考慮。</span><span class="sxs-lookup"><span data-stu-id="a595a-134">Excessive resource consumption is a security concern impacting availability.</span></span>

<span data-ttu-id="a595a-135">CPU 耗盡是所有公開應用程式的考慮。</span><span class="sxs-lookup"><span data-stu-id="a595a-135">CPU exhaustion is a concern for all public-facing apps.</span></span> <span data-ttu-id="a595a-136">在一般 web 應用程式中，要求和連線會以保護的形式提供，但 Blazor 伺服器應用程式不會提供相同的保護措施。</span><span class="sxs-lookup"><span data-stu-id="a595a-136">In regular web apps, requests and connections time out as a safeguard, but Blazor Server apps don't provide the same safeguards.</span></span> <span data-ttu-id="a595a-137">Blazor 伺服器應用程式必須先包含適當的檢查和限制，才能執行可能耗用大量 CPU 的工作。</span><span class="sxs-lookup"><span data-stu-id="a595a-137">Blazor Server apps must include appropriate checks and limits before performing potentially CPU-intensive work.</span></span>

### <a name="memory"></a><span data-ttu-id="a595a-138">記憶體</span><span class="sxs-lookup"><span data-stu-id="a595a-138">Memory</span></span>

<span data-ttu-id="a595a-139">當一或多個用戶端強制服務器耗用大量的記憶體時，可能會發生記憶體耗盡。</span><span class="sxs-lookup"><span data-stu-id="a595a-139">Memory exhaustion can occur when one or more clients force the server to consume a large amount of memory.</span></span>

<span data-ttu-id="a595a-140">例如，假設有一個 Blazor 伺服器端應用程式，其元件會接受並顯示專案清單。</span><span class="sxs-lookup"><span data-stu-id="a595a-140">For example, consider a Blazor-server side app with a component that accepts and displays a list of items.</span></span> <span data-ttu-id="a595a-141">如果 Blazor 應用程式不會限制允許的專案數，或轉譯回用戶端的專案數，則記憶體密集型處理和轉譯可能會讓伺服器的記憶體在伺服器的效能受到影響的時間點。</span><span class="sxs-lookup"><span data-stu-id="a595a-141">If the Blazor app doesn't place limits on the number of items allowed or the number of items rendered back to the client, the memory-intensive processing and rendering may dominate the server's memory to the point where performance of the server suffers.</span></span> <span data-ttu-id="a595a-142">伺服器可能當機，或其似乎損毀的時間點變慢。</span><span class="sxs-lookup"><span data-stu-id="a595a-142">The server may crash or slow to the point that it appears to have crashed.</span></span>

<span data-ttu-id="a595a-143">請考慮下列案例，以維護和顯示伺服器上潛在記憶體耗盡案例的相關專案清單：</span><span class="sxs-lookup"><span data-stu-id="a595a-143">Consider the following scenario for maintaining and displaying a list of items that pertain to a potential memory exhaustion scenario on the server:</span></span>

* <span data-ttu-id="a595a-144">`List<MyItem>`屬性或欄位中的專案會使用伺服器的記憶體。</span><span class="sxs-lookup"><span data-stu-id="a595a-144">The items in a `List<MyItem>` property or field use the server's memory.</span></span> <span data-ttu-id="a595a-145">如果應用程式允許專案清單不受限制地成長，伺服器就會發生記憶體不足的風險。</span><span class="sxs-lookup"><span data-stu-id="a595a-145">If the app allows the list of items to grow unbounded, there's a risk of the server running out of memory.</span></span> <span data-ttu-id="a595a-146">記憶體不足會導致目前的會話結束（損毀），而該伺服器實例中的所有並行會話都會收到記憶體不足的例外狀況。</span><span class="sxs-lookup"><span data-stu-id="a595a-146">Running out of memory causes the current session to end (crash) and all of the concurrent sessions in that server instance receive an out-of-memory exception.</span></span> <span data-ttu-id="a595a-147">為了避免發生這種情況，應用程式必須使用在並行使用者上強加專案限制的資料結構。</span><span class="sxs-lookup"><span data-stu-id="a595a-147">To prevent this scenario from occurring, the app must use a data structure that imposes an item limit on concurrent users.</span></span>
* <span data-ttu-id="a595a-148">如果分頁配置不會用於轉譯，伺服器會針對在 UI 中看不到的物件使用額外的記憶體。</span><span class="sxs-lookup"><span data-stu-id="a595a-148">If a paging scheme isn't used for rendering, the server uses additional memory for objects that aren't visible in the UI.</span></span> <span data-ttu-id="a595a-149">如果專案數沒有限制，記憶體需求可能會耗盡可用的伺服器記憶體。</span><span class="sxs-lookup"><span data-stu-id="a595a-149">Without a limit on the number of items, memory demands may exhaust the available server memory.</span></span> <span data-ttu-id="a595a-150">若要避免這種情況，請使用下列其中一種方法：</span><span class="sxs-lookup"><span data-stu-id="a595a-150">To prevent this scenario, use one of the following approaches:</span></span>
  * <span data-ttu-id="a595a-151">轉譯時使用分頁清單。</span><span class="sxs-lookup"><span data-stu-id="a595a-151">Use paginated lists when rendering.</span></span>
  * <span data-ttu-id="a595a-152">只顯示前100到1000個專案，並要求使用者輸入搜尋條件，以尋找超過所顯示專案的專案。</span><span class="sxs-lookup"><span data-stu-id="a595a-152">Only display the first 100 to 1,000 items and require the user to enter search criteria to find items beyond the items displayed.</span></span>
  * <span data-ttu-id="a595a-153">如需更先進的轉譯案例，請執行支援*虛擬化*的清單或格線。</span><span class="sxs-lookup"><span data-stu-id="a595a-153">For a more advanced rendering scenario, implement lists or grids that support *virtualization*.</span></span> <span data-ttu-id="a595a-154">使用虛擬化時，清單只會呈現使用者目前可見的專案子集。</span><span class="sxs-lookup"><span data-stu-id="a595a-154">Using virtualization, lists only render a subset of items currently visible to the user.</span></span> <span data-ttu-id="a595a-155">當使用者與 UI 中的捲軸互動時，元件只會轉譯顯示所需的專案。</span><span class="sxs-lookup"><span data-stu-id="a595a-155">When the user interacts with the scrollbar in the UI, the component renders only those items required for display.</span></span> <span data-ttu-id="a595a-156">目前不需要顯示的專案可以保留在次要儲存體中，這是理想的方法。</span><span class="sxs-lookup"><span data-stu-id="a595a-156">The items that aren't currently required for display can be held in secondary storage, which is the ideal approach.</span></span> <span data-ttu-id="a595a-157">Undisplayed 專案也可以保留在記憶體中，這比較不理想。</span><span class="sxs-lookup"><span data-stu-id="a595a-157">Undisplayed items can also be held in memory, which is less ideal.</span></span>

<span data-ttu-id="a595a-158">Blazor 伺服器應用程式會針對具狀態應用程式（例如 WPF、Windows Forms 或 Blazor WebAssembly）的其他 UI 架構提供類似的程式設計模型。</span><span class="sxs-lookup"><span data-stu-id="a595a-158">Blazor Server apps offer a similar programming model to other UI frameworks for stateful apps, such as WPF, Windows Forms, or Blazor WebAssembly.</span></span> <span data-ttu-id="a595a-159">主要的差異在於，在數個 UI 架構中，應用程式所耗用的記憶體屬於用戶端，而且只會影響該個別用戶端。</span><span class="sxs-lookup"><span data-stu-id="a595a-159">The main difference is that in several of the UI frameworks the memory consumed by the app belongs to the client and only affects that individual client.</span></span> <span data-ttu-id="a595a-160">例如，Blazor WebAssembly 應用程式會完全在用戶端上執行，而且只會使用用戶端記憶體資源。</span><span class="sxs-lookup"><span data-stu-id="a595a-160">For example, a Blazor WebAssembly app runs entirely on the client and only uses client memory resources.</span></span> <span data-ttu-id="a595a-161">在 Blazor 伺服器案例中，應用程式所耗用的記憶體屬於伺服器，而且會在伺服器實例的用戶端之間共用。</span><span class="sxs-lookup"><span data-stu-id="a595a-161">In the Blazor Server scenario, the memory consumed by the app belongs to the server and is shared among clients on the server instance.</span></span>

<span data-ttu-id="a595a-162">伺服器端記憶體需求是所有 Blazor 伺服器應用程式的考慮。</span><span class="sxs-lookup"><span data-stu-id="a595a-162">Server-side memory demands are a consideration for all Blazor Server apps.</span></span> <span data-ttu-id="a595a-163">不過，大部分的 web 應用程式都是無狀態的，而且在處理要求時所使用的記憶體會在傳迴響應時釋放。</span><span class="sxs-lookup"><span data-stu-id="a595a-163">However, most web apps are stateless, and the memory used while processing a request is released when the response is returned.</span></span> <span data-ttu-id="a595a-164">做為一般建議，請勿允許用戶端在保存用戶端連線的任何其他伺服器端應用程式中，配置未系結的記憶體數量。</span><span class="sxs-lookup"><span data-stu-id="a595a-164">As a general recommendation, don't permit clients to allocate an unbound amount of memory as in any other server-side app that persists client connections.</span></span> <span data-ttu-id="a595a-165">Blazor 伺服器應用程式所耗用的記憶體會持續一段時間，而不是單一要求。</span><span class="sxs-lookup"><span data-stu-id="a595a-165">The memory consumed by a Blazor Server app persists for a longer time than a single request.</span></span>

> [!NOTE]
> <span data-ttu-id="a595a-166">在開發期間，可以流量分析工具或捕捉到的追蹤來評估用戶端的記憶體需求。</span><span class="sxs-lookup"><span data-stu-id="a595a-166">During development, a profiler can be used or a trace captured to assess memory demands of clients.</span></span> <span data-ttu-id="a595a-167">Profiler 或追蹤不會捕獲配置給特定用戶端的記憶體。</span><span class="sxs-lookup"><span data-stu-id="a595a-167">A profiler or trace won't capture the memory allocated to a specific client.</span></span> <span data-ttu-id="a595a-168">若要在開發期間捕捉特定用戶端的記憶體使用量，請捕捉傾印，並檢查以使用者線路為根之所有物件的記憶體需求。</span><span class="sxs-lookup"><span data-stu-id="a595a-168">To capture the memory use of a specific client during development, capture a dump and examine the memory demand of all the objects rooted at a user's circuit.</span></span>

### <a name="client-connections"></a><span data-ttu-id="a595a-169">用戶端連接</span><span class="sxs-lookup"><span data-stu-id="a595a-169">Client connections</span></span>

<span data-ttu-id="a595a-170">當一或多個用戶端開啟太多與伺服器的並行連線，導致其他用戶端無法建立新的連線時，就會發生連線耗盡的情況。</span><span class="sxs-lookup"><span data-stu-id="a595a-170">Connection exhaustion can occur when one or more clients open too many concurrent connections to the server, preventing other clients from establishing new connections.</span></span>

<span data-ttu-id="a595a-171">Blazor 用戶端會在每個會話建立單一連線，只要開啟瀏覽器視窗，就會讓連線保持開啟狀態。</span><span class="sxs-lookup"><span data-stu-id="a595a-171">Blazor clients establish a single connection per session and keep the connection open for as long as the browser window is open.</span></span> <span data-ttu-id="a595a-172">維護所有連線的伺服器上的需求不是 Blazor 應用程式特有的。</span><span class="sxs-lookup"><span data-stu-id="a595a-172">The demands on the server of maintaining all of the connections isn't specific to Blazor apps.</span></span> <span data-ttu-id="a595a-173">由於連線的持續性和 Blazor 伺服器應用程式的具狀態本質，連接耗盡會對應用程式的可用性造成較大的風險。</span><span class="sxs-lookup"><span data-stu-id="a595a-173">Given the persistent nature of the connections and the stateful nature of Blazor Server apps, connection exhaustion is a greater risk to availability of the app.</span></span>

<span data-ttu-id="a595a-174">根據預設，Blazor 伺服器應用程式的每個使用者連線數目沒有限制。</span><span class="sxs-lookup"><span data-stu-id="a595a-174">By default, there's no limit on the number of connections per user for a Blazor Server app.</span></span> <span data-ttu-id="a595a-175">如果應用程式需要連線限制，請採取下列一或多種方法：</span><span class="sxs-lookup"><span data-stu-id="a595a-175">If the app requires a connection limit, take one or more of the following approaches:</span></span>

* <span data-ttu-id="a595a-176">需要驗證，這自然會限制未經授權的使用者連線到應用程式的能力。</span><span class="sxs-lookup"><span data-stu-id="a595a-176">Require authentication, which naturally limits the ability of unauthorized users to connect to the app.</span></span> <span data-ttu-id="a595a-177">為了讓此案例生效，使用者必須避免布建新的使用者。</span><span class="sxs-lookup"><span data-stu-id="a595a-177">For this scenario to be effective, users must be prevented from provisioning new users at will.</span></span>
* <span data-ttu-id="a595a-178">限制每個使用者的連接數目。</span><span class="sxs-lookup"><span data-stu-id="a595a-178">Limit the number of connections per user.</span></span> <span data-ttu-id="a595a-179">限制連接可以透過下列方法來完成。</span><span class="sxs-lookup"><span data-stu-id="a595a-179">Limiting connections can be accomplished via the following approaches.</span></span> <span data-ttu-id="a595a-180">請小心讓合法使用者存取應用程式（例如，根據用戶端的 IP 位址建立連線限制時）。</span><span class="sxs-lookup"><span data-stu-id="a595a-180">Exercise care to allow legitimate users to access the app (for example, when a connection limit is established based on the client's IP address).</span></span>
  * <span data-ttu-id="a595a-181">在應用層級：</span><span class="sxs-lookup"><span data-stu-id="a595a-181">At the app level:</span></span>
    * <span data-ttu-id="a595a-182">端點路由擴充性。</span><span class="sxs-lookup"><span data-stu-id="a595a-182">Endpoint routing extensibility.</span></span>
    * <span data-ttu-id="a595a-183">需要驗證才能連接到應用程式，並追蹤每位使用者的使用中會話。</span><span class="sxs-lookup"><span data-stu-id="a595a-183">Require authentication to connect to the app and keep track of the active sessions per user.</span></span>
    * <span data-ttu-id="a595a-184">在達到限制時拒絕新的會話。</span><span class="sxs-lookup"><span data-stu-id="a595a-184">Reject new sessions upon reaching a limit.</span></span>
    * <span data-ttu-id="a595a-185">Proxy WebSocket 透過使用 proxy 連線至應用程式，例如從用戶端將分離信號連線到應用程式的[Azure SignalR Service](/azure/azure-signalr/signalr-overview) 。</span><span class="sxs-lookup"><span data-stu-id="a595a-185">Proxy WebSocket connections to an app through the use of a proxy, such as the [Azure SignalR Service](/azure/azure-signalr/signalr-overview) that multiplexes connections from clients to an app.</span></span> <span data-ttu-id="a595a-186">這會提供比單一用戶端可以建立的連接容量更大的應用程式，以防止用戶端耗盡與伺服器的連接。</span><span class="sxs-lookup"><span data-stu-id="a595a-186">This provides an app with greater connection capacity than a single client can establish, preventing a client from exhausting the connections to the server.</span></span>
  * <span data-ttu-id="a595a-187">在伺服器層級：使用應用程式前方的 proxy/閘道。</span><span class="sxs-lookup"><span data-stu-id="a595a-187">At the server level: Use a proxy/gateway in front of the app.</span></span> <span data-ttu-id="a595a-188">例如， [Azure Front 門板](/azure/frontdoor/front-door-overview)可讓您定義、管理及監視應用程式的網路流量全域路由。</span><span class="sxs-lookup"><span data-stu-id="a595a-188">For example, [Azure Front Door](/azure/frontdoor/front-door-overview) enables you to define, manage, and monitor the global routing of web traffic to an app.</span></span>

## <a name="denial-of-service-dos-attacks"></a><span data-ttu-id="a595a-189">拒絕服務（DoS）攻擊</span><span class="sxs-lookup"><span data-stu-id="a595a-189">Denial of service (DoS) attacks</span></span>

<span data-ttu-id="a595a-190">阻絕服務（DoS）攻擊牽涉到用戶端導致伺服器耗盡其一或多項資源，讓應用程式無法使用。</span><span class="sxs-lookup"><span data-stu-id="a595a-190">Denial of service (DoS) attacks involve a client causing the server to exhaust one or more of its resources making the app unavailable.</span></span> <span data-ttu-id="a595a-191">Blazor 伺服器應用程式包含一些預設限制，並依賴其他 ASP.NET Core 和 SignalR 限制來防範 DoS 攻擊：</span><span class="sxs-lookup"><span data-stu-id="a595a-191">Blazor Server apps include some default limits and rely on other ASP.NET Core and SignalR limits to protect against DoS attacks:</span></span>

| <span data-ttu-id="a595a-192">Blazor 伺服器應用程式限制</span><span class="sxs-lookup"><span data-stu-id="a595a-192">Blazor Server app limit</span></span>                            | <span data-ttu-id="a595a-193">描述</span><span class="sxs-lookup"><span data-stu-id="a595a-193">Description</span></span> | <span data-ttu-id="a595a-194">預設</span><span class="sxs-lookup"><span data-stu-id="a595a-194">Default</span></span> |
| ------------------------------------------------------- | ----------- | ------- |
| `CircuitOptions.DisconnectedCircuitMaxRetained`         | <span data-ttu-id="a595a-195">給定伺服器一次保存在記憶體中的中斷連線線路數目上限。</span><span class="sxs-lookup"><span data-stu-id="a595a-195">Maximum number of disconnected circuits that a given server holds in memory at a time.</span></span> | <span data-ttu-id="a595a-196">100</span><span class="sxs-lookup"><span data-stu-id="a595a-196">100</span></span> |
| `CircuitOptions.DisconnectedCircuitRetentionPeriod`     | <span data-ttu-id="a595a-197">中斷連線的線路在損毀之前，保留在記憶體中的最大時間量。</span><span class="sxs-lookup"><span data-stu-id="a595a-197">Maximum amount of time a disconnected circuit is held in memory before being torn down.</span></span> | <span data-ttu-id="a595a-198">3分鐘</span><span class="sxs-lookup"><span data-stu-id="a595a-198">3 minutes</span></span> |
| `CircuitOptions.JSInteropDefaultCallTimeout`            | <span data-ttu-id="a595a-199">伺服器在計時非同步 JavaScript 函式呼叫之前等待的最大時間量。</span><span class="sxs-lookup"><span data-stu-id="a595a-199">Maximum amount of time the server waits before timing out an asynchronous JavaScript function invocation.</span></span> | <span data-ttu-id="a595a-200">1 分鐘</span><span class="sxs-lookup"><span data-stu-id="a595a-200">1 minute</span></span> |
| `CircuitOptions.MaxBufferedUnacknowledgedRenderBatches` | <span data-ttu-id="a595a-201">伺服器在指定時間將每個線路保留在記憶體中的未認可轉譯批次數目上限，以支援健全的重新連接。</span><span class="sxs-lookup"><span data-stu-id="a595a-201">Maximum number of unacknowledged render batches the server keeps in memory per circuit at a given time to support robust reconnection.</span></span> <span data-ttu-id="a595a-202">達到此限制之後，伺服器就會停止產生新的轉譯批次，直到用戶端認可一或多個批次為止。</span><span class="sxs-lookup"><span data-stu-id="a595a-202">After reaching the limit, the server stops producing new render batches until one or more batches have been acknowledged by the client.</span></span> | <span data-ttu-id="a595a-203">10</span><span class="sxs-lookup"><span data-stu-id="a595a-203">10</span></span> |


| <span data-ttu-id="a595a-204">SignalR 和 ASP.NET Core 限制</span><span class="sxs-lookup"><span data-stu-id="a595a-204">SignalR and ASP.NET Core limit</span></span>             | <span data-ttu-id="a595a-205">描述</span><span class="sxs-lookup"><span data-stu-id="a595a-205">Description</span></span> | <span data-ttu-id="a595a-206">預設</span><span class="sxs-lookup"><span data-stu-id="a595a-206">Default</span></span> |
| ------------------------------------------ | ----------- | ------- |
| `CircuitOptions.MaximumReceiveMessageSize` | <span data-ttu-id="a595a-207">個別訊息的訊息大小。</span><span class="sxs-lookup"><span data-stu-id="a595a-207">Message size for an individual message.</span></span> | <span data-ttu-id="a595a-208">32 KB</span><span class="sxs-lookup"><span data-stu-id="a595a-208">32 KB</span></span> |

## <a name="interactions-with-the-browser-client"></a><span data-ttu-id="a595a-209">與瀏覽器的互動（用戶端）</span><span class="sxs-lookup"><span data-stu-id="a595a-209">Interactions with the browser (client)</span></span>

<span data-ttu-id="a595a-210">用戶端會透過 JS interop 事件分派和轉譯完成來與伺服器互動。</span><span class="sxs-lookup"><span data-stu-id="a595a-210">A client interacts with the server through JS interop event dispatching and render completion.</span></span> <span data-ttu-id="a595a-211">JS interop 通訊會在 JavaScript 和 .NET 之間進行這兩種方式：</span><span class="sxs-lookup"><span data-stu-id="a595a-211">JS interop communication goes both ways between JavaScript and .NET:</span></span>

* <span data-ttu-id="a595a-212">瀏覽器事件會以非同步方式從用戶端分派至伺服器。</span><span class="sxs-lookup"><span data-stu-id="a595a-212">Browser events are dispatched from the client to the server in an asynchronous fashion.</span></span>
* <span data-ttu-id="a595a-213">伺服器會視需要以非同步方式回應 rerendering UI。</span><span class="sxs-lookup"><span data-stu-id="a595a-213">The server responds asynchronously rerendering the UI as necessary.</span></span>

### <a name="javascript-functions-invoked-from-net"></a><span data-ttu-id="a595a-214">從 .NET 叫用的 JavaScript 函式</span><span class="sxs-lookup"><span data-stu-id="a595a-214">JavaScript functions invoked from .NET</span></span>

<span data-ttu-id="a595a-215">針對從 .NET 方法到 JavaScript 的呼叫：</span><span class="sxs-lookup"><span data-stu-id="a595a-215">For calls from .NET methods to JavaScript:</span></span>

* <span data-ttu-id="a595a-216">所有調用都有一個可設定的超時時間，在這<xref:System.OperationCanceledException>之後，會將傳回給呼叫者。</span><span class="sxs-lookup"><span data-stu-id="a595a-216">All invocations have a configurable timeout after which they fail, returning a <xref:System.OperationCanceledException> to the caller.</span></span>
  * <span data-ttu-id="a595a-217">呼叫（`CircuitOptions.JSInteropDefaultCallTimeout`）的預設超時時間為一分鐘。</span><span class="sxs-lookup"><span data-stu-id="a595a-217">There's a default timeout for the calls (`CircuitOptions.JSInteropDefaultCallTimeout`) of one minute.</span></span> <span data-ttu-id="a595a-218">若要設定此限制， <xref:blazor/javascript-interop#harden-js-interop-calls>請參閱。</span><span class="sxs-lookup"><span data-stu-id="a595a-218">To configure this limit, see <xref:blazor/javascript-interop#harden-js-interop-calls>.</span></span>
  * <span data-ttu-id="a595a-219">可以提供解除標記來控制每個呼叫的取消。</span><span class="sxs-lookup"><span data-stu-id="a595a-219">A cancellation token can be provided to control the cancellation on a per-call basis.</span></span> <span data-ttu-id="a595a-220">如果提供解除標記，則依賴預設的呼叫超時時間（如果有的話，也可以呼叫用戶端）。</span><span class="sxs-lookup"><span data-stu-id="a595a-220">Rely on the default call timeout where possible and time-bound any call to the client if a cancellation token is provided.</span></span>
* <span data-ttu-id="a595a-221">無法信任 JavaScript 呼叫的結果。</span><span class="sxs-lookup"><span data-stu-id="a595a-221">The result of a JavaScript call can't be trusted.</span></span> <span data-ttu-id="a595a-222">在瀏覽器中執行的 Blazor 應用程式用戶端會搜尋 JavaScript 函數來叫用。</span><span class="sxs-lookup"><span data-stu-id="a595a-222">The Blazor app client running in the browser searches for the JavaScript function to invoke.</span></span> <span data-ttu-id="a595a-223">系統會叫用函式，並產生結果或錯誤。</span><span class="sxs-lookup"><span data-stu-id="a595a-223">The function is invoked, and either the result or an error is produced.</span></span> <span data-ttu-id="a595a-224">惡意用戶端可以嘗試：</span><span class="sxs-lookup"><span data-stu-id="a595a-224">A malicious client can attempt to:</span></span>
  * <span data-ttu-id="a595a-225">從 JavaScript 函式傳回錯誤，導致應用程式發生問題。</span><span class="sxs-lookup"><span data-stu-id="a595a-225">Cause an issue in the app by returning an error from the JavaScript function.</span></span>
  * <span data-ttu-id="a595a-226">藉由從 JavaScript 函式傳回非預期的結果，在伺服器上引發非預期的行為。</span><span class="sxs-lookup"><span data-stu-id="a595a-226">Induce an unintended behavior on the server by returning an unexpected result from the JavaScript function.</span></span>

<span data-ttu-id="a595a-227">請採取下列預防措施來防範前述案例：</span><span class="sxs-lookup"><span data-stu-id="a595a-227">Take the following precautions to guard against the preceding scenarios:</span></span>

* <span data-ttu-id="a595a-228">在[try catch](/dotnet/csharp/language-reference/keywords/try-catch)語句中包裝 JS interop 呼叫，以考慮調用期間可能發生的錯誤。</span><span class="sxs-lookup"><span data-stu-id="a595a-228">Wrap JS interop calls within [try-catch](/dotnet/csharp/language-reference/keywords/try-catch) statements to account for errors that might occur during the invocations.</span></span> <span data-ttu-id="a595a-229">如需詳細資訊，請參閱<xref:blazor/handle-errors#javascript-interop>。</span><span class="sxs-lookup"><span data-stu-id="a595a-229">For more information, see <xref:blazor/handle-errors#javascript-interop>.</span></span>
* <span data-ttu-id="a595a-230">在採取任何動作之前，請先驗證從 JS interop 調用傳回的資料，包括錯誤訊息。</span><span class="sxs-lookup"><span data-stu-id="a595a-230">Validate data returned from JS interop invocations, including error messages, before taking any action.</span></span>

### <a name="net-methods-invoked-from-the-browser"></a><span data-ttu-id="a595a-231">從瀏覽器叫用的 .NET 方法</span><span class="sxs-lookup"><span data-stu-id="a595a-231">.NET methods invoked from the browser</span></span>

<span data-ttu-id="a595a-232">不信任從 JavaScript 到 .NET 方法的呼叫。</span><span class="sxs-lookup"><span data-stu-id="a595a-232">Don't trust calls from JavaScript to .NET methods.</span></span> <span data-ttu-id="a595a-233">將 .NET 方法公開給 JavaScript 時，請考慮 .NET 方法的叫用方式：</span><span class="sxs-lookup"><span data-stu-id="a595a-233">When a .NET method is exposed to JavaScript, consider how the .NET method is invoked:</span></span>

* <span data-ttu-id="a595a-234">將任何公開給 JavaScript 的 .NET 方法視為應用程式的公用端點。</span><span class="sxs-lookup"><span data-stu-id="a595a-234">Treat any .NET method exposed to JavaScript as you would a public endpoint to the app.</span></span>
  * <span data-ttu-id="a595a-235">驗證輸入。</span><span class="sxs-lookup"><span data-stu-id="a595a-235">Validate input.</span></span>
    * <span data-ttu-id="a595a-236">確定值在預期的範圍內。</span><span class="sxs-lookup"><span data-stu-id="a595a-236">Ensure that values are within expected ranges.</span></span>
    * <span data-ttu-id="a595a-237">請確定使用者具有執行所要求動作的許可權。</span><span class="sxs-lookup"><span data-stu-id="a595a-237">Ensure that the user has permission to perform the action requested.</span></span>
  * <span data-ttu-id="a595a-238">請勿在 .NET 方法調用中配置過多的資源數量。</span><span class="sxs-lookup"><span data-stu-id="a595a-238">Don't allocate an excessive quantity of resources as part of the .NET method invocation.</span></span> <span data-ttu-id="a595a-239">例如，執行檢查並對 CPU 和記憶體使用量進行限制。</span><span class="sxs-lookup"><span data-stu-id="a595a-239">For example, perform checks and place limits on CPU and memory use.</span></span>
  * <span data-ttu-id="a595a-240">請考慮靜態和實例方法可以公開給 JavaScript 用戶端。</span><span class="sxs-lookup"><span data-stu-id="a595a-240">Take into account that static and instance methods can be exposed to JavaScript clients.</span></span> <span data-ttu-id="a595a-241">避免在會話間共用狀態，除非設計呼叫具有適當條件約束的共用狀態。</span><span class="sxs-lookup"><span data-stu-id="a595a-241">Avoid sharing state across sessions unless the design calls for sharing state with appropriate constraints.</span></span>
    * <span data-ttu-id="a595a-242">若為透過`DotNetReference`相依性插入（DI）所建立的物件所公開的實例方法，則應該將物件註冊為已設定範圍的物件。</span><span class="sxs-lookup"><span data-stu-id="a595a-242">For instance methods exposed through `DotNetReference` objects that are originally created through dependency injection (DI), the objects should be registered as scoped objects.</span></span> <span data-ttu-id="a595a-243">這適用于 Blazor 伺服器應用程式所使用的任何 DI 服務。</span><span class="sxs-lookup"><span data-stu-id="a595a-243">This applies to any DI service that the Blazor Server app uses.</span></span>
    * <span data-ttu-id="a595a-244">針對靜態方法，除非應用程式在伺服器實例上的所有使用者之間依設計明確共用狀態，否則請避免建立不能限定于用戶端範圍的狀態。</span><span class="sxs-lookup"><span data-stu-id="a595a-244">For static methods, avoid establishing state that can't be scoped to the client unless the app is explicitly sharing state by-design across all users on a server instance.</span></span>
  * <span data-ttu-id="a595a-245">避免將參數中使用者提供的資料傳遞至 JavaScript 呼叫。</span><span class="sxs-lookup"><span data-stu-id="a595a-245">Avoid passing user-supplied data in parameters to JavaScript calls.</span></span> <span data-ttu-id="a595a-246">如果絕對需要在參數中傳遞資料，請確定 JavaScript 程式碼會在不引進[跨網站腳本（XSS）](#cross-site-scripting-xss)弱點的情況下，處理傳遞資料。</span><span class="sxs-lookup"><span data-stu-id="a595a-246">If passing data in parameters is absolutely required, ensure that the JavaScript code handles passing the data without introducing [Cross-site scripting (XSS)](#cross-site-scripting-xss) vulnerabilities.</span></span> <span data-ttu-id="a595a-247">例如，請不要藉由設定`innerHTML`專案的屬性，將使用者提供的資料寫入檔物件模型（DOM）。</span><span class="sxs-lookup"><span data-stu-id="a595a-247">For example, don't write user-supplied data to the Document Object Model (DOM) by setting the `innerHTML` property of an element.</span></span> <span data-ttu-id="a595a-248">請考慮使用[內容安全性原則（CSP）](https://developer.mozilla.org/docs/Web/HTTP/CSP)來`eval`停用和其他不安全的 JavaScript 基本類型。</span><span class="sxs-lookup"><span data-stu-id="a595a-248">Consider using [Content Security Policy (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP) to disable `eval` and other unsafe JavaScript primitives.</span></span>
* <span data-ttu-id="a595a-249">避免在架構的分派執行上，執行 .NET 調用的自訂分派。</span><span class="sxs-lookup"><span data-stu-id="a595a-249">Avoid implementing custom dispatching of .NET invocations on top of the framework's dispatching implementation.</span></span> <span data-ttu-id="a595a-250">將 .NET 方法公開至瀏覽器是一個先進的案例，不建議用於一般的 Blazor 開發。</span><span class="sxs-lookup"><span data-stu-id="a595a-250">Exposing .NET methods to the browser is an advanced scenario, not recommended for general Blazor development.</span></span>

### <a name="events"></a><span data-ttu-id="a595a-251">事件</span><span class="sxs-lookup"><span data-stu-id="a595a-251">Events</span></span>

<span data-ttu-id="a595a-252">事件會提供 Blazor 伺服器應用程式的進入點。</span><span class="sxs-lookup"><span data-stu-id="a595a-252">Events provide an entry point to a Blazor Server app.</span></span> <span data-ttu-id="a595a-253">Web apps 中保護端點的相同規則適用于 Blazor 伺服器應用程式中的事件處理。</span><span class="sxs-lookup"><span data-stu-id="a595a-253">The same rules for safeguarding endpoints in web apps apply to event handling in Blazor Server apps.</span></span> <span data-ttu-id="a595a-254">惡意用戶端可以將任何想要傳送的資料傳送為事件的內容。</span><span class="sxs-lookup"><span data-stu-id="a595a-254">A malicious client can send any data it wishes to send as the payload for an event.</span></span>

<span data-ttu-id="a595a-255">例如：</span><span class="sxs-lookup"><span data-stu-id="a595a-255">For example:</span></span>

* <span data-ttu-id="a595a-256">的變更事件`<select>`可能會傳送的值不在應用程式呈現給用戶端的選項內。</span><span class="sxs-lookup"><span data-stu-id="a595a-256">A change event for a `<select>` could send a value that isn't within the options that the app presented to the client.</span></span>
* <span data-ttu-id="a595a-257">`<input>`可以將任何文字資料傳送至伺服器，略過用戶端驗證。</span><span class="sxs-lookup"><span data-stu-id="a595a-257">An `<input>` could send any text data to the server, bypassing client-side validation.</span></span>

<span data-ttu-id="a595a-258">應用程式必須驗證應用程式所處理之任何事件的資料。</span><span class="sxs-lookup"><span data-stu-id="a595a-258">The app must validate the data for any event that the app handles.</span></span> <span data-ttu-id="a595a-259">Blazor framework [forms 元件](xref:blazor/forms-validation)會執行基本驗證。</span><span class="sxs-lookup"><span data-stu-id="a595a-259">The Blazor framework [forms components](xref:blazor/forms-validation) perform basic validations.</span></span> <span data-ttu-id="a595a-260">如果應用程式使用自訂表單元件，就必須撰寫自訂程式碼來適當地驗證事件資料。</span><span class="sxs-lookup"><span data-stu-id="a595a-260">If the app uses custom forms components, custom code must be written to validate event data as appropriate.</span></span>

<span data-ttu-id="a595a-261">Blazor 伺服器事件是非同步，因此，您可以將多個事件分派至伺服器，應用程式才會產生新的轉譯來做出反應。</span><span class="sxs-lookup"><span data-stu-id="a595a-261">Blazor Server events are asynchronous, so multiple events can be dispatched to the server before the app has time to react by producing a new render.</span></span> <span data-ttu-id="a595a-262">這有一些要考慮的安全性含意。</span><span class="sxs-lookup"><span data-stu-id="a595a-262">This has some security implications to consider.</span></span> <span data-ttu-id="a595a-263">限制應用程式中的用戶端動作必須在事件處理常式內部執行，而不是取決於目前呈現的檢視狀態。</span><span class="sxs-lookup"><span data-stu-id="a595a-263">Limiting client actions in the app must be performed inside event handlers and not depend on the current rendered view state.</span></span>

<span data-ttu-id="a595a-264">假設有一個「計數器」元件，應該允許使用者增加最多三次的計數器。</span><span class="sxs-lookup"><span data-stu-id="a595a-264">Consider a counter component that should allow a user to increment a counter a maximum of three times.</span></span> <span data-ttu-id="a595a-265">要遞增計數器的按鈕是根據的值`count`而有條件的：</span><span class="sxs-lookup"><span data-stu-id="a595a-265">The button to increment the counter is conditionally based on the value of `count`:</span></span>

```cshtml
<p>Count: @count<p>

@if (count < 3)
{
    <button @onclick="IncrementCount" value="Increment count" />
}

@code 
{
    private int count = 0;

    private void IncrementCount()
    {
        count++;
    }
}
```

<span data-ttu-id="a595a-266">用戶端可以分派一或多個增量事件，架構才會產生這個元件的新轉譯。</span><span class="sxs-lookup"><span data-stu-id="a595a-266">A client can dispatch one or more increment events before the framework produces a new render of this component.</span></span> <span data-ttu-id="a595a-267">結果是`count` ，使用者可以遞增*三次*，因為 UI 不會快速移除按鈕。</span><span class="sxs-lookup"><span data-stu-id="a595a-267">The result is that the `count` can be incremented *over three times* by the user because the button isn't removed by the UI quickly enough.</span></span> <span data-ttu-id="a595a-268">下列範例會顯示達成三個`count`增量限制的正確方式：</span><span class="sxs-lookup"><span data-stu-id="a595a-268">The correct way to achieve the limit of three `count` increments is shown in the following example:</span></span>

```cshtml
<p>Count: @count<p>

@if (count < 3)
{
    <button @onclick="IncrementCount" value="Increment count" />
}

@code 
{
    private int count = 0;

    private void IncrementCount()
    {
        if (count < 3)
        {
            count++;
        }
    }
}
```

<span data-ttu-id="a595a-269">藉由在`if (count < 3) { ... }`處理常式內加入檢查，就會根據`count`目前的應用程式狀態來遞增決策。</span><span class="sxs-lookup"><span data-stu-id="a595a-269">By adding the `if (count < 3) { ... }` check inside the handler, the decision to increment `count` is based on the current app state.</span></span> <span data-ttu-id="a595a-270">這項決策不是以上一個範例中的 UI 狀態為基礎，這可能會暫時過時。</span><span class="sxs-lookup"><span data-stu-id="a595a-270">The decision isn't based on the state of the UI as it was in the previous example, which might be temporarily stale.</span></span>

### <a name="guard-against-multiple-dispatches"></a><span data-ttu-id="a595a-271">防護多個分派</span><span class="sxs-lookup"><span data-stu-id="a595a-271">Guard against multiple dispatches</span></span>

<span data-ttu-id="a595a-272">如果事件回呼叫用長時間執行的作業（例如從外部服務或資料庫提取資料），請考慮使用「防護」。</span><span class="sxs-lookup"><span data-stu-id="a595a-272">If an event callback invokes a long running operation, such as fetching data from an external service or database, consider using a guard.</span></span> <span data-ttu-id="a595a-273">此防護可以防止使用者在作業進行中時，使用視覺效果的意見反應來排入多個作業。</span><span class="sxs-lookup"><span data-stu-id="a595a-273">The guard can prevent the user from queueing up multiple operations while the operation is in progress with visual feedback.</span></span> <span data-ttu-id="a595a-274">下列元件程式碼會`isLoading`將`true`設定`GetForecastAsync`為，同時從伺服器取得資料。</span><span class="sxs-lookup"><span data-stu-id="a595a-274">The following component code sets `isLoading` to `true` while `GetForecastAsync` obtains data from the server.</span></span> <span data-ttu-id="a595a-275">當`isLoading` 為`true`時，UI 中的按鈕會停用：</span><span class="sxs-lookup"><span data-stu-id="a595a-275">While `isLoading` is `true`, the button is disabled in the UI:</span></span>

```cshtml
@page "/fetchdata"
@using BlazorServerSample.Data
@inject WeatherForecastService ForecastService

<button disabled="@isLoading" @onclick="UpdateForecasts">Update</button>

@code {
    private bool isLoading;
    private WeatherForecast[] forecasts;

    private async Task UpdateForecasts()
    {
        if (!isLoading)
        {
            isLoading = true;
            forecasts = await ForecastService.GetForecastAsync(DateTime.Now);
            isLoading = false;
        }
    }
}
```

### <a name="cancel-early-and-avoid-use-after-dispose"></a><span data-ttu-id="a595a-276">及早取消並避免使用-處置後</span><span class="sxs-lookup"><span data-stu-id="a595a-276">Cancel early and avoid use-after-dispose</span></span>

<span data-ttu-id="a595a-277">除了使用防護[多個分派](#guard-against-multiple-dispatches)一節中所述的防護以外，請考慮在處置<xref:System.Threading.CancellationToken>元件時，使用來取消長時間執行的作業。</span><span class="sxs-lookup"><span data-stu-id="a595a-277">In addition to using a guard as described in the [Guard against multiple dispatches](#guard-against-multiple-dispatches) section, consider using a <xref:System.Threading.CancellationToken> to cancel long-running operations when the component is disposed.</span></span> <span data-ttu-id="a595a-278">這種方法的優點是避免在元件中*使用-dispose* ：</span><span class="sxs-lookup"><span data-stu-id="a595a-278">This approach has the added benefit of avoiding *use-after-dispose* in components:</span></span>

```cshtml
@implements IDisposable

...

@code {
    private readonly CancellationTokenSource TokenSource = 
        new CancellationTokenSource();

    private async Task UpdateForecasts()
    {
        ...

        forecasts = await ForecastService.GetForecastAsync(DateTime.Now, 
            TokenSource.Token);

        if (TokenSource.Token.IsCancellationRequested)
        {
           return;
        }

        ...
    }

    public void Dispose()
    {
        CancellationTokenSource.Cancel();
    }
}
```

### <a name="avoid-events-that-produce-large-amounts-of-data"></a><span data-ttu-id="a595a-279">避免產生大量資料的事件</span><span class="sxs-lookup"><span data-stu-id="a595a-279">Avoid events that produce large amounts of data</span></span>

<span data-ttu-id="a595a-280">某些 DOM 事件（例如`oninput`或`onscroll`）可能會產生大量資料。</span><span class="sxs-lookup"><span data-stu-id="a595a-280">Some DOM events, such as `oninput` or `onscroll`, can produce a large amount of data.</span></span> <span data-ttu-id="a595a-281">避免在 Blazor 伺服器應用程式中使用這些事件。</span><span class="sxs-lookup"><span data-stu-id="a595a-281">Avoid using these events in Blazor server apps.</span></span>

## <a name="additional-security-guidance"></a><span data-ttu-id="a595a-282">其他安全性指引</span><span class="sxs-lookup"><span data-stu-id="a595a-282">Additional security guidance</span></span>

<span data-ttu-id="a595a-283">保護 ASP.NET Core 應用程式的指導方針適用于 Blazor 伺服器應用程式，並將在下列各節中討論：</span><span class="sxs-lookup"><span data-stu-id="a595a-283">The guidance for securing ASP.NET Core apps apply to Blazor Server apps and are covered in the following sections:</span></span>

* [<span data-ttu-id="a595a-284">記錄和敏感性資料</span><span class="sxs-lookup"><span data-stu-id="a595a-284">Logging and sensitive data</span></span>](#logging-and-sensitive-data)
* [<span data-ttu-id="a595a-285">使用 HTTPS 保護傳輸中的資訊</span><span class="sxs-lookup"><span data-stu-id="a595a-285">Protect information in transit with HTTPS</span></span>](#protect-information-in-transit-with-https)
* <span data-ttu-id="a595a-286">[跨網站腳本（XSS）](#cross-site-scripting-xss)</span><span class="sxs-lookup"><span data-stu-id="a595a-286">[Cross-site scripting (XSS)](#cross-site-scripting-xss))</span></span>
* [<span data-ttu-id="a595a-287">跨原始來源保護</span><span class="sxs-lookup"><span data-stu-id="a595a-287">Cross-origin protection</span></span>](#cross-origin-protection)
* [<span data-ttu-id="a595a-288">按一下-劫持</span><span class="sxs-lookup"><span data-stu-id="a595a-288">Click-jacking</span></span>](#click-jacking)
* [<span data-ttu-id="a595a-289">開啟重新導向</span><span class="sxs-lookup"><span data-stu-id="a595a-289">Open redirects</span></span>](#open-redirects)

### <a name="logging-and-sensitive-data"></a><span data-ttu-id="a595a-290">記錄和敏感性資料</span><span class="sxs-lookup"><span data-stu-id="a595a-290">Logging and sensitive data</span></span>

<span data-ttu-id="a595a-291">用戶端與伺服器之間的 JS interop 互動會記錄在具有<xref:Microsoft.Extensions.Logging.ILogger>實例的伺服器記錄中。</span><span class="sxs-lookup"><span data-stu-id="a595a-291">JS interop interactions between the client and server are recorded in the server's logs with <xref:Microsoft.Extensions.Logging.ILogger> instances.</span></span> <span data-ttu-id="a595a-292">Blazor 可避免記錄敏感性資訊，例如實際事件或 JS interop 輸入和輸出。</span><span class="sxs-lookup"><span data-stu-id="a595a-292">Blazor avoids logging sensitive information, such as actual events or JS interop inputs and outputs.</span></span>

<span data-ttu-id="a595a-293">當伺服器上發生錯誤時，架構會通知用戶端並向下眼淚會話。</span><span class="sxs-lookup"><span data-stu-id="a595a-293">When an error occurs on the server, the framework notifies the client and tears down the session.</span></span> <span data-ttu-id="a595a-294">根據預設，用戶端會收到一般錯誤訊息，可在瀏覽器的開發人員工具中看到。</span><span class="sxs-lookup"><span data-stu-id="a595a-294">By default, the client receives a generic error message that can be seen in the browser's developer tools.</span></span>

<span data-ttu-id="a595a-295">用戶端錯誤不會包含呼叫堆疊，也不會提供錯誤原因的詳細資料，但伺服器記錄檔包含這類資訊。</span><span class="sxs-lookup"><span data-stu-id="a595a-295">The client-side error doesn't include the callstack and doesn't provide detail on the cause of the error, but server logs do contain such information.</span></span> <span data-ttu-id="a595a-296">基於開發目的，您可以藉由啟用詳細錯誤，將敏感性錯誤資訊提供給用戶端。</span><span class="sxs-lookup"><span data-stu-id="a595a-296">For development purposes, sensitive error information can be made available to the client by enabling detailed errors.</span></span>

<span data-ttu-id="a595a-297">使用下列內容啟用詳細錯誤：</span><span class="sxs-lookup"><span data-stu-id="a595a-297">Enable detailed errors with:</span></span>

* <span data-ttu-id="a595a-298">`CircuitOptions.DetailedErrors`.</span><span class="sxs-lookup"><span data-stu-id="a595a-298">`CircuitOptions.DetailedErrors`.</span></span>
* <span data-ttu-id="a595a-299">`DetailedErrors`設定金鑰。</span><span class="sxs-lookup"><span data-stu-id="a595a-299">`DetailedErrors` configuration key.</span></span> <span data-ttu-id="a595a-300">例如，將`ASPNETCORE_DETAILEDERRORS`環境變數設定為的`true`值。</span><span class="sxs-lookup"><span data-stu-id="a595a-300">For example, set the `ASPNETCORE_DETAILEDERRORS` environment variable to a value of `true`.</span></span>

> [!WARNING]
> <span data-ttu-id="a595a-301">將錯誤資訊公開給網際網路上的用戶端，是應一律避免的安全性風險。</span><span class="sxs-lookup"><span data-stu-id="a595a-301">Exposing error information to clients on the Internet is a security risk that should always be avoided.</span></span>

### <a name="protect-information-in-transit-with-https"></a><span data-ttu-id="a595a-302">使用 HTTPS 保護傳輸中的資訊</span><span class="sxs-lookup"><span data-stu-id="a595a-302">Protect information in transit with HTTPS</span></span>

<span data-ttu-id="a595a-303">Blazor 伺服器會使用 SignalR 來進行用戶端與伺服器之間的通訊。</span><span class="sxs-lookup"><span data-stu-id="a595a-303">Blazor Server uses SignalR for communication between the client and the server.</span></span> <span data-ttu-id="a595a-304">Blazor 伺服器通常會使用 SignalR 協商的傳輸，這通常是 Websocket。</span><span class="sxs-lookup"><span data-stu-id="a595a-304">Blazor Server normally uses the transport that SignalR negotiates, which is typically WebSockets.</span></span>

<span data-ttu-id="a595a-305">Blazor 伺服器不會確保在伺服器與用戶端之間傳送之資料的完整性與機密性。</span><span class="sxs-lookup"><span data-stu-id="a595a-305">Blazor Server doesn't ensure the integrity and confidentiality of the data sent between the server and the client.</span></span> <span data-ttu-id="a595a-306">一律使用 HTTPS。</span><span class="sxs-lookup"><span data-stu-id="a595a-306">Always use HTTPS.</span></span>

### <a name="cross-site-scripting-xss"></a><span data-ttu-id="a595a-307">跨網站腳本（XSS）</span><span class="sxs-lookup"><span data-stu-id="a595a-307">Cross-site scripting (XSS)</span></span>

<span data-ttu-id="a595a-308">跨網站腳本（XSS）可讓未經授權的合作物件在瀏覽器的內容中執行任意邏輯。</span><span class="sxs-lookup"><span data-stu-id="a595a-308">Cross-site scripting (XSS) allows an unauthorized party to execute arbitrary logic in the context of the browser.</span></span> <span data-ttu-id="a595a-309">遭入侵的應用程式可能會在用戶端上執行任意程式碼。</span><span class="sxs-lookup"><span data-stu-id="a595a-309">A compromised app could potentially run arbitrary code on the client.</span></span> <span data-ttu-id="a595a-310">此弱點可能會用來對伺服器執行一些惡意動作：</span><span class="sxs-lookup"><span data-stu-id="a595a-310">The vulnerability could be used to potentially perform a number of malicious actions against the server:</span></span>

* <span data-ttu-id="a595a-311">將假/無效事件分派至伺服器。</span><span class="sxs-lookup"><span data-stu-id="a595a-311">Dispatch fake/invalid events to the server.</span></span>
* <span data-ttu-id="a595a-312">分派失敗/不正確轉譯完成。</span><span class="sxs-lookup"><span data-stu-id="a595a-312">Dispatch fail/invalid render completions.</span></span>
* <span data-ttu-id="a595a-313">避免分派轉譯完成。</span><span class="sxs-lookup"><span data-stu-id="a595a-313">Avoid dispatching render completions.</span></span>
* <span data-ttu-id="a595a-314">將來自 JavaScript 的 interop 呼叫分派至 .NET。</span><span class="sxs-lookup"><span data-stu-id="a595a-314">Dispatch interop calls from JavaScript to .NET.</span></span>
* <span data-ttu-id="a595a-315">修改從 .NET 到 JavaScript 的 interop 呼叫回應。</span><span class="sxs-lookup"><span data-stu-id="a595a-315">Modify the response of interop calls from .NET to JavaScript.</span></span>
* <span data-ttu-id="a595a-316">避免將 .NET 分派到 JS interop 結果。</span><span class="sxs-lookup"><span data-stu-id="a595a-316">Avoid dispatching .NET to JS interop results.</span></span>

<span data-ttu-id="a595a-317">Blazor 伺服器架構會採取下列步驟來防範先前的威脅：</span><span class="sxs-lookup"><span data-stu-id="a595a-317">The Blazor Server framework takes steps to protect against some of the preceding threats:</span></span>

* <span data-ttu-id="a595a-318">如果用戶端未確認轉譯批次，則停止產生新的 UI 更新。</span><span class="sxs-lookup"><span data-stu-id="a595a-318">Stops producing new UI updates if the client isn't acknowledging render batches.</span></span> <span data-ttu-id="a595a-319">已使用`CircuitOptions.MaxBufferedUnacknowledgedRenderBatches`設定。</span><span class="sxs-lookup"><span data-stu-id="a595a-319">Configured with `CircuitOptions.MaxBufferedUnacknowledgedRenderBatches`.</span></span>
* <span data-ttu-id="a595a-320">在一分鐘之後，任何 .NET 到 JavaScript 的呼叫都不會收到來自用戶端的回應。</span><span class="sxs-lookup"><span data-stu-id="a595a-320">Times out any .NET to JavaScript call after one minute without receiving a response from the client.</span></span> <span data-ttu-id="a595a-321">已使用`CircuitOptions.JSInteropDefaultCallTimeout`設定。</span><span class="sxs-lookup"><span data-stu-id="a595a-321">Configured with `CircuitOptions.JSInteropDefaultCallTimeout`.</span></span>
* <span data-ttu-id="a595a-322">在 JS interop 期間，對來自瀏覽器的所有輸入執行基本驗證：</span><span class="sxs-lookup"><span data-stu-id="a595a-322">Performs basic validation on all input coming from the browser during JS interop:</span></span>
  * <span data-ttu-id="a595a-323">.NET 參考是有效的，而且是 .NET 方法所預期的類型。</span><span class="sxs-lookup"><span data-stu-id="a595a-323">.NET references are valid and of the type expected by the .NET method.</span></span>
  * <span data-ttu-id="a595a-324">資料的格式不正確。</span><span class="sxs-lookup"><span data-stu-id="a595a-324">The data isn't malformed.</span></span>
  * <span data-ttu-id="a595a-325">此方法的引數數目正確，會出現在裝載中。</span><span class="sxs-lookup"><span data-stu-id="a595a-325">The correct number of arguments for the method are present in the payload.</span></span>
  * <span data-ttu-id="a595a-326">引數或結果可以在叫用方法之前正確還原序列化。</span><span class="sxs-lookup"><span data-stu-id="a595a-326">The arguments or result can be deserialized correctly before invoking the method.</span></span>
* <span data-ttu-id="a595a-327">在來自瀏覽器的所有輸入中，從分派的事件執行基本驗證：</span><span class="sxs-lookup"><span data-stu-id="a595a-327">Performs basic validation in all input coming from the browser from dispatched events:</span></span>
  * <span data-ttu-id="a595a-328">事件具有有效的類型。</span><span class="sxs-lookup"><span data-stu-id="a595a-328">The event has a valid type.</span></span>
  * <span data-ttu-id="a595a-329">事件的資料可以還原序列化。</span><span class="sxs-lookup"><span data-stu-id="a595a-329">The data for the event can be deserialized.</span></span>
  * <span data-ttu-id="a595a-330">有一個事件處理常式與事件相關聯。</span><span class="sxs-lookup"><span data-stu-id="a595a-330">There's an event handler associated with the event.</span></span>

<span data-ttu-id="a595a-331">除了架構所實行的保護措施以外，應用程式必須由開發人員撰寫程式碼，以防範威脅並採取適當的動作：</span><span class="sxs-lookup"><span data-stu-id="a595a-331">In addition to the safeguards that the framework implements, the app must be coded by the developer to safeguard against threats and take appropriate actions:</span></span>

* <span data-ttu-id="a595a-332">在處理事件時，一律驗證資料。</span><span class="sxs-lookup"><span data-stu-id="a595a-332">Always validate data when handling events.</span></span>
* <span data-ttu-id="a595a-333">接收無效資料時採取適當的動作：</span><span class="sxs-lookup"><span data-stu-id="a595a-333">Take appropriate action upon receiving invalid data:</span></span>
  * <span data-ttu-id="a595a-334">忽略資料並返回。</span><span class="sxs-lookup"><span data-stu-id="a595a-334">Ignore the data and return.</span></span> <span data-ttu-id="a595a-335">這可讓應用程式繼續處理要求。</span><span class="sxs-lookup"><span data-stu-id="a595a-335">This allows the app to continue processing requests.</span></span>
  * <span data-ttu-id="a595a-336">如果應用程式判斷出輸入是非法的，而且無法由合法的用戶端產生，則會擲回例外狀況。</span><span class="sxs-lookup"><span data-stu-id="a595a-336">If the app determines that the input is illegitimate and couldn't be produced by legitimate client, throw an exception.</span></span> <span data-ttu-id="a595a-337">擲回眼淚的例外狀況，並結束會話。</span><span class="sxs-lookup"><span data-stu-id="a595a-337">Throwing an exception tears down the circuit and ends the session.</span></span>
* <span data-ttu-id="a595a-338">請勿信任記錄檔中所包含的轉譯批次完成所提供的錯誤訊息。</span><span class="sxs-lookup"><span data-stu-id="a595a-338">Don't trust the error message provided by render batch completions included in the logs.</span></span> <span data-ttu-id="a595a-339">此錯誤是由用戶端所提供，而且通常無法受到信任，因為用戶端可能會遭到入侵。</span><span class="sxs-lookup"><span data-stu-id="a595a-339">The error is provided by the client and can't generally be trusted, as the client might be compromised.</span></span>
* <span data-ttu-id="a595a-340">請勿信任 JavaScript 和 .NET 方法之間任一方向的 JS interop 呼叫上的輸入。</span><span class="sxs-lookup"><span data-stu-id="a595a-340">Don't trust the input on JS interop calls in either direction between JavaScript and .NET methods.</span></span>
* <span data-ttu-id="a595a-341">應用程式會負責驗證引數和結果的內容是否有效，即使引數或結果已正確還原序列化也一樣。</span><span class="sxs-lookup"><span data-stu-id="a595a-341">The app is responsible for validating that the content of arguments and results are valid, even if the arguments or results are correctly deserialized.</span></span>

<span data-ttu-id="a595a-342">若要讓 XSS 弱點存在，應用程式必須在呈現的頁面中納入使用者輸入。</span><span class="sxs-lookup"><span data-stu-id="a595a-342">For a XSS vulnerability to exist, the app must incorporate user input in the rendered page.</span></span> <span data-ttu-id="a595a-343">Blazor 伺服器元件會執行編譯時期步驟，其中*razor*檔案中的標記會轉換成程式C#邏輯。</span><span class="sxs-lookup"><span data-stu-id="a595a-343">Blazor Server components execute a compile-time step where the markup in a *.razor* file is transformed into procedural C# logic.</span></span> <span data-ttu-id="a595a-344">在執行時間， C#邏輯會建立描述元素、文字和子元件的轉譯*樹狀結構*。</span><span class="sxs-lookup"><span data-stu-id="a595a-344">At runtime, the C# logic builds a *render tree* describing the elements, text, and child components.</span></span> <span data-ttu-id="a595a-345">這會透過一系列的 JavaScript 指示套用至瀏覽器的 DOM （或在進行預建的情況下序列化為 HTML）：</span><span class="sxs-lookup"><span data-stu-id="a595a-345">This is applied to the browser's DOM via a sequence of JavaScript instructions (or is serialized to HTML in the case of prerendering):</span></span>

* <span data-ttu-id="a595a-346">透過一般 Razor 語法呈現的使用者輸入（例如， `@someStringValue`）不會公開 XSS 弱點，因為 Razor 語法是透過只能寫入文字的命令新增至 DOM。</span><span class="sxs-lookup"><span data-stu-id="a595a-346">User input rendered via normal Razor syntax (for example, `@someStringValue`) doesn't expose a XSS vulnerability because the Razor syntax is added to the DOM via commands that can only write text.</span></span> <span data-ttu-id="a595a-347">即使值包含 HTML 標籤，值也會顯示為靜態文字。</span><span class="sxs-lookup"><span data-stu-id="a595a-347">Even if the value includes HTML markup, the value is displayed as static text.</span></span> <span data-ttu-id="a595a-348">預先呈現時，輸出會以 HTML 編碼，這也會將內容顯示為靜態文字。</span><span class="sxs-lookup"><span data-stu-id="a595a-348">When prerendering, the output is HTML-encoded, which also displays the content as static text.</span></span>
* <span data-ttu-id="a595a-349">不允許腳本標記，且不應包含在應用程式的元件轉譯樹狀結構中。</span><span class="sxs-lookup"><span data-stu-id="a595a-349">Script tags aren't allowed and shouldn't be included in the app's component render tree.</span></span> <span data-ttu-id="a595a-350">如果腳本標記包含在元件的標記中，就會產生編譯時期錯誤。</span><span class="sxs-lookup"><span data-stu-id="a595a-350">If a script tag is included in a component's markup, a compile-time error is generated.</span></span>
* <span data-ttu-id="a595a-351">元件作者可以在中撰寫C#元件，而不需使用 Razor。</span><span class="sxs-lookup"><span data-stu-id="a595a-351">Component authors can author components in C# without using Razor.</span></span> <span data-ttu-id="a595a-352">元件作者負責在發出輸出時使用正確的 Api。</span><span class="sxs-lookup"><span data-stu-id="a595a-352">The component author is responsible for using the correct APIs when emitting output.</span></span> <span data-ttu-id="a595a-353">例如，使用`builder.AddContent(0, someUserSuppliedString)` ，而*不* `builder.AddMarkupContent(0, someUserSuppliedString)`是，後者可能會建立 XSS 弱點。</span><span class="sxs-lookup"><span data-stu-id="a595a-353">For example, use `builder.AddContent(0, someUserSuppliedString)` and *not* `builder.AddMarkupContent(0, someUserSuppliedString)`, as the latter could create a XSS vulnerability.</span></span>

<span data-ttu-id="a595a-354">在保護 XSS 攻擊的過程中，請考慮執行 XSS 緩和措施，例如[內容安全性原則（CSP）](https://developer.mozilla.org/docs/Web/HTTP/CSP)。</span><span class="sxs-lookup"><span data-stu-id="a595a-354">As part of protecting against XSS attacks, consider implementing XSS mitigations, such as [Content Security Policy (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP).</span></span>

<span data-ttu-id="a595a-355">如需詳細資訊，請參閱<xref:security/cross-site-scripting>。</span><span class="sxs-lookup"><span data-stu-id="a595a-355">For more information, see <xref:security/cross-site-scripting>.</span></span>

### <a name="cross-origin-protection"></a><span data-ttu-id="a595a-356">跨原始來源保護</span><span class="sxs-lookup"><span data-stu-id="a595a-356">Cross-origin protection</span></span>

<span data-ttu-id="a595a-357">跨原始來源攻擊牽涉到來自不同來源的用戶端對伺服器執行動作。</span><span class="sxs-lookup"><span data-stu-id="a595a-357">Cross-origin attacks involve a client from a different origin performing an action against the server.</span></span> <span data-ttu-id="a595a-358">惡意動作通常是 GET 要求或表單 POST （跨網站偽造要求，CSRF），但也可能開啟惡意的 WebSocket。</span><span class="sxs-lookup"><span data-stu-id="a595a-358">The malicious action is typically a GET request or a form POST (Cross-Site Request Forgery, CSRF), but opening a malicious WebSocket is also possible.</span></span> <span data-ttu-id="a595a-359">Blazor 伺服器應用程式[會提供使用中樞通訊協定供應專案的其他任何 SignalR 應用程式的相同保證](xref:signalr/security)：</span><span class="sxs-lookup"><span data-stu-id="a595a-359">Blazor Server apps offer [the same guarantees that any other SignalR app using the hub protocol offer](xref:signalr/security):</span></span>

* <span data-ttu-id="a595a-360">Blazor 伺服器應用程式可以跨來源存取，除非採取額外的措施來防止它。</span><span class="sxs-lookup"><span data-stu-id="a595a-360">Blazor Server apps can be accessed cross-origin unless additional measures are taken to prevent it.</span></span> <span data-ttu-id="a595a-361">若要停用跨原始來源存取，請在端點中停用 cors，方法是將 cors 中介軟體新增`DisableCorsAttribute`至管線，並將新增至 Blazor 端點中繼資料，或藉由設定[跨原始來源資源的 SignalR 來限制允許的原始來源集共用](xref:signalr/security#cross-origin-resource-sharing)。</span><span class="sxs-lookup"><span data-stu-id="a595a-361">To disable cross-origin access, either disable CORS in the endpoint by adding the CORS middleware to the pipeline and adding the `DisableCorsAttribute` to the Blazor endpoint metadata or limit the set of allowed origins by [configuring SignalR for cross-origin resource sharing](xref:signalr/security#cross-origin-resource-sharing).</span></span>
* <span data-ttu-id="a595a-362">如果已啟用 CORS，則可能需要額外的步驟來保護應用程式，視 CORS 設定而定。</span><span class="sxs-lookup"><span data-stu-id="a595a-362">If CORS is enabled, extra steps might be required to protect the app depending on the CORS configuration.</span></span> <span data-ttu-id="a595a-363">如果已全域啟用 cors，則可以停用 Blazor 伺服器中樞的 cors，方法是`DisableCorsAttribute`在呼叫`hub.MapBlazorHub()`之後，將中繼資料新增至端點中繼資料。</span><span class="sxs-lookup"><span data-stu-id="a595a-363">If CORS is globally enabled, CORS can be disabled for the Blazor Server hub by adding the `DisableCorsAttribute` metadata to the endpoint metadata after calling `hub.MapBlazorHub()`.</span></span>

<span data-ttu-id="a595a-364">如需詳細資訊，請參閱<xref:security/anti-request-forgery>。</span><span class="sxs-lookup"><span data-stu-id="a595a-364">For more information, see <xref:security/anti-request-forgery>.</span></span>

### <a name="click-jacking"></a><span data-ttu-id="a595a-365">按一下-劫持</span><span class="sxs-lookup"><span data-stu-id="a595a-365">Click-jacking</span></span>

<span data-ttu-id="a595a-366">按一下劫持牽涉到從不同的來源將`<iframe>`網站轉譯為網站內部，以誘騙使用者在遭受攻擊的網站上執行動作。</span><span class="sxs-lookup"><span data-stu-id="a595a-366">Click-jacking involves rendering a site as an `<iframe>` inside a site from a different origin in order to trick the user into performing actions on the site under attack.</span></span>

<span data-ttu-id="a595a-367">若要保護應用程式不`<iframe>`在內呈現，請使用[內容安全性原則（CSP](https://developer.mozilla.org/docs/Web/HTTP/CSP) `X-Frame-Options` ）和標頭。</span><span class="sxs-lookup"><span data-stu-id="a595a-367">To protect an app from rendering inside of an `<iframe>`, use [Content Security Policy (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP) and the `X-Frame-Options` header.</span></span> <span data-ttu-id="a595a-368">如需詳細資訊， [請參閱 MDN web 檔：X 框架-選項](https://developer.mozilla.org/docs/Web/HTTP/Headers/X-Frame-Options)。</span><span class="sxs-lookup"><span data-stu-id="a595a-368">For more information, see [MDN web docs: X-Frame-Options](https://developer.mozilla.org/docs/Web/HTTP/Headers/X-Frame-Options).</span></span>

### <a name="open-redirects"></a><span data-ttu-id="a595a-369">開啟重新導向</span><span class="sxs-lookup"><span data-stu-id="a595a-369">Open redirects</span></span>

<span data-ttu-id="a595a-370">當 Blazor 伺服器應用程式會話啟動時，伺服器會針對啟動會話時所傳送的 Url 執行基本驗證。</span><span class="sxs-lookup"><span data-stu-id="a595a-370">When a Blazor Server app session starts, the server performs basic validation of the URLs sent as part of starting the session.</span></span> <span data-ttu-id="a595a-371">在建立線路之前，架構會檢查基底 URL 是否為目前 URL 的父系。</span><span class="sxs-lookup"><span data-stu-id="a595a-371">The framework checks that the base URL is a parent of the current URL before establishing the circuit.</span></span> <span data-ttu-id="a595a-372">架構不會執行其他檢查。</span><span class="sxs-lookup"><span data-stu-id="a595a-372">No additional checks are performed by the framework.</span></span>

<span data-ttu-id="a595a-373">當使用者選取用戶端上的連結時，會將連結的 URL 傳送至伺服器，以決定要採取的動作。</span><span class="sxs-lookup"><span data-stu-id="a595a-373">When a user selects a link on the client, the URL for the link is sent to the server, which determines what action to take.</span></span> <span data-ttu-id="a595a-374">例如，應用程式可能會執行用戶端導覽，或向瀏覽器表示前往新的位置。</span><span class="sxs-lookup"><span data-stu-id="a595a-374">For example, the app may perform a client-side navigation or indicate to the browser to go to the new location.</span></span>

<span data-ttu-id="a595a-375">元件也可以透過使用`NavigationManager`，以程式設計方式觸發導覽要求。</span><span class="sxs-lookup"><span data-stu-id="a595a-375">Components can also trigger navigation requests programatically through the use of `NavigationManager`.</span></span> <span data-ttu-id="a595a-376">在這種情況下，應用程式可能會執行用戶端導覽，或向瀏覽器表示前往新的位置。</span><span class="sxs-lookup"><span data-stu-id="a595a-376">In such scenarios, the app might perform a client-side navigation or indicate to the browser to go to the new location.</span></span>

<span data-ttu-id="a595a-377">元件必須：</span><span class="sxs-lookup"><span data-stu-id="a595a-377">Components must:</span></span>

* <span data-ttu-id="a595a-378">避免在導覽呼叫引數中使用使用者輸入。</span><span class="sxs-lookup"><span data-stu-id="a595a-378">Avoid using user input as part of the navigation call arguments.</span></span>
* <span data-ttu-id="a595a-379">驗證引數，以確保應用程式允許該目標。</span><span class="sxs-lookup"><span data-stu-id="a595a-379">Validate arguments to ensure that the target is allowed by the app.</span></span>

<span data-ttu-id="a595a-380">否則，惡意使用者可以強制瀏覽器移至攻擊者控制的網站。</span><span class="sxs-lookup"><span data-stu-id="a595a-380">Otherwise, a malicious user can force the browser to go to an attacker-controlled site.</span></span> <span data-ttu-id="a595a-381">在此案例中，攻擊者會將應用程式訣竅為在叫用`NavigationManager.Navigate`方法時，使用部分使用者輸入。</span><span class="sxs-lookup"><span data-stu-id="a595a-381">In this scenario, the attacker tricks the app into using some user input as part of the invocation of the `NavigationManager.Navigate` method.</span></span>

<span data-ttu-id="a595a-382">這項建議也適用于將連結轉譯為應用程式的一部分時：</span><span class="sxs-lookup"><span data-stu-id="a595a-382">This advice also applies when rendering links as part of the app:</span></span>

* <span data-ttu-id="a595a-383">可能的話，請使用相對連結。</span><span class="sxs-lookup"><span data-stu-id="a595a-383">If possible, use relative links.</span></span>
* <span data-ttu-id="a595a-384">先驗證絕對連結目的地是否有效，再將它們包含在頁面中。</span><span class="sxs-lookup"><span data-stu-id="a595a-384">Validate that absolute link destinations are valid before including them in a page.</span></span>

<span data-ttu-id="a595a-385">如需詳細資訊，請參閱<xref:security/preventing-open-redirects>。</span><span class="sxs-lookup"><span data-stu-id="a595a-385">For more information, see <xref:security/preventing-open-redirects>.</span></span>

## <a name="authentication-and-authorization"></a><span data-ttu-id="a595a-386">驗證與授權</span><span class="sxs-lookup"><span data-stu-id="a595a-386">Authentication and authorization</span></span>

<span data-ttu-id="a595a-387">如需驗證和授權的指引， <xref:security/blazor/index>請參閱。</span><span class="sxs-lookup"><span data-stu-id="a595a-387">For guidance on authentication and authorization, see <xref:security/blazor/index>.</span></span>

## <a name="security-checklist"></a><span data-ttu-id="a595a-388">安全性檢查清單</span><span class="sxs-lookup"><span data-stu-id="a595a-388">Security checklist</span></span>

<span data-ttu-id="a595a-389">下列安全性考慮清單並不完整：</span><span class="sxs-lookup"><span data-stu-id="a595a-389">The following list of security considerations isn't comprehensive:</span></span>

* <span data-ttu-id="a595a-390">驗證來自事件的引數。</span><span class="sxs-lookup"><span data-stu-id="a595a-390">Validate arguments from events.</span></span>
* <span data-ttu-id="a595a-391">驗證 JS interop 呼叫的輸入和結果。</span><span class="sxs-lookup"><span data-stu-id="a595a-391">Validate inputs and results from JS interop calls.</span></span>
* <span data-ttu-id="a595a-392">避免使用（或預先驗證） .NET 到 JS interop 呼叫的使用者輸入。</span><span class="sxs-lookup"><span data-stu-id="a595a-392">Avoid using (or validate beforehand) user input for .NET to JS interop calls.</span></span>
* <span data-ttu-id="a595a-393">防止用戶端配置未系結的記憶體數量。</span><span class="sxs-lookup"><span data-stu-id="a595a-393">Prevent the client from allocating an unbound amount of memory.</span></span>
  * <span data-ttu-id="a595a-394">元件中的資料。</span><span class="sxs-lookup"><span data-stu-id="a595a-394">Data within the component.</span></span>
  * <span data-ttu-id="a595a-395">`DotNetObject`傳回給用戶端的參考。</span><span class="sxs-lookup"><span data-stu-id="a595a-395">`DotNetObject` references returned to the client.</span></span>
* <span data-ttu-id="a595a-396">針對多個分派進行防護。</span><span class="sxs-lookup"><span data-stu-id="a595a-396">Guard against multiple dispatches.</span></span>
* <span data-ttu-id="a595a-397">處置元件時，取消長時間執行的作業。</span><span class="sxs-lookup"><span data-stu-id="a595a-397">Cancel long-running operations when the component is disposed.</span></span>
* <span data-ttu-id="a595a-398">避免產生大量資料的事件。</span><span class="sxs-lookup"><span data-stu-id="a595a-398">Avoid events that produce large amounts of data.</span></span>
* <span data-ttu-id="a595a-399">如果無法`NavigationManager.Navigate`避免，請避免在的呼叫中使用使用者輸入，並在一組允許的原始來源驗證使用者輸入 url。</span><span class="sxs-lookup"><span data-stu-id="a595a-399">Avoid using user input as part of calls to `NavigationManager.Navigate` and validate user input for URLs against a set of allowed origins first if unavoidable.</span></span>
* <span data-ttu-id="a595a-400">請勿根據 UI 狀態（但僅從元件狀態）進行授權決策。</span><span class="sxs-lookup"><span data-stu-id="a595a-400">Don't make authorization decisions based on the state of the UI but only from component state.</span></span>
* <span data-ttu-id="a595a-401">請考慮使用[內容安全性原則（CSP）](https://developer.mozilla.org/docs/Web/HTTP/CSP)來防範 XSS 攻擊。</span><span class="sxs-lookup"><span data-stu-id="a595a-401">Consider using [Content Security Policy (CSP)](https://developer.mozilla.org/docs/Web/HTTP/CSP) to protect against XSS attacks.</span></span>
* <span data-ttu-id="a595a-402">請考慮使用 CSP 和[X 框架選項](https://developer.mozilla.org/docs/Web/HTTP/Headers/X-Frame-Options)來防止按一下劫持。</span><span class="sxs-lookup"><span data-stu-id="a595a-402">Consider using CSP and [X-Frame-Options](https://developer.mozilla.org/docs/Web/HTTP/Headers/X-Frame-Options) to protect against click-jacking.</span></span>
* <span data-ttu-id="a595a-403">請確定 CORS 設定適用于啟用 CORS，或明確停用 Blazor 應用程式的 CORS。</span><span class="sxs-lookup"><span data-stu-id="a595a-403">Ensure CORS settings are appropriate when enabling CORS or explicitly disable CORS for Blazor apps.</span></span>
* <span data-ttu-id="a595a-404">測試以確保 Blazor 應用程式的伺服器端限制提供可接受的使用者體驗，而不會有無法接受的風險層級。</span><span class="sxs-lookup"><span data-stu-id="a595a-404">Test to ensure that the server-side limits for the Blazor app provide an acceptable user experience without unacceptable levels of risk.</span></span>