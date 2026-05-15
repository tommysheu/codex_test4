# KG Chatbot Demo

## Run

```bash
pip install -r requirements.txt
export OPENAI_API_KEY=...
uvicorn main:app --reload
```

Swagger: `/docs`

## Verify KG

- GET /api/kg/status 可確認是否已建構、節點/邊數量、是否包含鄭崇華條目。
