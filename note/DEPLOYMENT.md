# DotNet6API 部署指南

這份文件說明如何將 `DotNet6API` 專案發布與部署至不同的生產環境。

## 1. 準備發布版本 (Publish)

在部署之前，您需要先將應用程式編譯為可執行的發布版本。請打開終端機並在專案根目錄下執行以下指令：

```bash
dotnet publish -c Release -o ./publish
```

- `-c Release`：使用 Release 模式編譯，優化程式碼並移除除錯資訊。
- `-o ./publish`：將編譯後的檔案輸出到專案根目錄下的 `publish` 資料夾。

---

## 2. 部署選項

### 選項 A：部署至 Linux (搭配 Nginx 或 Apache)

如果您的伺服器是 Ubuntu 或 CentOS，可以使用反向代理 (Reverse Proxy) 伺服器來處理 HTTP 請求。

1. **安裝 .NET 執行環境 (Runtime)**
   請確保伺服器上安裝了 **.NET 6 ASP.NET Core Runtime**。
   [官方安裝指南](https://docs.microsoft.com/zh-tw/dotnet/core/install/linux)

2. **上傳檔案**
   將剛才產生的 `publish` 資料夾內的檔案透過 SCP 或 FTP 上傳到伺服器上的目錄（例如：`/var/www/DotNet6API`）。

3. **設定 Kestrel 服務 (Systemd)**
   建立一個 systemd 服務檔案 `/etc/systemd/system/kestrel-dotnet6api.service`，內容如下：

   ```ini
   [Unit]
   Description=DotNet6API Application

   [Service]
   WorkingDirectory=/var/www/DotNet6API
   ExecStart=/usr/bin/dotnet /var/www/DotNet6API/DotNet6API.dll
   Restart=always
   RestartSec=10
   SyslogIdentifier=dotnet-api
   User=www-data
   Environment=ASPNETCORE_ENVIRONMENT=Production

   [Install]
   WantedBy=multi-user.target
   ```

   啟用並啟動服務：
   ```bash
   sudo systemctl enable kestrel-dotnet6api.service
   sudo systemctl start kestrel-dotnet6api.service
   ```

4. **設定 Nginx 反向代理**
   在 `/etc/nginx/sites-available/dotnet6api` 建立設定檔：

   ```nginx
   server {
       listen 80;
       server_name example.com; # 替換成您的網域

       location / {
           proxy_pass         http://localhost:5000; # Kestrel 預設的 Port
           proxy_http_version 1.1;
           proxy_set_header   Upgrade $http_upgrade;
           proxy_set_header   Connection keep-alive;
           proxy_set_header   Host $host;
           proxy_cache_bypass $http_upgrade;
           proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header   X-Forwarded-Proto $scheme;
       }
   }
   ```
   啟用設定並重新載入 Nginx：
   ```bash
   sudo ln -s /etc/nginx/sites-available/dotnet6api /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl reload nginx
   ```

---

### 選項 B：使用 Docker 部署

如果您的專案有包含 `Dockerfile`，可以使用 Docker 將應用程式容器化。

1. **建立 Dockerfile**
   在專案根目錄下建立一個 `Dockerfile`（如果尚未建立）：

   ```dockerfile
   FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
   WORKDIR /app
   EXPOSE 80
   EXPOSE 443

   FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
   WORKDIR /src
   COPY ["DotNet6API/DotNet6API.csproj", "DotNet6API/"]
   RUN dotnet restore "DotNet6API/DotNet6API.csproj"
   COPY . .
   WORKDIR "/src/DotNet6API"
   RUN dotnet build "DotNet6API.csproj" -c Release -o /app/build

   FROM build AS publish
   RUN dotnet publish "DotNet6API.csproj" -c Release -o /app/publish

   FROM base AS final
   WORKDIR /app
   COPY --from=publish /app/publish .
   ENTRYPOINT ["dotnet", "DotNet6API.dll"]
   ```

2. **建立與執行 Docker 映像檔**

   ```bash
   # 建立映像檔
   docker build -t dotnet6api .

   # 執行容器 (將本機的 8080 Port 映射到容器的 80 Port)
   docker run -d -p 8080:80 --name my_dotnet6api dotnet6api
   ```

---

### 選項 C：部署至 Azure App Service

1. 在 Visual Studio 中打開專案。
2. 對專案點擊右鍵，選擇 **發布 (Publish)**。
3. 目標選擇 **Azure**，具體服務選擇 **Azure App Service (Windows 或 Linux)**。
4. 登入您的 Azure 帳號並按照引導完成發布流程。
5. （也可使用 Azure CLI 或 GitHub Actions 自動部署，可參考 [Azure 官方文件](https://learn.microsoft.com/zh-tw/azure/app-service/deploy-github-actions?tabs=applevel))

---

## 3. 環境變數與組態設定

生產環境中，建議將敏感資料（如資料庫連線字串、API 金鑰等）存放在環境變數，避免直接寫死在 `appsettings.json` 中。

- 可以在伺服器系統中設定環境變數：`ASPNETCORE_ENVIRONMENT=Production`
- 您的程式碼會自動加載 `appsettings.Production.json`，請確認您已根據生產環境配置好該檔案，例如 CORS 設定、資料庫連線等。
