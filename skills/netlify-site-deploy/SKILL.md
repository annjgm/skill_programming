---
name: netlify-site-deploy
description: 本專案（全國高中商類競賽程式設計資源網）之 Netlify 網頁更新、檔案驗證與部署標準作業流程。當需要更新線上網站、修改 netlify_deploy_site 內容或進行同步部署時使用。
---

# 程式設計競賽資源網 Netlify 部署作業指引

本 Skill 定義本專案靜態網頁（https://python-program-exam-resource.netlify.app/）的更新、驗證與部署標準作業流程。

---

## 一、 專案發布配置

- **線上正式站台**：`https://python-program-exam-resource.netlify.app/`
- **Netlify Site ID**：`5d640b28-fc5d-474f-8dfc-59ef891e82c5`
- **發布來源目錄**：`netlify_deploy_site/`（相對於專案根目錄，嚴禁發布根目錄）
- **核心靜態架構**：
  - `netlify_deploy_site/index.html`：主頁面
  - `netlify_deploy_site/style.css`：樣式表
  - `netlify_deploy_site/app.js`：切換與互動邏輯
  - `netlify_deploy_site/files/`：試題 PDF 與詳解 Markdown 下載資源

---

## 二、 標準作業程序（SOP）

### 步驟一：靜態內容與資源雙向同步
1. 確保 `詳解專區/` 的 Markdown 詳解已複製至 `netlify_deploy_site/files/詳解專區/{年度}/`。
2. 模擬試題詳解檔名必須冠上年度前綴（例如 `111年度01_Advanced模擬試題詳解.md`）。
3. 編輯 `netlify_deploy_site/index.html`，同步更新：
   - Hero 區域統計數據（份數統計）。
   - 對應年度區塊之清單項目與檔案大小資訊。
   - 所有下載連結必須帶有 `target="_blank" rel="noopener noreferrer"`，且不得包含強制下載屬性（`download`）。

### 步驟二：強制執行 Fail-Fast 連結驗證
在進行任何部署上傳前，必須執行：
```powershell
python verify_links.py
```
- **通過標準**：必須輸出 `[SUCCESS] All X files exist locally with valid sizes!`。
- **失敗阻擋**：若有任何缺失檔案或大小為 0 位元組，嚴禁進行部署。

### 步驟三：執行 Netlify 部署（雙途徑支援）

#### 途徑 A（最推薦）：Netlify Zip Deploy API
直接透過 Python 將 `netlify_deploy_site/` 打包為 Zip 串流並透過官方 API 發布（無須依賴 npm/CLI）：
- **Endpoint**: `POST https://api.netlify.com/api/v1/sites/5d640b28-fc5d-474f-8dfc-59ef891e82c5/deploys`
- **Headers**:
  - `Authorization: Bearer <NETLIFY_AUTH_TOKEN>`
  - `Content-Type: application/zip`
- **Body**: 排除 `.netlify` 後的純靜態目錄 Zip 二進位資料。

#### 途徑 B：Netlify CLI 指令
```powershell
npx --yes netlify deploy --prod --dir=netlify_deploy_site --site=5d640b28-fc5d-474f-8dfc-59ef891e82c5 --auth=<TOKEN>
```

#### 途徑 C：網頁拖曳（手動）
開啟 [Netlify Deploys 後台](https://app.netlify.com/sites/python-program-exam-resource/deploys)，將本機 `netlify_deploy_site` 資料夾直接拖入網頁下方完成發布。

### 步驟四：發布狀態確認與線上驗證
1. 查詢該次 `deployId` 狀態，確認 `state` 轉為 `ready`。
2. 透過 HTTP 請求檢查正式網址回應代碼：
   ```powershell
   curl.exe -sI https://python-program-exam-resource.netlify.app/
   ```
   確認回應為 `HTTP/2 200` 或 `HTTP/1.1 200 OK`。
3. 部署完畢後立即清除所有包含 Token 之暫存腳本。
