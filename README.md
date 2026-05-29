# DotNet6API

這是一個用於學習目的的 .NET 6 API 專案。本專案主要是透過 Mac 版本的 Visual Studio 2022 所建立。

## 啟動專案

您可以透過以下兩種方式來啟動這個專案：

### 方式一：使用 Visual Studio 2022 for Mac
1. 開啟 Visual Studio 2022 for Mac。
2. 開啟專案根目錄下的 `DotNet6API.sln` 方案檔。
3. 點擊上方的「執行」按鈕 (或按下 `Cmd + Enter`) 即可啟動專案並開啟瀏覽器。

### 方式二：使用 .NET CLI (終端機)
1. 開啟您的終端機 (Terminal)。
2. 切換目錄到專案原始碼所在的資料夾（包含 `.csproj` 的那一層，例如 `DotNet6API` 子目錄）：
   ```bash
   cd DotNet6API
   ```
3. 執行以下指令來啟動應用程式：
   ```bash
   dotnet run
   ```
   > 啟動成功後，終端機會顯示應用程式正在監聽的本機 URL (例如 `https://localhost:<port>`)。您可以在瀏覽器中開啟該網址，或者在網址後面加上 `/swagger` 來查看 Swagger UI API 文件並進行測試。
