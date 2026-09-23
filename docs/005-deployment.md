# 進階部署與更新

本文件補充 README 的部署流程，說明 Cloudflare Access、Secrets、手動檢查更新、本機開發與既有 D1 部署。Cloudflare Dashboard 的名稱與位置可能調整；若畫面不同，請以官方文件為準。

## 部署前準備

- [Cloudflare 帳號](https://dash.cloudflare.com/signup)
- [GitHub 帳號](https://github.com/signup)
- 部署時填入的一組 `CONFIG_ENCRYPTION_KEY`

Workers、D1、Queues、Workers AI 與 Browser Run 均有免費額度，但並非無限。需要瀏覽器的連接器會共用 Browser Run 額度；Workers Free Plan 目前每日包含 10 分鐘，最新限制請參考 [Browser Run Pricing](https://developers.cloudflare.com/browser-run/pricing/)。

## 一鍵部署細節

### 1. 產生加密金鑰

```bash
openssl rand -hex 32
```

`CONFIG_ENCRYPTION_KEY` 用來加密 D1 中的連接器設定，一般私人部署仍然需要。使用一鍵部署時只需填入一次，Cloudflare 會保存並在後續部署中沿用；另存一份於密碼管理器。若日後要重建 Worker、搬移環境或沿用既有 D1，必須使用相同金鑰，否則需要重新設定所有連接器。不要將金鑰放進 repository、資料庫匯出檔或 log，也不要在既有部署中任意更換或刪除。

### 2. 執行 Deploy to Cloudflare

先建立公開 fork 並檢查要部署的 commit。將 README 的 Deploy URL 換成自己的 fork URL；Cloudflare 會從 fork 再建立第二個部署用 repository。若不想公開 fork，可用 GitHub **Import repository** 建立私人獨立副本，再在 Cloudflare 連接該 repository、設定 D1 與 bindings。兩種方式都要核對部署 repository、Workers Builds 連接的 repository 與 production branch。

先在 Cloudflare Zero Trust 建立只允許自己身分的 Access Application 與 policy，啟用 MFA，取得真實 Team Domain 與 Audience (aud)。部署表單填入實際 `TEAM_DOMAIN`、`POLICY_AUD` 和 `CONFIG_ENCRYPTION_KEY`，設定 `DEMO_MODE=false`，正式 Worker 不設定 `LOCAL_DEV_MODE`。不要使用暫時的 Audience 或 `POLICY_AUDS` 佔位值。

Deploy to Cloudflare 會建立部署用 repository、D1 並設定 Workers Builds。本專案的 build 與 deploy script 也會檢查排程同步所需的 `taiwan-fin-hub-sync` Queue，缺少時自動建立；部署 script 會保留既有 VAPID 金鑰，初次部署則自動產生。

若使用 Cloudflare Workers Builds 自動產生的 API token，請確認它具有帳戶層級的 **Queues Read** 與 **Queues Edit** 權限，否則 Queue 檢查或建立會失敗。

## Cloudflare Access

### 啟用登入保護

1. 前往 **Workers & Pages** 並選擇部署完成的 Worker。
2. 開啟 **Settings → Domains & Routes**。
3. 在 `workers.dev`、自訂網域及其他可到達此 Worker 的公開網址啟用 Cloudflare Access；無法保護的網址先停用。

部分 Dashboard 版本會顯示 **Domains** 頁籤及 **Public／Restricted** 選項，將網址設為 **Restricted** 即可。最新操作方式請參考 [Cloudflare Access for Workers](https://developers.cloudflare.com/changelog/post/2025-10-03-one-click-access-for-workers/)。

<img src="../images/deploy-domains-restricted.png" alt="啟用 Cloudflare Access" width="700">

啟用後，從 Access Application 取得：

- **Audience (aud)**：設為 Worker Secret `POLICY_AUD`。
- **JWKs URL**：取出前面的 Team Domain，設為 `TEAM_DOMAIN`，例如 `https://yourteam.cloudflareaccess.com`。

前往 Worker 的 **Settings → Variables and secrets** 儲存這兩項設定。

<img src="../images/deploy-secrets.png" alt="設定 Cloudflare Access Secrets" width="700">

若確實要接受多個 Access Application，才設定 `POLICY_AUDS` 為經核對的實際 Audience 清單，以逗號或空白分隔。一般單一部署只需 `POLICY_AUD`。在輸入銀行憑證前，確認 `DEMO_MODE=false`、`LOCAL_DEV_MODE` 未設定，並在登出狀態測試每個公開網址的 `/api/summary`：應由 Access 擋下，不應回傳金融資料；登入後再確認應用程式可用。

### 使用 Cloudflare 帳號登入

Cloudflare Access 預設可能使用 Email OTP。限制為自己控制的身分並啟用 MFA：

1. 前往 **Zero Trust → Integrations → Identity providers**，新增或開啟 **Cloudflare** Identity Provider，啟用 **Restrict to account members**。這項限制仍可能允許其他帳號成員。
2. 前往 **Access controls → Applications → taiwan-fin-hub → Authentication**，將登入方式設為 Cloudflare；若只保留此方式，可啟用 **Apply instant authentication**。
3. 在此 Application 的 **Policies** 中，建立或編輯 **Allow** policy，讓唯一的 **Include** 精確指定使用者本人的 email／身分；移除 **Everyone**、整個帳號或其他會允許更多人的 policy。以另一個帳號成員測試，確認無法登入。
4. 依 [Cloudflare MFA 指引](https://developers.cloudflare.com/cloudflare-one/access-controls/policies/mfa-requirements/)先在 organization 啟用 **independent MFA**，再於 Application 的 **Authentication → MFA** 選 **Custom MFA settings**、允許的驗證器與較短的 Authentication duration；核對 Policy 的 MFA 設定沒有覆寫為 **Disable MFA**。Policy 設定優先於 Application，Application 優先於 organization；實際登入一次確認會要求第二因子。

### 登入期限

在 Access Application、Policy 與 **Access controls → Access settings** 的全域設定中選擇較短的 Session Duration；實際期限會採最短的值。避免將金融資料的登入狀態延長至一個月。

## 手動檢查更新

部署用 repository 不應安裝自動同步上游的 workflow，也不要讓未審查的上游 commit 直接進入 production branch。每次更新時：

1. 查看上游 diff、安全公告及相依版本，選定要採用的 commit，並在 fork 或私人獨立副本合併、解決衝突及執行驗證。
2. 核對部署 repository 與 Workers Builds 連接的 repository／production branch，再將已審查版本更新至該分支；推送可能立即觸發建置與部署。
3. 部署後核對公開網址的 Access 保護、`/api/summary` 登出回應及資料顯示。若 Queue 建立失敗，檢查 Workers Builds API token 是否具有帳戶層級的 Queues Read 與 Queues Edit。

只有連接器設定使用 `CONFIG_ENCRYPTION_KEY` 作應用層加密，金融資料表沒有應用層加密。因此 D1 匯出／備份與 log 都屬敏感資料，不得提交 Git，須限制存取並妥善刪除。正式與開發環境使用不同 D1；先維持連接器排程停用，只設定一個連接器並手動同步，將結果與銀行原始紀錄核對後才啟用所需排程。D1 Time Travel 的保留期限依方案為 7 或 30 天，不可當成長期備份。

### 目前的瀏覽器相依警示

`npm audit --omit=dev` 仍會列出 `@cloudflare/puppeteer@1.1.0` → `@puppeteer/browsers@2.2.4` → `extract-zip@2.0.1` 的 [symlink 路徑穿越](https://github.com/advisories/GHSA-jmr9-qjv8-65gv)與[任意檔案寫入](https://github.com/advisories/GHSA-7pqw-9j4j-h8q3)警示；`extract-zip` 目前沒有修補版。這些問題需要解壓攻擊者控制的 ZIP。專案的銀行連接器使用 Cloudflare Browser Rendering binding，沒有接受 ZIP 上傳或呼叫瀏覽器下載／解壓 API；因此尚未發現可由使用者資料觸發的路徑，但仍須追蹤上游套件修補，不應以強制降版 Puppeteer 當成已修復。相同相依鏈的 `ip-address` 已在 lockfile 更新至 10.7.2。`yauzl` 鎖定 2.10.0；[影響 3.2.0 的公告](https://github.com/advisories/GHSA-gmq8-994r-jv83)不適用。

## 本機開發

建立私人設定檔並填入開發用 D1 Database ID 與加密金鑰：

```bash
cp apps/worker/wrangler.local.toml.example apps/worker/wrangler.local.toml
cp apps/worker/.dev.vars.example apps/worker/.dev.vars
npm install
npx wrangler login
npm run dev
```

範例設定中的 D1 與 Workers AI 使用 remote binding，會連到 Cloudflare 資源。請使用獨立的開發 D1，不要直接操作正式資料。

常用資料庫遷移指令：

```bash
npm run db:migrate:local -w @taiwan-fin-hub/worker
npm run db:migrate:remote
```

中信行動銀行的 TLS endpoint 無法由 local workerd 直接連線，因此 `npm run dev` 會自動啟動只監聽 `127.0.0.1`、限制目的端點並使用單次隨機 token 的 Node relay；正式 Worker 不使用此 relay。

## 部署至既有 D1

若要從本機部署至既有 D1，可在 repository 根目錄複製 `wrangler.toml` 為被忽略的 `wrangler.private.toml`，填入正確的 `database_id`，再執行：

```bash
XDG_CONFIG_HOME=.wrangler-config npx wrangler d1 migrations apply DB \
  --remote --config wrangler.private.toml
XDG_CONFIG_HOME=.wrangler-config node scripts/deploy-with-vapid.mjs \
  --config wrangler.private.toml
```

執行前請再次確認 `database_id`、Worker 名稱與所有 bindings 都指向預期環境。資料庫 migration 會修改遠端 schema，不要使用未確認的正式資料庫進行測試。

若既有 D1 已儲存連接器設定，部署時也必須提供原本相同的 `CONFIG_ENCRYPTION_KEY`；新的隨機金鑰無法解密既有資料。
