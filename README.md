<p align="center">
  <img src="apps/web/public/icon-512x512.png" alt="不用記帳 Logo" width="160">
</p>

# 不用記帳

**ALL SET — 自動同步銀行、信用卡、投資與電子發票的自架個人財務整合工具。**

**可免費自架：** 可透過 [Cloudflare Workers Free Plan](https://developers.cloudflare.com/workers/platform/pricing/) 一鍵部署，不需要自行準備伺服器；一般個人低頻使用可從免費方案開始。

## 目前介面

以下畫面使用匿名 Demo 資料，取自目前版本。

| 桌面版總覽                                                                                                                         | 手機版總覽                                                                                                                                     |
| ---------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| <a href="images/screenshots/01-dashboard.png"><img src="images/screenshots/01-dashboard.png" alt="桌面版總覽畫面" width="720"></a> | <a href="images/screenshots/02-overview-mobile.png"><img src="images/screenshots/02-overview-mobile.png" alt="手機版總覽畫面" width="260"></a> |

| 資產清冊                                                                                                       | 活動分析                                                                                                           | 設定與資料來源                                                                                                           |
| -------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------ |
| <a href="images/screenshots/03-assets.png"><img src="images/screenshots/03-assets.png" alt="資產清冊畫面"></a> | <a href="images/screenshots/04-activity.png"><img src="images/screenshots/04-activity.png" alt="活動分析畫面"></a> | <a href="images/screenshots/05-settings.png"><img src="images/screenshots/05-settings.png" alt="設定與資料來源畫面"></a> |

## 支援資料來源

| 資料來源     | 支援內容                                                                                              | 登入與驗證                   |
| ------------ | ----------------------------------------------------------------------------------------------------- | ---------------------------- |
| 電子發票載具 | 載具發票與品項明細                                                                                    | App 登入                     |
| 集保 e 存摺  | 交割帳戶餘額與明細（[支援銀行](https://epassbook.tdcc.com.tw/zh/g1.aspx)）、股票、ETF、基金持倉與交易 | App 登入；首次可能需要 OTP   |
| 玉山銀行     | 存款帳戶、餘額與交易；信用卡帳單與刷卡交易                                                            | 網銀登入                     |
| 國泰世華銀行 | 存款帳戶、餘額與交易；信用卡帳單與刷卡交易                                                            | 網銀登入；額外驗證需人工處理 |
| 永豐行動銀行 | 信用卡總覽、近期帳單與未出帳消費                                                                      | 網銀登入；AI 自動辨識驗證碼  |
| 台新銀行     | 信用卡額度、帳單、已入帳與即時授權消費                                                                | 網銀登入；AI 自動辨識驗證碼  |
| 中國信託銀行 | 存款帳戶、餘額與交易；信用卡帳單、已入帳、未出帳與即時消費明細                                        | App 登入                     |
| 新光銀行     | 臺外幣帳戶、餘額、交易明細與信用卡帳單                                                                | App 登入                     |
| 華南銀行     | 存款帳戶與餘額；信用卡帳單與刷卡明細                                                                  | 網銀登入；AI 自動辨識驗證碼  |
| 王道銀行     | 活存、定存、餘額與交易                                                                                | App 登入；AI 自動辨識驗證碼  |
| 第一銀行     | 存款帳戶、餘額與交易明細；信用卡帳單與刷卡明細                                                        | 網銀登入；AI 自動辨識驗證碼  |

## 使用限制

- 連接器依賴外部網頁、App API 與回應格式；資料來源改版後可能需要更新才能恢復同步。
- 系統不會繞過圖形驗證碼、OTP、裝置驗證等互動式安全機制；需要人工處理時會停止同步並顯示提示。
- 部分銀行自動登入可能中斷你正在使用的官方 App 或網銀工作階段。
- 資料更新時間與完整性取決於外部服務，不應視為銀行、券商或財政部的即時正式對帳資料。

## 免費部署

本專案使用的 Workers、D1、Queues、Workers AI 與 Browser Run 均提供免費額度。各項免費額度並非無限；超過服務限制時，相關功能可能暫停至額度重置。

**需要：** [Cloudflare 帳號](https://dash.cloudflare.com/signup)、[GitHub 帳號](https://github.com/signup)

### 步驟一：一鍵部署

先檢查這份[公開 fork](https://github.com/nyms0390/all-set-tw) 的程式碼及相依版本，並選定要部署的 commit。確認修補版本已在該 fork 後，再使用指向此 fork 的按鈕：

[![Deploy to Cloudflare](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/nyms0390/all-set-tw)

若從自己的 fork 部署，請把按鈕網址的 `url=` 參數改成自己的 GitHub repository URL；不要沿用指向原專案的部署按鈕。

Cloudflare 會從這個 fork 再建立**第二個部署用 repository**，並建立 D1、設定 Workers Builds。請在部署前後核對來源 fork、部署 repository 與 Workers Builds 連接的 repository／production branch；之後推送到部署分支可能觸發重新建置與部署。若不想公開 fork，可用 GitHub **Import repository** 從已審查的版本建立私人獨立副本，再於 Cloudflare 連接該私人 repository 並依[進階部署指引](docs/005-deployment.md)設定資源與 bindings。

Cloudflare Builds 會在 build 階段自動檢查並建立排程同步所需的 Queue；正式部署腳本也會再次檢查。既有安裝更新到使用 Queue 的版本時不需要手動建立資源。

首次使用時，依畫面透過 **Git account → New Github Connection → Install & Authorize** 授權 Cloudflare 存取 GitHub。

先在 Cloudflare Zero Trust 建立只允許自己身分的 Access policy，啟用 MFA 並設定較短的 session；取得實際 Team Domain 與 Application Audience (aud)。部署表單中的 `TEAM_DOMAIN`、`POLICY_AUD` 均填入實際值，`DEMO_MODE` 設為 `false`，正式 Worker 不設定 `LOCAL_DEV_MODE`；不要使用暫時值或新增 `POLICY_AUDS` 佔位設定。

`CONFIG_ENCRYPTION_KEY` 是系統加密連接器設定時必須使用的金鑰，可用下列指令產生：

```bash
openssl rand -hex 32
```

使用一鍵部署時只需填入一次，部署後由 Cloudflare 保存。另將此金鑰保存在密碼管理器；日後重建 Worker、搬移環境或沿用既有 D1 時必須使用相同金鑰，否則需要重新設定所有連接器。不要將金鑰提交到 Git、寫入匯出檔或任意更換。

填寫完成後點擊 **Deploy**。

### 步驟二：啟用登入保護

1. 前往 [Cloudflare Dashboard](https://dash.cloudflare.com/) → **Workers & Pages**，選擇剛建立的 `taiwan-fin-hub`。
2. 檢查 **Domains** 或 **Settings → Domains & Routes**，讓 `workers.dev`、自訂網域及任何其他可到達此 Worker 的公開網址都套用 Cloudflare Access；若有不受保護的網址，先停用該路徑。
3. 將 Worker 的 **Settings → Variables and secrets** 中 `TEAM_DOMAIN`、`POLICY_AUD` 核對為實際值，確認 `DEMO_MODE=false` 且沒有 `LOCAL_DEV_MODE` 或暫時的 `POLICY_AUDS`。

<img src="images/deploy-domains-restricted.png" alt="啟用 Cloudflare Access" width="700">

Access Application 會顯示以下資訊：

- **Audience (aud)**：填入 Worker Secret `POLICY_AUD`
- **JWKs URL**：取出前面的網域作為 `TEAM_DOMAIN`，例如 `https://yourteam.cloudflareaccess.com`

先完成這兩項設定，再進行登入與憑證設定。

<img src="images/deploy-secrets.png" alt="設定 Cloudflare Access Secrets" width="700">

### 步驟三：確認部署

1. 在登出狀態測試每個公開網址的 `/api/summary`，確認沒有金融資料回應且必須通過 Access；再登入確認能正常開啟。
2. 確認開發與正式環境使用不同 D1。先維持各連接器排程停用，只設定一個連接器並執行一次手動同步。
3. 將帳戶、餘額及交易與銀行原始紀錄核對；確認資料正確後，才在介面啟用所需的排程。匯出檔、D1 備份與 log 可能含敏感資料，應限制存取並妥善刪除。D1 Time Travel 保留期限依方案為 7 或 30 天，不能替代長期備份。

### 步驟四：限制登入身分與期限

Cloudflare Access 可能預設允許 Email OTP。僅允許自己控制的身分登入，啟用 MFA，並為 Application、Policy 及全域設定較短的 session duration。

#### 使用 Cloudflare 帳號登入

1. 前往 **Zero Trust → Integrations → Identity providers**，確認已有 **Cloudflare**；若沒有，點選 **Add new identity provider → Cloudflare**
2. 啟用 **Restrict to account members** 並儲存，避免非此 Cloudflare 帳號成員登入
3. 前往 **Zero Trust → Access controls → Applications → taiwan-fin-hub → Authentication**，將登入方式設為 **Cloudflare**
4. 若只使用此登入方式，可啟用 **Apply instant authentication**，略過登入方式選擇頁

新建立的 Zero Trust organization 通常已預設啟用 Cloudflare identity provider，不需要另外新增。

更多 Queue、Access 與更新操作請參考[進階部署與更新](docs/005-deployment.md)。

## 手動檢查更新

部署用 repository 不應安裝自動同步上游的 workflow。需要更新時，先檢查上游的變更、相依套件與安全公告，在 fork 或私人獨立副本中合併並測試選定的版本，再把經審查的 commit 更新至部署 repository。推送前核對 Cloudflare Workers Builds 連接的 repository 與 production branch，確認這次推送會部署哪個版本。

## 本機開發

建立不納入版本控制的私人設定，將 `wrangler.local.toml` 的 D1 Database ID 換成開發用資料庫，並在 `.dev.vars` 設定自己的 `CONFIG_ENCRYPTION_KEY`：

```bash
cp apps/worker/wrangler.local.toml.example apps/worker/wrangler.local.toml
cp apps/worker/.dev.vars.example apps/worker/.dev.vars
npm install
npx wrangler login
npm run dev
```

範例設定的 D1 與 Workers AI 會連到 Cloudflare remote binding，請勿使用正式資料庫。常用驗證指令：

```bash
npm run format:check
npm run typecheck
npm run verify:web
npm run test:backend
npm run build
```

本機 relay、資料庫遷移與既有 D1 部署方式請參考[進階部署與更新](docs/005-deployment.md)。

## 技術架構

前端使用 Svelte 5、TypeScript、Tailwind CSS 4 與 shadcn-svelte。

後端執行於 Cloudflare Workers，以 Hono 提供 API，並整合 D1、Access、Browser Run、Workers AI、Cron Triggers 與 Queues。

專案以 npm workspaces 管理 Web、Worker、共用型別、資料庫與連接器套件。

前後端與共用套件皆使用 TypeScript 7 型別檢查；Svelte 前端透過 `svelte-check --tsgo` 執行，並保留工具所需的 TypeScript 6 相依。

詳細設計請參考[後端架構](docs/002-backend-architecture.md)、[前端架構](docs/003-frontend-architecture.md)與[連接器開發](docs/004-connector-development.md)。

## 安全機制

- Cloudflare Access 是一般模式的登入閘道；Worker 會驗證 JWT 的簽章、issuer、audience 與有效期限。
- 連接器帳密以 `CONFIG_ENCRYPTION_KEY` 衍生的金鑰進行 AES-GCM 加密，D1 只儲存密文。
- 目前不支援金鑰輪替；若刪除或更換 Cloudflare 中的金鑰，必須重新設定所有連接器。

## 免責聲明

本程式僅供個人研究與自用，未與臺灣集中保管結算所、財政部、金融監督管理委員會、各銀行或任何金融機構合作，亦未獲前述機構授權或背書。本程式所呈現之資料以您自行提供之憑證取得，作者不保證資料之即時性、正確性與完整性，亦不對因使用本程式所產生之任何直接或間接損失負責。請勿將本程式用於任何商業用途。

## License

本專案採用 [MIT License](LICENSE)，並保留原專案的著作權與授權聲明。

> 本專案以 [kevchentw/taiwan-fin-hub](https://github.com/kevchentw/taiwan-fin-hub) 為基礎發展而來。感謝原作者與貢獻者奠定專案基礎。
