---
title: ASP.NET Core 基本概念
author: rick-anderson
description: 了解建置 ASP.NET Core 應用程式的基本概念。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/07/2019
uid: fundamentals/index
ms.openlocfilehash: a70d6aa05a2c92d19076b8d6e4ea24d7554368b6
ms.sourcegitcommit: 3d082bd46e9e00a3297ea0314582b1ed2abfa830
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/07/2019
ms.locfileid: "72007112"
---
# <a name="aspnet-core-fundamentals"></a>ASP.NET Core 基本概念

本文是了解如何開發 ASP.NET Core 應用程式的關鍵主題概觀。

## <a name="the-startup-class"></a>Startup 類別

`Startup` 類別是：

* 已設定應用程式所需的服務。
* 定義要求處理管線的位置。

「服務」是應用程式所使用的元件。 例如，記錄元件即為一項服務。 設定 (或「註冊」) 服務的程式碼會新增至 `Startup.ConfigureServices` 方法。

要求處理管線由一系列*中介軟體*元件所組成。 例如，中介軟體可能會處理靜態檔案的要求，或是將 HTTP 要求重新導向至 HTTPS。 每個中介軟體會在 `HttpContext` 上執行非同步操作，然後叫用管線中下一個中介軟體或終止要求。 設定要求處理管線的程式碼會新增至 `Startup.Configure` 方法。

以下是 `Startup` 類別範例：

[!code-csharp[](index/snapshots/2.x/Startup1.cs?highlight=3,12)]

如需詳細資訊，請參閱<xref:fundamentals/startup>。

## <a name="dependency-injection-services"></a>相依性插入 (服務)

ASP.NET Core 具有內建的相依性插入 (DI) 架構，可讓應用程式的類別使用所設定服務。 取得類別中一項服務執行個體的其中一種方式，便是使用所需類型的參數來建立建構函式。 參數可以是服務類型或是介面。 DI 系統會在執行階段提供服務。

以下是使用 DI 取得 Entity Framework Core 內容物件的類別。 醒目提示的那一行便是建構函式插入的範例：

[!code-csharp[](index/snapshots/2.x/Index.cshtml.cs?highlight=5)]

雖然 DI 為內建，但其設計用於讓您插入協力廠商的控制反轉 (IoC) 容器 (若您想要的話)。

如需詳細資訊，請參閱<xref:fundamentals/dependency-injection>。

## <a name="middleware"></a>中介軟體

要求處理管線是以一系列中介軟體元件組成。 每個元件會在 `HttpContext` 上執行非同步操作，然後叫用管線中下一個中介軟體或終止要求。

依照慣例，中介軟體元件會透過叫用其 `Startup.Configure` 方法中的 `Use...` 延伸模組方法來新增至管線。 例如，若要啟用靜態檔案轉譯，請呼叫 `UseStaticFiles`。

下列範例中醒目提示的程式碼會設定要求處理管線：

[!code-csharp[](index/snapshots/2.x/Startup1.cs?highlight=14-16)]

ASP.NET Core 包含一組豐富的內建中介軟體，您也可以撰寫自訂中介軟體。

如需詳細資訊，請參閱<xref:fundamentals/middleware/index>。

## <a name="host"></a>主機

ASP.NET Core 應用程式會在啟動時建置一個「主機」。 主機是封裝所有應用程式資源的物件，例如：

* HTTP 伺服器實作
* 中介軟體元件
* 記錄
* DI
* 組態

在單一物件中包含所有應用程式相互依存資源的主要理由便是生命週期管理：控制應用程式的啟動及順利關機。

::: moniker range=">= aspnetcore-3.0"

可以使用兩種主機：一般主機與 Web 主機。 建議您使用一般主機，Web 主機只適用於回溯相容性。

建立主機的程式碼位於 `Program.Main` 中：

[!code-csharp[](index/snapshots/3.x/Program1.cs)]

`CreateDefaultBuilder` 和 `ConfigureWebHostDefaults` 方法會設定具備一般常用選項的主機，如下所示：

