---
title: 將驗證新增至 ASP.NET Core Razor 頁面
author: rick-anderson
description: 了解如何將驗證新增至 ASP.NET Core 中的 Razor 頁面。
ms.author: riande
ms.custom: mvc
ms.date: 7/23/2019
uid: tutorials/razor-pages/validation
ms.openlocfilehash: 5c5419eb6ccfbd9ddd8d6fadb24d688966d76c10
ms.sourcegitcommit: 476ea5ad86a680b7b017c6f32098acd3414c0f6c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/14/2019
ms.locfileid: "69022396"
---
# <a name="add-validation-to-an-aspnet-core-razor-page"></a>將驗證新增至 ASP.NET Core Razor 頁面

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

在本節中將驗證邏輯新增至 `Movie` 模型。 使用者建立或編輯電影時，就會強制執行驗證規則。

## <a name="validation"></a>驗證

軟體開發的核心原則稱為 [DRY](https://wikipedia.org/wiki/Don%27t_repeat_yourself)("**D**on't **R**epeat **Y**ourself", 不重複原則)。 Razor Pages可促進開發，只要指定功能一次，就能在整個應用程式中運用。 DRY 有助於：

* 降低應用程式中的程式碼數量。
* 使程式碼較少出現錯誤，而且更容易進行測試和維護。

Razor Pages 和 Entity Framework 所提供的驗證支援就是 DRY 準則的絶佳範例。 驗證規則是在單一位置 (在模型類別中) 以宣告方式指定，而規則可在應用程式的任何位置強制執行。

## <a name="add-validation-rules-to-the-movie-model"></a>將驗證規則新增至電影模型

DataAnnotations 命名空間提供一組內建的驗證屬性 (attribute)，其以宣告方式套用至類別或屬性 (property)。 DataAnnotations 也包含格式化屬性 (如 `DataType`)，可協助進行格式化，但不提供任何驗證。

更新 `Movie` 類別，以充分利用內建的 `Required`、`StringLength`、`RegularExpression` 和 `Range` 驗證屬性。

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Models/MovieDateRatingDA.cs?name=snippet1)]

驗證屬性會指定您想要對套用目標模型屬性強制執行的行為：

* `Required` 和 `MinimumLength` 屬性 (attribute) 指出屬性 (property) 必須是值；但無法防止使用者輸入空格以滿足此驗證。
* `RegularExpression` 屬性則用來限制可輸入的字元。 在上述程式碼中，"Genre"：

  * 必須指使用字母。
  * 第一個字母必須是大寫。 不允許使用空格、數字和特殊字元。

* `RegularExpression` "Rating"：

  * 第一個字元必須為大寫字母。
  * 允許後續空格中的特殊字元和數位。 "PG-13" 對分級而言有效，但不適用於 "Genre"。

* `Range` 屬性會將值限制在指定的範圍內。
* `StringLength` 屬性可讓您設定字串屬性的最大長度，並選擇性設定其最小長度。
* 實值型別 (如`decimal`、`int`、`float`、`DateTime`) 原本就是必要項目，而且不需要 `[Required]` 屬性。

擁有 ASP.NET Core 自動強制執行的驗證規則有助於讓您的應用程式更穩固。 它也確保您不會忘記要驗證某些項目，不小心讓不正確的資料進入資料庫。

### <a name="validation-error-ui-in-razor-pages"></a>Razor Pages 中的驗證錯誤 UI

執行應用程式，並巡覽至 Pages/Movies。

選取 **Create New** 連結。 使用某些無效值完成表單。 當 jQuery 用戶端驗證偵測到錯誤時，它會顯示錯誤訊息。

![有多個 jQuery 用戶端驗證錯誤的電影檢視表單](validation/_static/val.png)

[!INCLUDE[](~/includes/currency.md)]

請注意表單在包含無效值的每個欄位中自動呈現驗證錯誤訊息的方式。 用戶端 (使用 JavaScript 和 jQuery) 與伺服器端 (當使用者已停用 JavaScript 時) 都會強制執行這些錯誤。

明顯的好處是：**不**需要在 Create 或 Edit 頁面中進行程式碼變更。 一旦 DataAnnotations 套用至模型，就會啟用驗證 UI。 本教學課程中建立的 Razor 頁面會自動拾取驗證規則 (在 `Movie` 模型類別的屬性 (property) 上使用驗證屬性 (attribute))。 使用 Edit 頁面測試驗證，會套用相同的驗證。

要一直到沒有任何用戶端驗證錯誤之後，才會將表單資料發佈到伺服器。 請確認表單資料不會經由下列一或多種方式發佈：

