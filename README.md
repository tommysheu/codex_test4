# KG Chatbot Demo

## Run

```bash
pip install -r requirements.txt
export OPENAI_API_KEY=...
uvicorn main:app --reload
```

Swagger: `/docs`

## Knowledge Graph 建好後放在哪裡？

- 記憶體中的 `kg` 物件（程式執行期使用）。
- 另外會落地一份快照檔：`data/kg_snapshot.json`（方便檢查與除錯）。

## 如何測試回答時「真的有查找 KG」？

1. 先打 `GET /api/kg/status`
   - 看 `is_built=true`
   - 看 `node_count / edge_count`
   - 看 `contains_zheng_chonghua`
   - 看 `snapshot_exists=true`
2. 再打 `GET /api/kg/query?q=鄭崇華`
   - 檢查是否有回傳命中節點（title/url/source）
3. 用 `POST /api/chat/stream` 問台達相關問題
   - SSE 事件會先回 `token`
   - 最後會回 `trace`（含 `used_kg` 與命中節點）
   - 若 `used_kg=true`，再回 `kg_result`

這樣可完整驗證：KG 有建、能查、且聊天時有實際使用。
