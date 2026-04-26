# 🍸 My Private Bar

> 個人私藏調酒工作站 — 管理你的酒櫃、收藏酒譜、讓 AI 幫你調出今晚的那一杯

![Static Badge](https://img.shields.io/badge/單檔案-HTML-gold?style=flat-square)
![Static Badge](https://img.shields.io/badge/跨裝置-Gist_同步-4a8b5a?style=flat-square)
![Static Badge](https://img.shields.io/badge/AI-推薦-c9a84c?style=flat-square)
![Static Badge](https://img.shields.io/badge/無需安裝-直接使用-555?style=flat-square)

---

## ✨ 功能總覽

| 功能 | 說明 |
|------|------|
| 🍾 **酒櫃管理** | 記錄手邊所有酒和飲料材料，分類管理、標注庫存量 |
| 📖 **酒譜收藏** | 手動新增或從 AI 推薦一鍵收藏，支援星級評分 |
| ✨ **AI 推薦** | 根據現有材料、口味偏好，即時生成經典、創意調酒或採購建議 |
| ⚙️ **偏好設定** | 設定喜歡的口味、基酒、場合，讓 AI 推薦更貼近你 |
| 🔄 **Gist 同步** | 透過 GitHub Gist 實時同步，在任何裝置上都能看到你的酒吧 |

---

## 🖥️ 使用方式

這是一個**單檔案 HTML 應用**，不需要安裝任何東西。

1. 下載 `my_private_bar.html`
2. 用瀏覽器直接打開
3. 開始管理你的酒吧 🍹

或者直接部署到 GitHub Pages、Netlify、Cloudflare Pages 等靜態托管平台。

---

## 🔄 跨裝置同步（GitHub Gist）

My Private Bar 使用 **GitHub Gist** 作為免費的雲端儲存，讓你在手機、電腦、平板之間無縫切換。

### 設定步驟

**第一步：建立 GitHub Personal Access Token**

1. 前往 [github.com/settings/tokens](https://github.com/settings/tokens/new?scopes=gist)
2. 在 `Select scopes` 中只勾選 **`gist`** 即可
3. 點「Generate token」並複製 token（只會顯示一次！）

**第二步：在應用中設定**

1. 開啟 App → 點「⚙️ 偏好設定」
2. 在頁面最上方找到「GitHub Gist 同步」區塊
3. 貼上你的 Token
4. Gist ID 欄位**留空**（第一次會自動建立新 Gist）
5. 點「儲存並連線」

**第三步：在其他裝置使用**

1. 在新裝置上打開同一份 HTML
2. 到偏好設定貼入相同的 Token 和 Gist ID
3. 點「↓ 拉取」即可同步所有資料

### 同步狀態指示

右上角小圓點即時顯示同步狀態：

- ⚫ 灰色 — 尚未連線
- 🟡 金色閃爍 — 同步中
- 🟢 綠色 — 同步成功
- 🔴 紅色 — 同步失敗（請確認 Token 是否有效）

> **隱私說明**：Token 和 Gist ID 只儲存在你本機的 `localStorage`，不會經過任何第三方伺服器。Gist 預設建立為 **Secret**（私密），只有擁有連結才能查看。

---

## 🤖 AI 推薦功能

AI 推薦功能透過 [Cloudflare Worker](https://workers.cloudflare.com/) 代理呼叫 Claude API，提供三種模式：

### 🥂 經典調酒
根據你的庫存，推薦 3 款可以立刻製作的經典款，附完整配方與步驟。

### 🎨 創意調酒
AI 發揮創意，推薦 2–3 款獨特調酒，有靈感來源、個性描述，適合想嘗試新東西的時候。

### 🛒 採購建議
分析你的庫存缺口，推薦最值得購買的下一瓶酒，並說明與現有材料的搭配方式。

> AI 推薦完成後，下方會出現「📖 儲存此推薦至酒譜收藏」按鈕，一鍵存入你的酒譜庫。

---

## 📖 酒譜收藏

- **手動新增**：填入名稱、材料（每行一種）、製作步驟、備注
- **從 AI 儲存**：AI 推薦後一鍵收藏，自動帶入類型標記
- **星級評分**：為每款酒評分 ★ 到 ★★★★★
- **摺疊卡片**：點標題展開查看，介面清爽不雜亂
- **類型標記**：🥂 經典 / 🎨 創意 / ✍️ 自創

---

## 🛠️ 自行部署 Cloudflare Worker

若你想使用自己的 AI API，可以自行部署 Worker：

```js
// worker.js 範例
export default {
  async fetch(request) {
    if (request.method !== 'POST') return new Response('Method Not Allowed', { status: 405 });

    const { system, prompt } = await request.json();

    const res = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'x-api-key': YOUR_ANTHROPIC_API_KEY,
        'anthropic-version': '2023-06-01',
        'content-type': 'application/json',
      },
      body: JSON.stringify({
        model: 'claude-opus-4-5',
        max_tokens: 1024,
        system,
        messages: [{ role: 'user', content: prompt }],
      }),
    });

    const data = await res.json();
    const text = data.content?.[0]?.text || '';
    return new Response(JSON.stringify({ text }), {
      headers: { 'Content-Type': 'application/json', 'Access-Control-Allow-Origin': '*' },
    });
  }
};
```

部署後，將 HTML 中的 `WORKER_URL` 替換為你自己的 Worker 網址。

---

## 📁 資料結構

Gist 中儲存的 JSON 格式如下：

```json
{
  "inventory": [
    { "id": 1234567890, "name": "Tanqueray Gin", "cat": "烈酒", "amt": "充足" }
  ],
  "recipes": [
    {
      "id": 1234567891,
      "name": "Negroni",
      "type": "classic",
      "ingredients": ["Gin 45ml", "Sweet Vermouth 25ml", "Campari 25ml"],
      "steps": ["將所有材料加冰塊放入調酒杯", "攪拌 30 秒", "濾入已冰鎮的杯子"],
      "note": "苦甜平衡，植物香氣豐富",
      "rating": 5,
      "savedAt": "2025-01-01T00:00:00.000Z"
    }
  ],
  "exportedAt": "2025-01-01T00:00:00.000Z"
}
```

---

## 🍷 支援的材料分類

`烈酒` `利口酒` `葡萄酒` `啤酒` `果汁` `糖漿` `苦精` `氣泡水` `香料` `其他`

---

## 📄 License

MIT — 隨意使用、修改、分享。乾杯 🥂
