# 🤖 本地開源模型 LINE Bot (Ollama Gemma)

這是一支運行在個人本機電腦上的 LINE Bot。透過 Cloudflare 穿隧技術（Tunnel），讓 LINE 訊息能夠安全傳入本機，並交由本機執行的開源語言模型（Ollama Gemma）進行即時推論與聯網檢索。

---

## 📌 功能特點

1. **問答與聯網查證**：模型具備工具調用能力（Tool Calling / Function Calling）。遇到需要最新時事、價格、版本等問題時，會自動調用網路搜尋檢索資料並附上來源網址。
2. **行程自動辨識**：傳送自然語言行程（如「明天下午三點開會」），能自動解析日期時間結構化資料。
3. **圖片理解**：支援傳送照片（如活動海報），由多模態模型辨識海報內容與日期時間。
4. **雲端協作串接**：可整合 Google Workspace（Google Drive 照片備份、Google Calendar 行程寫入、Google Sheets 訊息紀錄）。

---

## 🛠️ 怎麼裝（環境準備）

### 1. 系統需求
- Windows 10 / 11
- Python 3.10 以上（本專案使用 Python 內建標準函式庫，**無需 `pip install` 額外套件**）

### 2. 安裝 Ollama 與模型
1. 安裝 Ollama：
   ```powershell
   winget install --id Ollama.Ollama -e --accept-package-agreements --accept-source-agreements --silent
   ```
2. 下載模型（可依硬體規格選擇）：
   - 標準版（支援文字與視覺，約 6.6 GB）：
     ```powershell
     ollama pull gemma4:e4b
     ```
   - 輕量版（若顯卡記憶體較小）：
     ```powershell
     ollama pull gemma4:e2b
     ```

### 3. 安裝 Cloudflared（通道工具）
讓 LINE 的 Webhook 能夠連線進本機：
```powershell
winget install --id Cloudflare.cloudflared -e --accept-package-agreements --accept-source-agreements
```

### 4. （可選）安裝 Google Workspace CLI (gws)
若需要雲端硬碟存圖、排日曆、寫試算表功能，需安裝 Node.js 與 gws：
```powershell
winget install --id OpenJS.NodeJS.LTS -e --silent
npm install -g --allow-scripts=@googleworkspace/cli @googleworkspace/cli
gws auth login -s "drive,sheets,calendar"
```

---

## 🚀 怎麼跑（執行方式）

### 1. 準備 LINE Developers 金鑰
前往 [LINE Developers Console](https://developers.line.biz/)：
- **Channel Access Token (long-lived)**：位於 `Messaging API` 分頁最下方。
- **Channel Secret**：位於 `Basic settings` 分頁。
- （可選）**Google 試算表 ID**：欲記錄訊息的試算表網址中段英數字。

### 2. 設定環境變數或啟動程式
可以直接在 PowerShell 啟動程式，程式會引導輸入金鑰（輸入密鑰時畫面不顯示字元屬正常防窺保護）：

```powershell
python line_bot_local.py
```

或建立 `.env` 檔案（已加入 `.gitignore` 防止洩漏）：
```env
LINE_CHANNEL_ACCESS_TOKEN=你的長效通行碼
LINE_CHANNEL_SECRET=你的ChannelSecret
MODEL=gemma4:e4b
```

### 3. 執行流程
1. 程式啟動後會先檢查本機 Ollama 服務與載入模型。
2. 自動建立 Cloudflare 穿隧通道，並將臨時通道網址自動更新至 LINE 後台的 Webhook Endpoint。
3. 終端機顯示 `開好了。手機傳訊息給 bot 試試看` 即代表連線成功。
4. 結束時按 `Ctrl + C`，程式會自動將 LINE Webhook 還原為原本設定並關閉通道。

---

## 💡 我改了什麼（程式修改紀錄）

相較於課堂範例程式，本版本進行了以下幾項改進與相容性強化：

1. **Windows 終端機 UTF-8 編碼保護**：
   - 在 Windows 環境（預設 `cp950` 代碼頁）下，原程式輸出 Emoji（如 🤖、🔍、📝、✅、⚠️）容易引發 `UnicodeEncodeError` 導致程式異常崩潰。
   - 新增 `sys.stdout.reconfigure(encoding="utf-8", errors="replace")` 與 `sys.stderr` 保護機制，確保終端機能正常印出完整圖示日誌。
2. **新增 `.env` 輕量化設定檔支援**：
   - 實作無額外依賴（純 Python 標準庫）的 `load_dotenv()` 函式。
   - 允許將 `LINE_CHANNEL_ACCESS_TOKEN`、`LINE_CHANNEL_SECRET` 與 `MODEL` 寫在本機 `.env` 內，重啟除錯時無需手動重複複製貼上，大幅提升開發與測試效率。
3. **路徑相容性與容錯增強**：
   - 強化 `cloudflared` 與 `gws` 的本機可執行檔自動搜尋路徑（即使環境變數 PATH 尚未更新亦能自動由預設安裝路徑偵測）。
   - 在模型載入處加上明確的例外提示，若本機 Ollama 尚未啟動或尚未 pull 模型時能給出清晰指引。

---

## 📱 測試與驗證範例

- **測試問答（需聯網查證）**：
  - 提問：「2026 年諾貝爾物理獎得主是誰？」
  - 手機端：顯示結論與附帶之官方新聞稿來源連結。
  - 電腦端終端機：顯示 `🔍 模型決定搜：2026年諾貝爾物理獎得主` 與 `✅ 回覆了` 日誌。
- **測試行程排定**：
  - 提問：「明天下午三點要跟專題組開會」
  - 手機端：解析為精確日期與時間並回報。
