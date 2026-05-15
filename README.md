# KG Chatbot Demo

## Run

```bash
pip install -r requirements.txt
export OPENAI_API_KEY=...
uvicorn main:app --reload
```

Swagger: `/docs`

## Knowledge Graph 是怎麼建立的？

- 種子來源：
  - 維基百科台達電子：`https://zh.wikipedia.org/zh-tw/台達電子`
  - 台達官網：`https://www.deltaww.com/zh-TW/index`
- 建圖流程：
  1. 下載 seed 頁面，抽取文字與同網域連結。
  2. 連結做一階展開（只抓 seed 連出的第一層頁面）。
  3. 轉成節點（node）與邊（edge）。
  4. 存在記憶體 `kg`，並落地到 `data/kg_snapshot.json`。
- 維基頁面連結會優先保留條目型連結（`/wiki/...`、`/zh-tw/...`），降低導覽連結干擾，提升抓到「台達電子相關條目、鄭崇華」的機率。

## KG 何時建立？

- **FastAPI 啟動時會先建一次**（`startup` event）。
- 若啟動時失敗，後續在呼叫以下 API 時也會再確保建置：
  - `GET /api/kg/status`
  - `GET /api/kg/query`
  - `POST /api/chat/stream`

## Knowledge Graph 建好後放在哪裡？

- 記憶體中的 `kg` 物件（程式執行期使用）。
- 快照檔：`data/kg_snapshot.json`（可直接開檔檢查）。

## 如何測試回答時「真的有查找 KG」？

1. `GET /api/kg/status`
   - 看 `is_built=true`
   - 看 `node_count / edge_count`
   - 看 `source_node_counts.wikipedia` 是否 > 0（確認維基節點有進圖）
   - 看 `contains_zheng_chonghua`
   - 看 `snapshot_exists=true`
2. `GET /api/kg/query?q=鄭崇華`
   - 檢查回傳命中節點（title/url/source）
3. `POST /api/chat/stream` 問台達相關問題
   - SSE 先回 `token`
   - 之後回 `trace`（含 `used_kg` 與命中節點）
   - 若 `used_kg=true`，再回 `kg_result`

這樣可完整驗證：KG 有建、含維基/官網、能查、且聊天時有實際使用。
