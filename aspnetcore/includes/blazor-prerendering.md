當 Blazor 伺服器應用程式進行預先處理時，某些動作（例如呼叫 JavaScript）並不可能，因為尚未建立與瀏覽器的連接。 元件可能需要在資源清單時以不同的方式呈現。

若要延遲 JavaScript interop 呼叫，直到建立與瀏覽器的連線之後，您可以使用 `OnAfterRenderAsync` 元件生命週期事件。 只有在完整呈現應用程式並建立用戶端連線之後，才會呼叫此事件。

```cshtml
@using Microsoft.JSInterop
@inject IJSRuntime JSRuntime

<input @ref="myInput" value="Value set during render" />

@code {
    private ElementReference myInput;

    protected override void OnAfterRender(bool firstRender)
    {
        if (firstRender)
        {
            JSRuntime.InvokeVoidAsync(
                "setElementValue", myInput, "Value set after render");
        }
    }
}
```

下列元件示範如何使用 JavaScript interop 做為元件初始化邏輯的一部分，使其與可處理性相容。 此元件顯示可以從 `OnAfterRenderAsync` 內觸發轉譯更新。 開發人員必須避免在此案例中建立無限迴圈。

呼叫 `JSRuntime.InvokeAsync` 的位置時，`ElementRef` 只會用於 `OnAfterRenderAsync` 中，而不會用於任何較早的生命週期方法，因為在轉譯元件之前，沒有 JavaScript 元素。

呼叫 `StateHasChanged`，以從 JavaScript interop 呼叫取得新狀態來 rerender 元件。 程式碼不會建立無限迴圈，因為只有在 `infoFromJs` `null` 時，才會呼叫 `StateHasChanged`。

```cshtml
@page "/prerendered-interop"
@using Microsoft.AspNetCore.Components
@using Microsoft.JSInterop
@inject IJSRuntime JSRuntime

<p>
    Get value via JS interop call:
    <strong id="val-get-by-interop">@(infoFromJs ?? "No value yet")</strong>
</p>

<p>
    Set value via JS interop call:
    <input id="val-set-by-interop" @ref="myElem" />
</p>

@code {
    private string infoFromJs;
    private ElementReference myElem;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender && infoFromJs == null)
        {
            infoFromJs = await JSRuntime.InvokeAsync<string>(
                "setElementValue", myElem, "Hello from interop call");

            StateHasChanged();
        }
    }
}
```