* 使用 [Kestrel](#servers) 作為網頁伺服器，並啟用 IIS 整合。
* 從 *appsettings.json*、*appsettings.{Environment Name}.json*、環境變數與命令列引數載入設定。
* 將記錄輸出傳送到主控台及偵錯提供者。

如需詳細資訊，請參閱<xref:fundamentals/host/generic-host>。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

可以使用兩種主機：Web 主機與一般主機。 在 ASP.NET Core 2.x 中，一般主機僅適用於非 Web 案例。

建立主機的程式碼位於 `Program.Main` 中：

[!code-csharp[](index/snapshots/2.x/Program1.cs)]

`CreateDefaultBuilder` 方法會設定具備一般常用選項的主機，如下所示：

* 使用 [Kestrel](#servers) 作為網頁伺服器，並啟用 IIS 整合。
* 從 *appsettings.json*、*appsettings.{Environment Name}.json*、環境變數與命令列引數載入設定。
* 將記錄輸出傳送到主控台及偵錯提供者。

如需詳細資訊，請參閱<xref:fundamentals/host/web-host>。

::: moniker-end

### <a name="non-web-scenarios"></a>非 Web 案例

一般主機允許其他類型的應用程式，使用交叉剪輯架構延伸模組，例如記錄、相依性插入 (DI)、設定與應用程式存留期管理。 如需詳細資訊，請參閱 <xref:fundamentals/host/generic-host> 與 <xref:fundamentals/host/hosted-services>。

## <a name="servers"></a>伺服器

ASP.NET Core 應用程式使用 HTTP 伺服器實作來接聽 HTTP 要求。 伺服器會把向應用程式發出的要求作為一組[要求功能](xref:fundamentals/request-features)，合併成一個 `HttpContext`。

::: moniker range=">= aspnetcore-2.2"

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

ASP.NET Core 隨附下列伺服器實作：

* *Kestrel* 是跨平台的網頁伺服器。 Kestrel 通常會使用 [IIS](https://www.iis.net/)在反向 Proxy 設定中執行。 在 ASP.NET Core 2.0 或更新版本中，Kestrel 可以作為直接向網際網路公開的公眾 Edge Server 執行。
* *IIS HTTP 伺服器*則是適用於使用 IIS Windows 的伺服器。 透過此伺服器，ASP.NET Core 應用程式及 IIS 便可以在相同處理序中執行。
* *HTTP.sys* 則是適用於並未搭配 IIS 使用 Windows 的伺服器。

# <a name="macostabmacos"></a>[macOS](#tab/macos)

ASP.NET Core 提供 *Kestrel* 跨平台伺服器實作。 在 ASP.NET Core 2.0 或更新版本中，Kestrel 可以作為直接向網際網路公開的公眾 Edge Server 執行。 Kestrel 通常會使用 [Nginx](https://nginx.org) 或 [Apache](https://httpd.apache.org/)在反向 Proxy 設定中執行。

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

ASP.NET Core 提供 *Kestrel* 跨平台伺服器實作。 在 ASP.NET Core 2.0 或更新版本中，Kestrel 可以作為直接向網際網路公開的公眾 Edge Server 執行。 Kestrel 通常會使用 [Nginx](https://nginx.org) 或 [Apache](https://httpd.apache.org/)在反向 Proxy 設定中執行。

---

::: moniker-end

::: moniker range="< aspnetcore-2.2"

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

ASP.NET Core 隨附下列伺服器實作：

* *Kestrel* 是跨平台的網頁伺服器。 Kestrel 通常會使用 [IIS](https://www.iis.net/)在反向 Proxy 設定中執行。 在 ASP.NET Core 2.0 或更新版本中，Kestrel 可以作為直接向網際網路公開的公眾 Edge Server 執行。
* *HTTP.sys* 則是適用於並未搭配 IIS 使用 Windows 的伺服器。

# <a name="macostabmacos"></a>[macOS](#tab/macos)

ASP.NET Core 提供 *Kestrel* 跨平台伺服器實作。 在 ASP.NET Core 2.0 或更新版本中，Kestrel 可以作為直接向網際網路公開的公眾 Edge Server 執行。 Kestrel 通常會使用 [Nginx](https://nginx.org) 或 [Apache](https://httpd.apache.org/)在反向 Proxy 設定中執行。

# <a name="linuxtablinux"></a>[Linux](#tab/linux)

ASP.NET Core 提供 *Kestrel* 跨平台伺服器實作。 在 ASP.NET Core 2.0 或更新版本中，Kestrel 可以作為直接向網際網路公開的公眾 Edge Server 執行。 Kestrel 通常會使用 [Nginx](https://nginx.org) 或 [Apache](https://httpd.apache.org/)在反向 Proxy 設定中執行。

---

::: moniker-end

如需詳細資訊，請參閱<xref:fundamentals/servers/index>。

## <a name="configuration"></a>組態

ASP.NET Core 提供組態架構，可從組態提供者的已排序集合中，以成對名稱和數值的形式取得設定。 您可以使用各種來源的內建組態提供者，例如 *.json* 檔案、 *.xml* 檔案、環境變數及命令列引數。 您也可以撰寫自訂組態提供者。

例如，您可以指定組態來自 *appsettings.json* 和環境變數。 然後當要求 *ConnectionString* 的值時，架構便會先在 *appsettings.json* 檔案中尋找。 若有找到值，但在環境變數中也存在該值，則會優先使用來自環境變數的值。

針對管理保密組態資料 (例如密碼)，ASP.NET Core 提供[祕密管理員工具](xref:security/app-secrets)。 針對生產祕密，我們建議使用 [Azure Key Vault](xref:security/key-vault-configuration)。

如需詳細資訊，請參閱<xref:fundamentals/configuration/index>。

## <a name="options"></a>選項。

在可能的情況下，ASP.NET Core 會遵循「選項模式」來儲存及擷取組態值。 選項模式使用類別來代表一組相關的設定。

例如，下列程式碼會設定 WebSocket 選項：

```csharp
var options = new WebSocketOptions  
{  
   KeepAliveInterval = TimeSpan.FromSeconds(120),  
   ReceiveBufferSize = 4096
};  
app.UseWebSockets(options);
```

如需詳細資訊，請參閱<xref:fundamentals/configuration/options>。

## <a name="environments"></a>環境

執行環境 (例如「開發」、「預備」及「生產」) 是 ASP.NET Core 中的第一級概念。 您可以透過設定 `ASPNETCORE_ENVIRONMENT` 環境變數，指定應用程式執行的環境。 ASP.NET Core 會在應用程式啟動時讀取環境變數，然後將值儲存在 `IHostingEnvironment` 實作中。 環境物件可在應用程式中的任何位置，透過 DI 使用。

下列 `Startup` 類別的範例程式碼會設定應用程式，只在於「開發」環境中執行時提供詳細的錯誤資訊：

[!code-csharp[](index/snapshots/2.x/Startup2.cs?highlight=3-6)]

如需詳細資訊，請參閱<xref:fundamentals/environments>。

## <a name="logging"></a>記錄

ASP.NET Core 支援記錄 API，此 API 能與各種內建和第三方記錄提供者搭配使用。 可用的提供者包括如下：

* 主控台
* 偵錯
* Windows 上的事件追蹤
* Windows 事件記錄檔
* TraceSource
* Azure App Service
* Azure Application Insights

透過從 DI 取得 `ILogger` 物件並呼叫記錄方法，從應用程式程式碼中的任何位置寫入記錄。

以下是使用 `ILogger` 物件的範例程式碼，其中建構函式插入及記錄方法呼叫都已醒目提示。

[!code-csharp[](index/snapshots/2.x/TodoController.cs?highlight=5,13,17)]

`ILogger` 介面可讓您將任何數量的欄位傳遞給記錄提供者。 欄位常用於建構訊息字串，但提供者也可以將它們作為個別欄位，傳送至資料存放區。 這項功能可讓記錄提供者實作 [semantic logging (語意記錄)，又稱為 structured logging (結構化記錄)](https://softwareengineering.stackexchange.com/questions/312197/benefits-of-structured-logging-vs-basic-logging)。

如需詳細資訊，請參閱<xref:fundamentals/logging/index>。

## <a name="routing"></a>路由

「路由」是一種對應到處理常式的 URL 模式。 處理常式通常是 Razor 頁面、MVC 控制器中的動作方法，或是中介軟體。 ASP.NET Core 路由可讓您控制您應用程式使用的 URL。

如需詳細資訊，請參閱<xref:fundamentals/routing>。

## <a name="error-handling"></a>錯誤處理

ASP.NET Core 具有處理錯誤的內建功能，例如：

* 開發人員例外狀況頁面
* 自訂錯誤頁面
* 靜態狀態碼頁面
* 啟動例外狀況處理

如需詳細資訊，請參閱<xref:fundamentals/error-handling>。

## <a name="make-http-requests"></a>發出 HTTP 要求

`IHttpClientFactory` 的實作可用於建立 `HttpClient` 執行個體。 Factory：

* 提供一個集中位置以便命名和設定邏輯 `HttpClient` 執行個體。 例如，您可以註冊 *github* 並設定用戶端以存取 GitHub。 預設用戶端可以註冊用於其他用途。
* 支援註冊及多個委派處理常式的鏈結，以用於建置傳出要求中介軟體管線。 此模式與 ASP.NET Core 中的輸入中介軟體管線相似。 模式提供一個機制來管理 HTTP 要求的跨領域關注，包括快取、錯誤處理、序列化和記錄。
* 與 *Polly* 整合，Polly 是一種熱門的協力廠商程式庫，用於進行暫時性的錯誤處理。
* 管理基礎 `HttpClientMessageHandler` 執行個體的共用和存留期，以避免在手動管理 `HttpClient` 存留期時，發生的常見 DNS 問題。
* 針對透過處理站所建立之用戶端傳送的所有要求，新增可設定的記錄體驗 (透過 `ILogger`)。

如需詳細資訊，請參閱<xref:fundamentals/http-requests>。

## <a name="content-root"></a>內容根目錄

內容根目錄是的基底路徑：

* 裝載應用程式的可執行檔（ *.exe*）。
* 組成應用程式的已編譯元件（ *.dll*）。
* 應用程式所使用的非程式碼內容檔案，例如：
  * Razor 檔案（ *. cshtml*， *razor*）
  * 設定檔（ *. json*、 *.xml*）
  * 資料檔案（ *.db*）
* [Web 根目錄](#web-root)，通常是已發佈的*wwwroot*資料夾。

在開發期間：

* 內容根目錄預設為專案的根目錄。
* 專案的根目錄是用來建立：
  * 應用程式在專案根目錄中的非程式碼內容檔案路徑。
  * [Web 根目錄](#web-root)，通常是專案根目錄中的*wwwroot*資料夾。

::: moniker range=">= aspnetcore-3.0"

[建立主機](#host)時，可以指定替代的內容根路徑。 如需詳細資訊，請參閱<xref:fundamentals/host/generic-host#contentrootpath>。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[建立主機](#host)時，可以指定替代的內容根路徑。 如需詳細資訊，請參閱<xref:fundamentals/host/web-host#content-root>。

::: moniker-end

## <a name="web-root"></a>Web 根目錄

Web 根目錄是公用、非程式碼、靜態資源檔的基底路徑，例如：

* 樣式表單（ *.css*）
* JavaScript （ *.js*）
* 影像（ *.png*、 *.jpg*）

根據預設，靜態檔案只會從 web 根目錄（和子目錄）提供。

::: moniker range=">= aspnetcore-3.0"

Web 根目錄路徑預設為 *{content root}/wwwroot*，但在[建立主機](#host)時，可以指定不同的 web 根目錄。 如需詳細資訊，請參閱<xref:fundamentals/host/generic-host#webroot>。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Web 根目錄路徑預設為 *{content root}/wwwroot*，但在[建立主機](#host)時，可以指定不同的 web 根目錄。 如需詳細資訊，請參閱 [Web 根目錄](xref:fundamentals/host/web-host#web-root)。

::: moniker-end

在 Razor （*cshtml*）檔案中，波狀符號-斜線（`~/`）會指向 web 根目錄。 開頭為 `~/` 的路徑稱為*虛擬路徑*。

如需詳細資訊，請參閱<xref:fundamentals/static-files>。
