# idGoLive 控制台 · GitHub Pages

靜態 PWA（可離線），把整個資料夾丟到 GitHub Pages 就能用，不需要後端、不需要伺服器。

第一次有網路時開啟後，Service Worker 會快取整頁；之後 iPhone 連記錄器 Wi-Fi 即使沒網路，
也能從主畫面像 App 一樣直接開。

## 部署

1. GitHub 新建一個 repository（例如 `dashcam-console`），設為 Public。
2. 把這四個檔案全部上傳到 repo 根目錄：`index.html`、`sw.js`、`manifest.json`、`icon.png`。
3. repo → **Settings** → **Pages** → Build and deployment：
   - Source: **Deploy from a branch**
   - Branch: `main` / `/ (root)`
4. 等 1～2 分鐘，網址是：
   ```
   https://<你的帳號>.github.io/<repo名稱>/
   ```

## 頁面功能

- **連線檢測**：心跳測試（`cmd=3016`）、目前所有設定、韌體版本、串流網址、支援指令
- **即時影像**：三步啟動 MJPEG（照片模式）+ 一鍵降位元率
- **錄影/檔案**：錄影、拍照、回放、檔案列表、SD 卡網頁
- **開關設定**：循環錄影、WDR、錄音、時間浮水印、移動偵測、自動錄影
- **數值設定**：錄影/預覽位元率與解析度、G-Sensor、自動斷電、語言、日期時間
- **系統**：儲存設定、重連 Wi-Fi、格式化、重置（有確認對話框）

## 關鍵設計：為什麼 HTTPS 的 GitHub Pages 還能控制區網記錄器

瀏覽器不允許 HTTPS 頁面用 `fetch()` 去讀 `http://192.168.1.254`（混內容 + CORS +
Private Network Access）。所以這個頁面**完全不用 fetch**，所有按鈕都是
`<a href="http://192.168.1.254/..." target="_blank">`。

「點連結開新分頁」屬於使用者主動觸發的 top-level navigation，不受上述限制，
因此可以直接看到記錄器回傳的 XML / MJPEG 影像，不需要任何中繼或代理。

IP 切換（A: `192.168.1.254` / B: `192.168.169.1`）只是用 JS 改連結的 `href`，
不發出任何請求，所以也不受限制。選擇會存到 `localStorage`。

## 使用限制

- **第一次必須有網路**讓 Service Worker 安裝完成（頁面會顯示「離線可用」）。
  之後 iPhone 連上記錄器 AP 沒網路也能用。
- Safari 分享選單 →「加入主畫面」，可以像 App 一樣全螢幕啟動。
- iOS 長時間不使用或儲存空間不足時，可能會清除離線快取；偶爾連網路開一次即可恢復。
- 即時影像只在**照片模式**（`cmd=3001&par=0`）提供 HTTP MJPEG `:8192`。
  錄影模式只有 RTSP H.264，Safari 不支援。
- 點按鈕會開新分頁看完再返回，不是單頁非同步更新（這是刻意設計，見上）。

## 檔案

- `index.html` — 介面（HTML + CSS + JS）
- `sw.js` — Service Worker，離線快取
- `manifest.json` — PWA 設定（加入主畫面用）
- `icon.png` — 主畫面圖示
