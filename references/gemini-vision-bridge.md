# Gemini Vision Bridge 脚本模板

纯 Python stdlib，零依赖。保存为 `gemini_vision_mcp.py`。

```python
#!/usr/bin/env python3
"""Gemini Vision 图片识别桥接 — 纯 Python stdlib，零额外依赖"""
import json, sys, base64, urllib.request, os

API_KEY = os.environ.get("GEMINI_API_KEY", "").strip()
if not API_KEY:
    key_file = os.path.join(os.path.dirname(os.path.abspath(__file__)), ".gemini_key")
    try:
        with open(key_file) as f:
            API_KEY = f.read().strip()
    except: pass

MODEL = "gemini-2.5-flash"
API_URL = f"https://generativelanguage.googleapis.com/v1beta/models/{MODEL}:generateContent"

def analyze_image(file_path, prompt):
    if not API_KEY:
        return "错误：未配置 Gemini API Key。"
    with open(file_path, "rb") as f:
        img_b64 = base64.b64encode(f.read()).decode()
    payload = {
        "contents": [{"parts": [
            {"inline_data": {"mime_type": "image/jpeg", "data": img_b64}},
            {"text": prompt}
        ]}],
        "safetySettings": [
            {"category": "HARM_CATEGORY_HARASSMENT", "threshold": "BLOCK_NONE"},
            {"category": "HARM_CATEGORY_HATE_SPEECH", "threshold": "BLOCK_NONE"},
            {"category": "HARM_CATEGORY_SEXUALLY_EXPLICIT", "threshold": "BLOCK_NONE"},
            {"category": "HARM_CATEGORY_DANGEROUS_CONTENT", "threshold": "BLOCK_NONE"},
        ]
    }
    req = urllib.request.Request(
        f"{API_URL}?key={API_KEY}",
        data=json.dumps(payload).encode("utf-8"),
        headers={"Content-Type": "application/json"}
    )
    resp = json.loads(urllib.request.urlopen(req, timeout=60).read())
    if "candidates" in resp and resp["candidates"]:
        return resp["candidates"][0]["content"]["parts"][0]["text"]
    return f"未知响应：{json.dumps(resp, ensure_ascii=False)[:500]}"

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("用法: python gemini_vision_mcp.py <图片路径> [提示词]")
        sys.exit(1)
    prompt = sys.argv[2] if len(sys.argv) > 2 else "请详细描述这张图片。"
    print(analyze_image(sys.argv[1], prompt))
```

## API Key 获取

1. 打开 https://aistudio.google.com/apikey
2. 点击 "Create API Key"
3. 保存 Key：`echo "你的Key" > .gemini_key`（与脚本同目录）
4. 或设置环境变量：`export GEMINI_API_KEY="你的Key"`

## 免费额度

- 10 次/分钟，~1500 次/天
- 模型：gemini-2.5-flash（免费层）
- 日常食物分析绰绰有余
