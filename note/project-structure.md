# DotNet6API 專案結構與設定檔說明

這份文件說明了 `DotNet6API` 專案中的目錄結構以及主要設定檔的用途，幫助您快速了解整個專案的架構。

## 目錄結構概覽

整個解決方案的結構大致如下：

```
DotNet6API/
├── .vs/                          # Visual Studio 或其他 IDE 產生的暫存/設定目錄 (自動產生)
├── .git/                         # Git 版本控制目錄
├── .gitignore                    # Git 忽略清單，定義哪些檔案不要進入版本控制
├── DotNet6API.sln                # 方案檔 (Solution File)，包含專案的參考與架構資訊
├── global.json                   # 定義 .NET SDK 的版本設定
├── README.md                     # 專案說明文件 (中文)
├── README-en.md                  # 專案說明文件 (英文)
├── note/                         # 存放筆記與說明文件的目錄 (包含本文件)
└── DotNet6API/                   # 實際的專案程式碼目錄 (Main Project)
    ├── Controllers/              # 存放 Web API 控制器 (Controllers)
    ├── Properties/               # 存放專案屬性設定，如 launchSettings.json
    ├── bin/                      # 編譯後產生的執行檔與 DLL 檔存放目錄
    ├── obj/                      # 編譯過程中的暫存檔存放目錄
    ├── DotNet6API.csproj         # 專案檔 (Project File)
    ├── Program.cs                # 應用程式的進入點與中介軟體 (Middleware) 設定
    ├── appsettings.json          # 應用程式的全域設定檔
    ├── appsettings.Development.json # 開發環境專用的設定檔
    └── WeatherForecast.cs        # 範例資料模型 (Model)
```

---

## 主要設定檔說明

### 1. `DotNet6API.csproj` (專案設定檔)
這是 .NET 專案的核心設定檔，採用 XML 格式。
- 定義了專案使用的目標框架 (Target Framework)，例如 `<TargetFramework>net6.0</TargetFramework>` 表示使用 .NET 6。
- 啟用了可為 Null 參考型別 (`<Nullable>enable</Nullable>`) 以及隱含的 `using` 指示詞 (`<ImplicitUsings>enable</ImplicitUsings>`)，這是 .NET 6 的新特性。
- 包含了專案相依的 NuGet 套件，例如 `Swashbuckle.AspNetCore` (用於 Swagger API 文件)。

### 2. `Program.cs` (應用程式進入點)
在 .NET 6 中，捨棄了過去 `Startup.cs` 的設計，改採頂級語句 (Top-level statements)，將應用程式的註冊與設定集中在 `Program.cs`。
- **服務註冊 (Services)**：例如 `builder.Services.AddControllers()` 註冊控制器服務，`builder.Services.AddSwaggerGen()` 註冊 Swagger 文件產生器。
- **中介軟體 (Middleware) 管線設定**：定義了 HTTP 請求的處理流程。例如 `app.UseHttpsRedirection()` (強制導向 HTTPS)、`app.UseAuthorization()` (授權中介軟體) 以及 `app.MapControllers()` (路由映射)。
- **開發環境設定**：使用 `app.Environment.IsDevelopment()` 來判斷是否為開發環境，若是則啟用 Swagger UI 供開發者測試 API。

### 3. `appsettings.json` 與 `appsettings.Development.json` (應用程式設定檔)
這是用來存放應用程式組態設定的 JSON 檔案（例如資料庫連線字串、API 金鑰等）。
- **`appsettings.json`**：全域的預設設定。裡面預設配置了系統的日誌紀錄等級 (Logging) 以及允許的主機 (`AllowedHosts`)。
- **`appsettings.Development.json`**：開發環境專用的設定。當系統在開發環境執行時，這裡的設定會覆蓋 `appsettings.json` 中同名的設定，方便開發者使用不同的組態。

### 4. `Properties/launchSettings.json` (啟動設定檔)
- 用於定義當我們在本地端開發與測試時，應用程式該如何啟動。
- 包含了應用程式的啟動 URL (如 `http://localhost:5000`)、環境變數 (`ASPNETCORE_ENVIRONMENT` 設為 `Development`) 以及是否啟動瀏覽器等設定。此檔案僅在本地開發時有效，不會發布到生產環境中。

### 5. `DotNet6API.sln` (方案檔)
- .NET 的方案檔，用於將一個或多個專案組織在一起。當您用 Visual Studio 或是 Rider 開啟專案時，通常會直接開啟這個 `.sln` 檔案，IDE 就會知道該載入哪些專案以及它們之間的依賴關係。

### 6. `global.json`
- 用於鎖定專案使用的 .NET SDK 版本。確保所有開發者以及 CI/CD 流程中都使用相同版本的 .NET SDK 來編譯與執行，避免版本不一致導致的問題。
