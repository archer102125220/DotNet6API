# 使用 asdf 在 macOS 安裝與管理 .NET SDK

這份文件記錄了如何在 macOS 環境下，使用套件版本管理工具 [asdf](https://asdf-vm.com/) 來安裝和管理 .NET SDK。

## 1. 系統環境準備

請確保您的 Mac 上已經安裝了 `asdf`。如果您尚未安裝，可以透過 Homebrew 來安裝：

```bash
# 透過 Homebrew 安裝 asdf
brew install asdf

# 請根據您使用的 Shell (如 zsh 或 bash)，將 asdf 載入腳本加入至環境變數設定檔中（例如 ~/.zshrc 或 ~/.bash_profile）
# (詳細請參考 asdf 官方安裝文件)
```

## 2. 加入 .NET Core 外掛 (Plugin)

首先，您需要將 .NET Core 的外掛加入 asdf。執行以下指令：

```bash
asdf plugin add dotnet-core https://github.com/emersonmx/asdf-dotnet-core.git
```
*(提示：通常只需執行 `asdf plugin add dotnet-core`，asdf 也會自動找到官方列表中的路徑)*

## 3. 安裝 .NET SDK

### 查看可用的版本
您可以列出所有可透過 asdf 安裝的 .NET 版本：

```bash
asdf list all dotnet-core
```
這會列出非常多版本（包括不同的 Patch 號）。建議找出您需要的 Major 版本最新版（如 `8.0.xxx` 或 `7.0.xxx`）。

### 執行安裝
假設我們想要安裝最新版的 .NET 8.0 SDK (例如 `8.0.204`)，或者安裝本專案所需的 `6.0.425`，請執行：

```bash
# 安裝 .NET 8.0
asdf install dotnet-core 8.0.204

# 安裝 .NET 6.0 (本專案需求)
asdf install dotnet-core 6.0.425
```

> **注意**：下載與解壓縮過程可能會需要一些時間，請耐心等候。

## 4. 設定與切換版本

安裝完成後，您必須告訴 asdf 在哪裡使用這個版本。您可以選擇設定為全域 (Global) 或專案目錄 (Local) 層級。

### 設定為全域 (Global)
如果您希望整個系統預設都使用某個版本（例如最新版）：

```bash
# -u 參數等同於 --home，會寫入 ~/.tool-versions (這是新版 asdf 的語法)
asdf set -u dotnet-core 8.0.204
```
這會在您的主目錄 (`~/.tool-versions`) 中記錄該版本。

### 設定為單一專案 (Local)
如果您只希望在這個特定專案中使用，請進入專案根目錄後執行（以本專案使用的 `6.0.425` 為例）：

```bash
asdf set dotnet-core 6.0.425
```
這會在當前專案資料夾下建立或更新 `.tool-versions` 檔案，確保團隊其他開發者如果也用 asdf，能自動匹配對應的版本。

## 5. 驗證安裝

完成設定後，檢查 `dotnet` 是否已成功透過 asdf 的 shim 載入，並確認版本號：

```bash
dotnet --version
```
如果出現對應的 SDK 版本號 (例如 `8.0.204`)，代表您已成功透過 asdf 裝好並啟用了 .NET SDK！

## 6. 使用 `global.json` 鎖定專案 SDK 版本

除了 asdf 的 `.tool-versions` 之外，.NET 官方提供了一個更原生的方式來控制專案使用的 SDK 版本，那就是 `global.json`。

### `global.json` 的作用
`global.json` 檔案放在專案目錄，用來告訴 `dotnet` CLI 在建置或執行此專案時，必須使用哪個特定版本的 .NET SDK。這能確保團隊中所有開發者和 CI/CD 流程都使用完全一樣的 SDK 版本，避免因為 SDK 版本差異導致不可預期的建置或編譯錯誤。以這份專案為例，它鎖定了 `6.0.425` 版的 SDK。

### 設定與使用方式
您可以直接在專案根目錄建立或編輯 `global.json` 檔案。例如，本專案的 `global.json` 內容如下：

```json
{
  "sdk": {
    "version": "6.0.425"
  }
}
```

如果您想透過指令快速產生這個檔案，可以在專案目錄下執行：
```bash
dotnet new globaljson --sdk-version 6.0.425
```
設定好之後，只要在該目錄下執行任何 `dotnet` 相關指令，CLI 就會自動優先使用 `global.json` 所定義的 SDK 版本。

> **⚠️ 極度重要：asdf 與 global.json 的版本衝突地雷**
>
> 因為 `asdf` 會將每個安裝的 SDK 版本實體隔離（各自存放在 `~/.asdf/installs/dotnet-core/<版本號>`），這與原本在 Windows 上所有 SDK 都裝在同一個目錄下的行為不同。
> 如果您的 `.tool-versions` (asdf 啟用的版本) 是 `8.0.204`，但專案內的 `global.json` 鎖定為 `6.0.425`，執行 `dotnet` 指令時會報錯：「找不到指定的 SDK 版本」。
> 
> **解決方式**：在使用 `global.json` 的專案中，**必須確保 asdf 的 Local 版本 (`.tool-versions`) 設定與 `global.json` 的版本完全一致**。也就是在專案內一定要正確執行過 `asdf set dotnet-core 6.0.425`。

---

## 7. 解決 IDE 找不到 .NET SDK 的問題 (設定 DOTNET_ROOT)

透過 asdf 安裝的 .NET SDK 會存放在 asdf 的內部資料夾中，並且透過 shim 來執行。這會導致 IDE（如 JetBrains Rider、Visual Studio Code）或 C# 開發套件無法自動偵測到 .NET SDK 的實際安裝路徑，進而出現找不到 SDK 的錯誤。

為了解決這個問題，`asdf-dotnet-core` 外掛提供了一個內建的輔助腳本。它能自動根據您當前所在的目錄，動態切換並設定 `DOTNET_ROOT` 環境變數。

請將以下指令加入您的 Shell 設定檔（例如 `~/.zshrc` 或 `~/.bash_profile`）的結尾（請務必放在 asdf 初始化指令**之後**）：

**對於 Zsh 使用者 (macOS 預設)：**
```bash
. ~/.asdf/plugins/dotnet-core/set-dotnet-home.zsh
```

**對於 Bash 使用者：**
```bash
. ~/.asdf/plugins/dotnet-core/set-dotnet-home.bash
```

加入並存檔後，請執行 `source ~/.zshrc` (或對應檔案) 重新載入設定。未來只要透過終端機啟動 IDE，或是重新啟動 IDE，它們就能透過 `DOTNET_ROOT` 變數正確找到 .NET SDK。

---

## 補充筆記 (關於 .NET 全域工具)

如果您習慣透過 `dotnet tool install -g <tool_name>` 安裝全域命令列工具 (例如 `dotnet-ef`)，它們預設會安裝在 `~/.dotnet/tools`。

針對 asdf 使用者，若系統找不到安裝好的全域工具，建議將以下路徑加入到您的 `~/.zshrc` 或 `~/.bash_profile` 中：

```bash
export PATH="$HOME/.dotnet/tools:$PATH"
```
加入後，記得執行 `source ~/.zshrc` 使環境變數生效。