* 將中斷點放置在 `OnPostAsync` 方法中。 提交表單 (選取 [建立]  或 [儲存]  )。 永遠不會叫用中斷點。
* 使用 [Fiddler 工具](https://www.telerik.com/fiddler)。
* 使用瀏覽器開發人員工具來監視網路流量。

### <a name="server-side-validation"></a>伺服器端驗證

在瀏覽器中停用 JavaScript 時，提交含有錯誤的表單將發佈到伺服器。

選擇性地測試伺服器端驗證：

* 在瀏覽器中停用 JavaScript。 您可以使用瀏覽器的開發人員工具來停用 JavaScript。 如果您無法在該瀏覽器中停用 JavaScript，請嘗試另一個瀏覽器。
* 在 Create 或 Edit 頁面的 `OnPostAsync` 方法中設定中斷點。
* 提交含有無效資料的表單。
* 確認模型狀態無效：

  ```csharp
   if (!ModelState.IsValid)
   {
      return Page();
   }
  ```

下列程式碼顯示稍早在此教學課程中包含 Scaffold 的部分 *Create.cshtml* 頁面。 Create 和 Edit 頁面會使用它來顯示初始表單，以及在發生錯誤時重新顯示格式。

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Create.cshtml?range=14-20)]

[輸入標記協助程式](xref:mvc/views/working-with-forms)會使用 [DataAnnotations](/aspnet/mvc/overview/older-versions/mvc-music-store/mvc-music-store-part-6) 屬性，並產生在用戶端上進行 jQuery 驗證所需的 HTML 屬性。 [驗證標記協助程式](xref:mvc/views/working-with-forms#the-validation-tag-helpers)會顯示驗證錯誤。 如需詳細資訊，請參閱[驗證](xref:mvc/models/validation)。

Create 和 Edit 頁面中沒有任何驗證規則。 只有在 `Movie` 類別中才能指定驗證規則和錯誤字串。 這些驗證規則會自動套用至編輯 `Movie` 模型的 Razor 頁面。

當驗證邏輯需要變更時，它只會在模型中進行。 驗證會一致地套用在整個應用程式中(驗證邏輯定義於一個位置)。 位於一個位置的驗證有助於讓程式碼保持整潔，並可讓您更容易進行維護和更新。

## <a name="using-datatype-attributes"></a>使用 DataType 屬性

檢查 `Movie` 類別。 除了一組內建的驗證屬性之外，`System.ComponentModel.DataAnnotations` 命名空間還提供了格式屬性。 `DataType` 屬性 (attribute) 會套用至 `ReleaseDate` 和 `Price` 屬性 (property)。

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie/Models/MovieDateRatingDA.cs?highlight=2,6&name=snippet2)]

`DataType` 屬性只提供檢視引擎將資料格式化的提示 (以及提供一些屬性，例如用於 URL 的 `<a>` 和用於電子郵件的 `<a href="mailto:EmailAddress.com">`)。 使用 `RegularExpression` 屬性來驗證資料的格式。 `DataType` 屬性可用於指定比資料庫內建類型更特定的資料類型。 `DataType` 屬性不是驗證屬性。 在範例應用程式中，只會顯示日期，而不含時間。

`DataType` 列舉可提供給許多資料類型，例如 Date、Time、PhoneNumber、Currency、EmailAddress 等等。 `DataType` 屬性也可讓應用程式自動提供類型的特定功能。 例如，可針對 `DataType.EmailAddress` 建立 `mailto:` 連結。 在支援 HTML5 的瀏覽器中，可以為 `DataType.Date` 提供日期選取器。 `DataType` 屬性會發出 HTML 5 瀏覽器使用的 HTML 5 `data-` (唸成的 data dash) 屬性。 `DataType` 屬性**不**會提供任何驗證。

`DataType.Date` 未指定顯示日期的格式。 根據預設，將依據以伺服器 `CultureInfo` 為基礎的預設格式顯示資料欄位。

`[Column(TypeName = "decimal(18, 2)")]` 資料註解為必要項，因此 Entity Framework Core 可將 `Price` 正確對應到資料庫中的貨幣。 如需詳細資訊，請參閱[資料類型](/ef/core/modeling/relational/data-types)。

`DisplayFormat` 屬性用來明確指定日期格式：

```csharp
[DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
public DateTime ReleaseDate { get; set; }
```

`ApplyFormatInEditMode` 設定可指定當顯示值以供編輯時，應該套用的格式。 但在某些欄位中，您可能不想要套用該行為。 例如，在貨幣值中，應該不會希望編輯 UI 中出現貨幣符號。

