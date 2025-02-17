---
title: 開始使用 ASP.NET Core Blazor
author: guardrex
description: 使用您選擇的工具來建立 Blazor 應用程式，以開始使用 Blazor。
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/15/2019
uid: blazor/get-started
ms.openlocfilehash: fc368be5eb2e5d8f7c80071dc86a02ae986a685f
ms.sourcegitcommit: 35a86ce48041caaf6396b1e88b0472578ba24483
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2019
ms.locfileid: "72391046"
---
# <a name="get-started-with-aspnet-core-blazor"></a>開始使用 ASP.NET Core Blazor

作者：[Daniel Roth](https://github.com/danroth27) 和 [Luke Latham](https://github.com/guardrex)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

開始使用 Blazor：

::: moniker range=">= aspnetcore-3.1"

1. 安裝[.Net Core 3.1 PREVIEW SDK](https://dotnet.microsoft.com/download/dotnet-core/3.1)。

1. 在命令 shell 中執行下列命令，以安裝[Blazor WebAssembly](xref:blazor/hosting-models#blazor-webassembly)範本。 Blazor WebAssembly 處於預覽階段時， [AspNetCore. Blazor](https://www.nuget.org/packages/Microsoft.AspNetCore.Blazor.Templates/)套件具有預覽版本。

   ```dotnetcli
   dotnet new -i Microsoft.AspNetCore.Blazor.Templates::3.1.0-preview1.19508.20
   ```

1. 遵循您選擇的工具的指導方針：

   # <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

   1 \。 使用**ASP.NET 和 網頁程式開發**工作負載安裝[Visual Studio 16.4 Preview 2 或更新版本](https://visualstudio.microsoft.com/vs/preview/)。

   2 \。 建立新的專案。

   3 \。 選取 [ **Blazor 應用程式**]。 選取 [下一步]。

   4 \。 在 [專案名稱] 欄位中提供專案名稱，或接受預設專案名稱。 確認 [**位置**] 專案正確，或提供專案的 [位置]。 選取 [建立]。

   5 \。 如需 Blazor 的 WebAssembly 體驗，請選擇 [ **Blazor WebAssembly 應用程式**] 範本。 如需 Blazor 伺服器體驗，請選擇 [ **Blazor 伺服器應用程式**] 範本。 選取 [建立]。 如需這兩個 Blazor 裝載模型、 *Blazor 伺服器*和*Blazor WebAssembly*的相關資訊，請參閱 <xref:blazor/hosting-models>。

   6。 按下 **F5** 即可執行應用程式。

   > [!NOTE]
   > 如果您已安裝 ASP.NET Core Blazor （Preview 6 或更早版本）先前預覽版本的 Blazor Visual Studio 延伸模組，則可以卸載擴充功能。 在命令介面中安裝 Blazor 範本，現在已足以在 Visual Studio 中呈現範本。

   # <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

   1 \。 安裝 [Visual Studio Code (英文)](https://code.visualstudio.com/)。

   2 \。 安裝[ C# Visual Studio Code 擴充](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp)功能的最新版本。

   3 \。 如需 Blazor 的 WebAssembly 體驗，請在命令 shell 中執行下列命令：

      ```dotnetcli
      dotnet new blazorwasm -o WebApplication1
      ```

      如需 Blazor 伺服器體驗，請在命令 shell 中執行下列命令：

      ```dotnetcli
      dotnet new blazorserver -o WebApplication1
      ```

      如需這兩個 Blazor 裝載模型、 *Blazor 伺服器*和*Blazor WebAssembly*的相關資訊，請參閱 <xref:blazor/hosting-models>。

   4 \。 在 Visual Studio Code 中開啟 [ *WebApplication1* ] 資料夾。

   5 \。 若為 Blazor 伺服器專案，IDE 會要求您新增資產以建立和對專案進行偵錯工具。 選取 [是]。

   6。 如果使用 Blazor 伺服器應用程式，請使用 Visual Studio Code 偵錯工具來執行應用程式。 如果使用 Blazor WebAssembly 應用程式，請從應用程式的專案資料夾執行 `dotnet run`。

   7 \。 在瀏覽器中，巡覽至 `https://localhost:5001`。

   <!--

   # [Visual Studio for Mac](#tab/visual-studio-mac)

   1\. Install [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/). Switch the [Update channel to Preview](/visualstudio/mac/install-preview).

   2\. Select **File** > **New Solution** or **New Project**.

   3\. In the sidebar, select **.NET Core** > **App**.

   4\. For a Blazor Server experience, select the **Blazor Server App** template. For a Blazor WebAssembly experience, select the **Blazor WebAssembly App** template. Select **Next**. For information on the two Blazor hosting models, *Blazor Server* and *Blazor WebAssembly*, see <xref:blazor/hosting-models>.

   5\. The **Target Framework** defaults to **.NET Core 3.0**. Select **Next**.

   6\. In the **Project Name** field, enter `WebApplication1`. Select **Create**.

   7\. Select **Run** > **Run Without Debugging** to run the app *without the debugger*. Running with the debugger isn't supported at this time.

   -->

   # <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli/)

   如需 Blazor 的 WebAssembly 體驗，請在命令 shell 中執行下列命令：

   ```dotnetcli
   dotnet new blazorwasm -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

   如需 Blazor 伺服器體驗，請在命令 shell 中執行下列命令：

   ```dotnetcli
   dotnet new blazorserver -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

   如需這兩個 Blazor 裝載模型、 *Blazor 伺服器*和*Blazor WebAssembly*的相關資訊，請參閱 <xref:blazor/hosting-models>。

   在瀏覽器中，巡覽至 `https://localhost:5001`。

   ---

::: moniker-end

::: moniker range="< aspnetcore-3.1"

1. 安裝最新的[.Net Core 3.0 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.0)版本。

1. 安裝[.Net Core 3.1 PREVIEW SDK](https://dotnet.microsoft.com/download/dotnet-core/3.1) ，然後在命令 shell 中執行下列命令，選擇性地安裝[Blazor WebAssembly](xref:blazor/hosting-models#blazor-webassembly)範本：

   ```dotnetcli
   dotnet new -i Microsoft.AspNetCore.Blazor.Templates::3.1.0-preview1.19508.20
   ```

1. 遵循您選擇的工具的指導方針：

   # <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

   1 \。 使用**ASP.NET 和 網頁程式開發**工作負載安裝最新的[Visual Studio](https://visualstudio.com/vs/) 。

   2 \。 選擇性地使用適用于 Blazor WebAssembly 應用程式開發的**ASP.NET 和 網頁程式開發**工作負載，安裝[Visual Studio 16.4 Preview 2 或更新版本](https://visualstudio.microsoft.com/vs/preview/)。

   3 \。 建立新的專案。

   4 \。 選取 [ **Blazor 應用程式**]。 選取 [下一步]。

   5 \。 在 [專案名稱] 欄位中提供專案名稱，或接受預設專案名稱。 確認 [**位置**] 專案正確，或提供專案的 [位置]。 選取 [建立]。

   6。 如需 Blazor 的 WebAssembly 體驗，請選擇 [ **Blazor WebAssembly 應用程式**] 範本。 如需 Blazor 伺服器體驗，請選擇 [ **Blazor 伺服器應用程式**] 範本。 選取 [建立]。 如需這兩個 Blazor 裝載模型、 *Blazor 伺服器*和*Blazor WebAssembly*的相關資訊，請參閱 <xref:blazor/hosting-models>。

   7 \。 按下 **F5** 即可執行應用程式。

   > [!NOTE]
   > 如果您已安裝 ASP.NET Core Blazor （Preview 6 或更早版本）先前預覽版本的 Blazor Visual Studio 延伸模組，則可以卸載擴充功能。 在命令介面中安裝 Blazor 範本，現在已足以在 Visual Studio 中呈現範本。

   # <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

   1 \。 安裝 [Visual Studio Code (英文)](https://code.visualstudio.com/)。

   2 \。 安裝[ C# Visual Studio Code 擴充](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp)功能的最新版本。

   3 \。 如需 Blazor 的 WebAssembly 體驗，請在命令 shell 中執行下列命令：

      ```dotnetcli
      dotnet new blazorwasm -o WebApplication1
      ```

      如需 Blazor 伺服器體驗，請在命令 shell 中執行下列命令：

      ```dotnetcli
      dotnet new blazorserver -o WebApplication1
      ```

      如需這兩個 Blazor 裝載模型、 *Blazor 伺服器*和*Blazor WebAssembly*的相關資訊，請參閱 <xref:blazor/hosting-models>。

   4 \。 在 Visual Studio Code 中開啟 [ *WebApplication1* ] 資料夾。

   5 \。 若為 Blazor 伺服器專案，IDE 會要求您新增資產以建立和對專案進行偵錯工具。 選取 [是]。

   6。 如果使用 Blazor 伺服器應用程式，請使用 Visual Studio Code 偵錯工具來執行應用程式。 如果使用 Blazor WebAssembly 應用程式，請從應用程式的專案資料夾執行 `dotnet run`。

   7 \。 在瀏覽器中，巡覽至 `https://localhost:5001`。

   <!--

   # [Visual Studio for Mac](#tab/visual-studio-mac)

   1\. Install [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/). Switch the [Update channel to Preview](/visualstudio/mac/install-preview).

   2\. Select **File** > **New Solution** or **New Project**.

   3\. In the sidebar, select **.NET Core** > **App**.

   4\. For a Blazor Server experience, select the **Blazor Server App** template. For a Blazor WebAssembly experience, select the **Blazor WebAssembly App** template. Select **Next**. For information on the two Blazor hosting models, *Blazor Server* and *Blazor WebAssembly*, see <xref:blazor/hosting-models>.

   5\. The **Target Framework** defaults to **.NET Core 3.0**. Select **Next**.

   6\. In the **Project Name** field, enter `WebApplication1`. Select **Create**.

   7\. Select **Run** > **Run Without Debugging** to run the app *without the debugger*. Running with the debugger isn't supported at this time.

   -->

   # <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli/)

   如需 Blazor 的 WebAssembly 體驗，請在命令 shell 中執行下列命令：

   ```dotnetcli
   dotnet new blazorwasm -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

   如需 Blazor 伺服器體驗，請在命令 shell 中執行下列命令：

   ```dotnetcli
   dotnet new blazorserver -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

   如需這兩個 Blazor 裝載模型、 *Blazor 伺服器*和*Blazor WebAssembly*的相關資訊，請參閱 <xref:blazor/hosting-models>。

   在瀏覽器中，巡覽至 `https://localhost:5001`。

   ---

::: moniker-end

提要欄位中的索引標籤可使用多個頁面：

* 首頁
* 計數器
* 提取資料

在 [計數器] 頁面上，選取 [按我] 按鈕以在不重新整理頁面的情況下讓計數器遞增。 將網頁中的計數器遞增通常需要撰寫 JavaScript，但透過 Blazor，您可以C#使用。

*Pages/Counter.razor*：

[!code-cshtml[](get-started/samples_snapshot/3.x/Counter1.razor?highlight=7,12-15)]

在瀏覽器中 `/counter` 的要求，如同頂端的 `@page` 指示詞所指定，會導致 @no__t 2 元件轉譯其內容。 元件會轉譯成轉譯樹狀結構的記憶體中標記法，然後用來以彈性且有效率的方式更新 UI。

每次選取 [**按我**] 按鈕時：

* @No__t-0 事件會引發。
* 已呼叫 `IncrementCount` 方法。
* @No__t-0 會遞增。
* 元件會再次轉譯。

執行時間會比較新的內容與先前的內容，而且只會將已變更的內容套用至檔物件模型（DOM）。

使用 HTML 語法將元件新增至另一個元件。 例如，將 `Counter` 元件新增至應用程式的首頁，方法是將 `<Counter />` 元素新增至 @no__t 2 元件。

*Pages/Index.razor*：

[!code-cshtml[](get-started/samples_snapshot/3.x/Index1.razor?highlight=7)]

執行應用程式。 首頁有自己的計數器，由 @no__t 0 元件提供。

元件參數是使用屬性或[子內容](xref:blazor/components#child-content)所指定，可讓您設定子元件上的屬性。 若要將參數新增至 `Counter` 元件，請更新元件的 `@code` 區塊：

* 使用 `[Parameter]` 屬性來新增 `IncrementAmount` 的公用屬性。
* 將 `IncrementCount` 方法變更為在增加 `currentCount`的值時使用 `IncrementAmount`。

*Pages/Counter.razor*：

[!code-cshtml[](get-started/samples_snapshot/3.x/Counter2.razor?highlight=12-13,17)]

使用屬性，在 `Index` 元件的 `<Counter>` 元素中指定 `IncrementAmount`。

*Pages/Index.razor*：

[!code-cshtml[](get-started/samples_snapshot/3.x/Index2.razor?highlight=7)]

執行應用程式。 @No__t 0 元件有自己的計數器，每次選取 [按**我**] 按鈕時，就會遞增10。 @No__t-2 的 `Counter` 元件（*razor*）會繼續遞增一。

## <a name="next-steps"></a>後續步驟

<xref:tutorials/first-blazor-app>

## <a name="additional-resources"></a>其他資源

* <xref:signalr/introduction>
