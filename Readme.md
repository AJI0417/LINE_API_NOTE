# LINE API 使用教學

本專案包含三個 Python 檔案，分別展示了與 LINE Messaging API 互動的不同方法。

---

## 使用 ngrok 進行本機測試

在開發 LINE Bot 時，LINE Platform 需要一個公開的 HTTPS 網址來發送 Webhook 事件。當你在本機（自己的電腦）上執行 Flask 伺服器時，網址會是 `http://localhost:5000` 或 `http://127.0.0.1:5000`，這是一個私有網址，LINE Platform 無法存取。

`ngrok` 是一個好用的工具，它可以為你的本機伺服器建立一個安全的公開網址（HTTPS），讓外部服務（如 LINE）可以連線到你的本機程式。

### 安裝 ngrok

1.  前往 [ngrok 官方網站](https://ngrok.com/download) 下載適合你作業系統的版本。
2.  解壓縮下載的檔案，你會得到一個 `ngrok.exe` 執行檔。

### 使用步驟

1.  **啟動你的 Python 伺服器**:
    打開一個終端機（命令提示字元或 PowerShell），執行其中一個 Python 檔案，例如：
    ```bash
    python LINE_BOT_SDK.py
    ```
    你的 Flask 伺服器將會在本機的 5000 通訊埠上運行。

2.  **啟動 ngrok**:
    打開 **另一個** 終端機，切換到 `ngrok.exe` 所在的目錄，然後執行以下指令：
    ```bash
    ngrok http 5000
    ```
    這個指令會告訴 ngrok 將所有傳送到它產生的公開網址的 HTTP 請求，都轉發到你本機的 `localhost:5000`。

3.  **取得公開網址**:
    啟動成功後，ngrok 的畫面會顯示你的公開網址，看起來像這樣：
    ```
    Forwarding                    https://<隨機字串>.ngrok-free.app -> http://localhost:5000
    ```
    其中 `https://<隨機字串>.ngrok-free.app` 就是 LINE 需要的公開網址。

4.  **設定 Webhook URL**:
    *   複製 ngrok 提供的 `https://...` 開頭的網址。
    *   在你的 Python 程式中，Webhook 的路徑是 `/callback`，所以你需要將這個路徑加到網址後面，變成：`https://<隨機字串>.ngrok-free.app/callback`。
    *   前往 [LINE Developers Console](https://developers.line.biz/console/)，選擇你的 Channel，進入 "Messaging API" 分頁。
    *   找到 "Webhook settings" -> "Webhook URL"，點擊 "Edit"，將上面組合好的完整網址貼上，然後儲存。
    *   確保 "Use webhook" 的開關是開啟的。

現在，當你的 LINE Bot 收到訊息時，LINE Platform 就會將事件透過 ngrok 轉發到你本機正在執行的 Python 程式了！

**注意**:
*   免費版的 ngrok 每次重新啟動時，都會產生一個新的隨機網址，所以你每次都需要去 LINE Developers Console 更新 Webhook URL。
*   執行 ngrok 的那個終-端機視窗必須保持開啟，關閉後通道就會中斷。

---

## `LINE_BOT_SDK.py` 與 `LINE_REST_API.py` 的差異

這兩個檔案展示了兩種不同的方法來建立 LINE Bot：一種是使用官方的 `line-bot-sdk`，另一種是直接呼叫 LINE 的 RESTful API。

### `LINE_BOT_SDK.py`：使用官方 SDK

這個檔案使用了 `line-bot-sdk` 這個 Python 套件。這是一個高階的抽象層，將許多與 LINE API 互動的細節封裝起來。

*   **優點**:
    *   **開發快速**: SDK 提供了簡單易用的介面，例如 `LineBotApi` 和 `WebhookHandler`，可以讓你用更少的程式碼處理訊息的接收、驗證和回覆。
    *   **可讀性高**: 程式碼語意清晰，專注於業務邏輯，而不是 API 的技術細節。
    *   **自動處理簽章驗證**: `WebhookHandler` 會自動幫你處理來自 LINE Platform 的請求簽章驗證，增加了安全性。
    *   **內建模型**: 提供如 `TextMessage`, `ImageMessage`, `StickerSendMessage` 等模型，方便你建立各種格式的訊息。

*   **缺點**:
    *   **彈性較低**: 如果 LINE API 推出了新的功能，但 SDK 還沒有更新支援，你就無法使用該功能。
    *   **依賴套件**: 需要額外安裝 `line-bot-sdk` 套件。

### `LINE_REST_API.py`：直接呼叫 RESTful API

這個檔案不依賴任何 LINE 的 SDK，而是使用 `requests` 套件直接對 LINE Messaging API 的端點 (Endpoint) 發送 HTTP 請求。

*   **優點**:
    *   **完全控制與最大彈性**: 你可以完全控制 HTTP 請求的每一個細節（Headers, Body），因此可以使用任何 LINE API 提供的功能，不受 SDK 版本的限制。
    *   **無額外依賴**: 除了 `requests`（通常是 Python 專案的標準配備），不需要安裝特定的 LINE 套件。
    *   **有助於理解底層運作**: 透過手動組合請求，你可以更深入地了解 LINE API 的運作原理。

*   **缺點**:
    *   **程式碼較冗長**: 每個 API 呼叫都需要手動設定 URL、Headers 和 JSON Payload，程式碼量較多。
    *   **需要自行處理細節**: 你需要自己處理簽章驗證、解析 JSON 回應等工作，增加了複雜度。

### 總結

| 特性 | `LINE_BOT_SDK.py` (使用 SDK) | `LINE_REST_API.py` (直接呼叫 API) |
| :--- | :--- | :--- |
| **抽象層級** | 高 | 低 |
| **開發速度** | 快 | 慢 |
| **程式碼量** | 少 | 多 |
| **彈性** | 受限於 SDK | 高 |
| **依賴** | `line-bot-sdk` | `requests` |
| **適合情境** | 快速開發、標準功能、新手入門 | 需要使用最新功能、需要完全控制、輕量級專案 |

---

## `LINE_SDK_REST.py` 的使用方法與時機

經過分析，`LINE_SDK_REST.py` 的內容與 `LINE_REST_API.py` 幾乎完全相同。兩者都展示了如何直接透過 `requests` 函式庫來呼叫 LINE Messaging API。

因此，你可以將 `LINE_SDK_REST.py` 視為 `LINE_REST_API.py` 的一個副本或備份。

### 使用方法

其使用方法與 `LINE_REST_API.py` 完全相同：

1.  **設定憑證**: 將 `YOUR_CHANNEL_ACCESS_TOKEN` 和 `YOUR_CHANNEL_SECRET` 替換成你自己的 LINE Bot 憑證。
2.  **執行程式**: 透過 `python LINE_SDK_REST.py` 啟動 Flask 伺服器。
3.  **設定 Webhook**: 將你的伺服器 URL（例如 `https://your-domain.com/callback`）設定到 LINE Developer Console 的 Webhook URL 中。

### 使用時機

直接呼叫 RESTful API（如此檔案所示範的）的時機與前述 `LINE_REST_API.py` 的建議相同：

1.  **需要使用 SDK 尚未支援的最新功能**: 當 LINE 剛推出新的 API 功能，而官方 SDK 還來不及更新時，你可以直接參考 API 文件來發送請求。
2.  **追求最大限度的客製化與控制**: 當你需要精確控制請求的每個細節，或是想用自己的方式來組織程式碼，而不希望被 SDK 的框架限制時。
3.  **不想為專案增加額外的依賴**: 如果你的專案很小，或是不想引入 `line-bot-sdk` 這個額外的套件，直接使用 `requests` 會讓專案更輕量。

總而言之，當你發現官方 SDK 無法滿足你的特定需求，或者你更喜歡貼近底層 API 的開發方式時，就可以參考 `LINE_SDK_REST.py` 或 `LINE_REST_API.py` 的寫法。