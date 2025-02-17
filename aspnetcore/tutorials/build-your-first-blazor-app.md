---
title: 組建第一個 Blazor 應用程式
author: guardrex
description: 逐步建置 Blazor 應用程式。
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/15/2019
uid: tutorials/first-blazor-app
ms.openlocfilehash: c357b324905ee3a4c9f4bd167dbbcacaf7e1bc76
ms.sourcegitcommit: 35a86ce48041caaf6396b1e88b0472578ba24483
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2019
ms.locfileid: "72391213"
---
# <a name="build-your-first-blazor-app"></a>組建第一個 Blazor 應用程式

作者：[Daniel Roth](https://github.com/danroth27) 和 [Luke Latham](https://github.com/guardrex)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

本教學課程示範如何建置和修改 Blazor 應用程式。

請依照 <xref:blazor/get-started> 文章中的指引來建立本教學課程的 Blazor 專案。 將專案命名為 *ToDoList*。

## <a name="build-components"></a>組建元件

1. 流覽至每個應用程式的 [ *pages* ] 資料夾中的三個頁面： [首頁]、[計數器] 和 [提取資料]。 這些頁面會透過 Razor 元件檔案 *Index.razor*、*Counter.razor* 及 *FetchData.razor* 來實作。

1. 在 [計數器] 頁面上，選取 [按我] 按鈕以在不重新整理頁面的情況下讓計數器遞增。 讓網頁中的計數器遞增通常需要撰寫 JavaScript，但 Blazor 提供更好的 C# 使用方法。

1. 檢查 *Counter.razor`Counter` 檔案中*  元件的實作。

   *Pages/Counter.razor*：

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/Counter1.razor)]

   `Counter` 元件的 UI 是使用 HTML 來定義的。 動態轉譯邏輯 (例如迴圈、條件、運算式) 是使用內嵌的 C# 語法 (稱為 [Razor](xref:mvc/views/razor)) 來新增的。 HTML 標記和 C# 轉譯邏輯會在組建時轉換為元件類別。 所產生 .NET 類別的名稱會與檔案名稱相符。

   元件類別的成員均定義於 `@code` 區塊中。 在 `@code` 區塊中，會指定元件狀態 (屬性、欄位) 來處理事件或定義其他元件邏輯。 然後會使用這些成員作為元件轉譯邏輯的一部分，並用於處理事件。

   選取 [Click me] \(按我\) 按鈕時：

   * 會呼叫 `Counter` 元件的已註冊 `onclick` 處理常式 (`IncrementCount` 方法)。
   * `Counter` 元件會重新產生其轉譯樹狀結構。
   * 新的轉譯樹狀結構會與先前的樹狀結構做比較。
   * 只會套用對「文件物件模型」(DOM) 所做的修改。 顯示的計數即會更新。

1. 修改 `Counter` 元件的 C# 邏輯，讓計數遞增 2 而不是 1。

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/Counter2.razor?highlight=14)]

1. 重建並執行應用程式以查看變更。 選取 [按我] 按鈕。 計數器會遞增二。

## <a name="use-components"></a>使用元件

使用 HTML 語法來將一個元件包含在另一個元件中。

1. 透過將 `<Counter />` 元素新增至 `Index` 元件 (*Index.razor*)，來將 `Counter` 元件新增至應用程式的 `Index` 元件。

   如果您使用 Blazor WebAssembly 來進行此體驗，則 `Index` 元件會使用 @no__t 0 元件。 使用 `<Counter />` 元素取代 `<SurveyPrompt>` 元素。 如果您使用 Blazor 伺服器應用程式進行此體驗，請將 `<Counter />` 元素新增至 `Index` 元件：

   *Pages/Index.razor*：

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/Index1.razor?highlight=7)]

1. 重建並執行應用程式。 `Index` 元件有自己的計數器。

## <a name="component-parameters"></a>元件參數

元件也可以有參數。 元件參數是在元件類別上使用具有 `[Parameter]` 屬性的公用屬性來定義。 使用這些屬性來指定標記中元件的引數。

1. 更新元件的 `@code` C# 程式碼：

   * 新增具有 `[Parameter]` 屬性的公用 `IncrementAmount` 屬性。
   * 將 `IncrementCount` 方法變更為在增加 `currentCount`的值時使用 `IncrementAmount`。

   *Pages/Counter.razor*：

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/Counter.razor?highlight=13,17)]

<!-- Add back when supported.
   > [!NOTE]
   > From Visual Studio, you can quickly add a component parameter by using the `para` snippet. Type `para` and press the `Tab` key twice.
-->

1. 使用屬性在 `Index` 元件的 `<Counter>` 元素中指定 `IncrementAmount` 參數。 設定值來讓計數器以 10 遞增。

   *Pages/Index.razor*：

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/Index2.razor?highlight=7)]

1. 重新載入 `Index` 元件。 每次選取 [Click me] \(按我\) 按鈕時，計數器都會以 10 遞增。 `Counter` 元件中的計數器會繼續遞增一。

## <a name="route-to-components"></a>路由到元件

*Counter.razor* 檔案頂端的 `@page` 指示詞會指定 `Counter` 元件是路由端點。 `Counter` 元件會處理傳送給 `/counter` 的要求。 若沒有 `@page` 指示詞，元件就不會處理路由傳送的要求，但其他元件仍可使用此元件。

## <a name="dependency-injection"></a>相依性插入

### <a name="blazor-server-experience"></a>Blazor 伺服器體驗