`DisplayFormat` 屬性可以單獨使用，但通常最好是使用 `DataType` 屬性。 `DataType` 屬性會傳逹資料的語意，而不是在螢幕上呈現資料的方式，並提供下列使用 DisplayFormat 無法取得的優點：

* 瀏覽器可以啟用 HTML5 功能 (例如顯示日曆控制項、適合地區設定的貨幣符號、電子郵件連結等等)。
* 根據預設，瀏覽器將根據您的地區設定，使用正確的格式呈現資料。
* `DataType` 屬性可以啟用 ASP.NET Core 架構，來選擇用於呈現資料的正確欄位範本。 `DisplayFormat` 若是單獨使用，則會使用字串範本。

請注意：jQuery 驗證不適用於 `Range` 屬性與 `DateTime`。 例如，下列程式碼一律會顯示用戶端驗證錯誤，即使當日期位在指定範圍中也一樣：

```csharp
[Range(typeof(DateTime), "1/1/1966", "1/1/2020")]
   ```

編譯模型中的硬性日期通常不是良好的做法，因此不建議使用 `Range` 屬性和 `DateTime`。

下列程式碼會顯示一行上的結合屬性：

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/MovieDateRatingDAmult.cs?name=snippet1)]

[Razor Pages 和 EF Core 使用者入門](xref:data/ef-rp/intro)說明使用 Razor Pages 執行進階 EF Core 作業。

### <a name="apply-migrations"></a>套用移轉

套用至類別的 DataAnnotations 會變更結構描述。 例如，套用至 `Title` 欄位的 DataAnnotations：

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Models/MovieDateRatingDA.cs?name=snippet11)]

* 將字元限制為 60 個。
* 不允許 `null` 值。

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

`Movie` 資料表目前具有下列結構描述：

``` sql
CREATE TABLE [dbo].[Movie] (
    [ID]          INT             IDENTITY (1, 1) NOT NULL,
    [Title]       NVARCHAR (MAX)  NULL,
    [ReleaseDate] DATETIME2 (7)   NOT NULL,
    [Genre]       NVARCHAR (MAX)  NULL,
    [Price]       DECIMAL (18, 2) NOT NULL,
    [Rating]      NVARCHAR (MAX)  NULL,
    CONSTRAINT [PK_Movie] PRIMARY KEY CLUSTERED ([ID] ASC)
);
```

上述結構描述變更不會造成 EF 擲回例外狀況。 不過，請建立移轉，讓結構描述與模型一致。

從 [工具]  功能表中，選取 [NuGet 套件管理員] > [套件管理員主控台]  。
在 PMC 中，輸入下列命令：

```powershell
Add-Migration New_DataAnnotations
Update-Database
```

`Update-Database` 會執行 `New_DataAnnotations` 類別的 `Up` 方法。 檢查 `Up` 方法：

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie30/Migrations/20190724163003_New_DataAnnotations.cs?name=snippet)]

已更新的 `Movie` 資料表具有下列結構描述：

``` sql
CREATE TABLE [dbo].[Movie] (
    [ID]          INT             IDENTITY (1, 1) NOT NULL,
    [Title]       NVARCHAR (60)   NOT NULL,
    [ReleaseDate] DATETIME2 (7)   NOT NULL,
    [Genre]       NVARCHAR (30)   NOT NULL,
    [Price]       DECIMAL (18, 2) NOT NULL,
    [Rating]      NVARCHAR (5)    NOT NULL,
    CONSTRAINT [PK_Movie] PRIMARY KEY CLUSTERED ([ID] ASC)
);
```

# <a name="visual-studio-code--visual-studio-for-mactabvisual-studio-codevisual-studio-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

SQLite 不需要移轉。

---

### <a name="publish-to-azure"></a>發佈至 Azure

如需部署至 Azure 的資訊，請參閱[教學課程：使用 SQL Database 在 Azure 中建置 ASP.NET Core 應用程式](/azure/app-service/app-service-web-tutorial-dotnetcore-sqldb)。

感謝您看完這份 Razor Pages 簡介。 完成本教學課程之後，非常建議您繼續參閱 [Razor 頁面與 EF Core 使用者入門](xref:data/ef-rp/intro)。

## <a name="additional-resources"></a>其他資源

* <xref:mvc/views/working-with-forms>
* <xref:fundamentals/localization>
* <xref:mvc/views/tag-helpers/intro>
* <xref:mvc/views/tag-helpers/authoring>
* [這個教學課程的 YouTube 版本](https://youtu.be/b63m66eu7us)

> [!div class="step-by-step"]
> [上一步：新增欄位](xref:tutorials/razor-pages/new-field)