如果使用 Blazor 伺服器應用程式，則 `WeatherForecastService` 服務會在 `Startup.ConfigureServices` 中註冊為[singleton](xref:fundamentals/dependency-injection#service-lifetimes) 。 服務的實例可透過相依性[插入（DI）](xref:fundamentals/dependency-injection)在整個應用程式中使用：

[!code-csharp[](build-your-first-blazor-app/samples_snapshot/3.x/Startup.cs?highlight=5)]

@No__t-0 指示詞是用來將 @no__t 1 服務的實例插入至 @no__t 2 元件。

*Pages/FetchData.razor*：

[!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/FetchData1.razor?highlight=3)]

`FetchData` 元件會使用插入的服務作為 `ForecastService`，以擷取 `WeatherForecast` 物件的陣列：

[!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/FetchData2.razor?highlight=6)]

### <a name="blazor-webassembly-experience"></a>Blazor WebAssembly 體驗

如果使用 Blazor WebAssembly 應用程式，則會插入 `HttpClient`，以從*wwwroot/sample-data*資料夾中的*氣象*檔案取得氣象預報資料。

*Pages/FetchData.razor*：

[!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/FetchData1_client.razor?highlight=7-8)]

[@No__t 1](/dotnet/csharp/language-reference/keywords/foreach-in)迴圈是用來將每個預測實例轉譯為天氣資料的資料表中的資料列：

[!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/FetchData3.razor?highlight=11-19)]

## <a name="build-a-todo-list"></a>組建待辦事項清單

在實作簡單待辦事項清單的應用程式中新增元件。

1. 將名為 *Todo.razor* 的空白檔案新增至 *Pages* 資料夾中的應用程式：

1. 提供元件的初始標記：

   ```cshtml
   @page "/todo"

   <h1>Todo</h1>
   ```

1. 將 `Todo` 元件新增至導覽列。

   `NavMenu` 元件 (*Shared/NavMenu.razor*) 會用於應用程式的版面配置。 版面配置是可讓您避免應用程式中內容重複的元件。

   透過在 *Shared/NavMenu.razor* 檔案中的現有清單項目下方新增下列清單項目標記，為 `Todo` 元件新增 `<NavLink>` 元素：

   ```cshtml
   <li class="nav-item px-3">
       <NavLink class="nav-link" href="todo">
           <span class="oi oi-list-rich" aria-hidden="true"></span> Todo
       </NavLink>
   </li>
   ```

1. 重建並執行應用程式。 瀏覽新的 [待辦事項] 頁面，以確認 `Todo` 元件的連結可以運作。

1. 將 *TodoItem.cs* 檔案新增至專案的根目錄，以保存代表待辦事項的類別。 請使用下列 `TodoItem` 類別的 C# 程式碼：

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/TodoItem.cs)]

1. 返回 `Todo` 元件 (*Pages/Todo.razor*)：

   * 在 `@code` 區塊中新增待辦事項的欄位。 `Todo` 元件會使用此欄位來維護待辦事項清單的狀態。
   * 新增未排序的清單標記和 `foreach` 迴圈，將每個待辦事項轉譯為清單項目 (`<li>`)。

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/ToDo4.razor?highlight=5-10,12-14)]

1. 應用程式需要 UI 元素，才能將待辦事項新增至清單。 在未排序清單 (`<ul>...</ul>`) 下方新增文字輸出 (`<input>`) 與按鈕 (`<button>`)：

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/ToDo5.razor?highlight=12-13)]

1. 重建並執行應用程式。 選取 [Add todo] \(新增待辦事項\) 按鈕時，不會發生任何情況，因為事件處理常式並未連接至這個按鈕。

1. 將 `AddTodo` 方法新增至 `Todo` 元件並註冊，以便使用 `@onclick` 屬性來進行按鈕選取。 當選取按鈕時，就會呼叫 `AddTodo` C# 方法：

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/ToDo6.razor?highlight=2,7-10)]

1. 若要取得新待辦事項的標題，請在 `@code` 區塊頂端新增 `newTodo` 字串欄位，然後使用 `<input>` 元素中的 `bind` 屬性將它繫結至文字輸入的值：

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/ToDo7.razor?highlight=2)]

   ```cshtml
   <input placeholder="Something todo" @bind="newTodo" />
   ```

1. 更新 `AddTodo` 方法，將 `TodoItem` 與指定的標題新增至清單。 請將 `newTodo` 設定為空字串，以清除文字輸入的值：

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/ToDo8.razor?highlight=19-26)]

1. 重建並執行應用程式。 請將一些待辦事項新增至待辦事項清單，以測試新程式碼。

1. 每個待辦事項的標題文字都可設定為可編輯，而核取方塊則可協助使用者記錄已完成的項目。 請為每個待辦事項新增核取方塊輸入，然後將其值繫結至 `IsDone` 屬性。 將 `@todo.Title` 變更為繫結至 `@todo.Title` 的 `<input>` 元素：

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/ToDo9.razor?highlight=5-6)]

1. 若要確認是否已繫結這些值，請更新 `<h1>` 標頭，以顯示未完成之待辦事項 (`IsDone` 為 `false`) 的數目計數。

   ```cshtml
   <h1>Todo (@todos.Count(todo => !todo.IsDone))</h1>
   ```

1. 已完成的 `Todo` 元件 (*Pages/Todo.razor*)：

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/Todo.razor)]

1. 重建並執行應用程式。 請新增待辦事項，以測試新程式碼。

> [!div class="nextstepaction"]
> <xref:blazor/components>
