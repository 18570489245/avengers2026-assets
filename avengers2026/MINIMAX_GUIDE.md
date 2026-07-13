# MiniMax 生图/视觉 API 使用指南 — 跨机器部署版

> **适用**：另一台 Hermes 也用同样的方式调用 MiniMax
> **验证**：2026-07-11 实际测试通过

---

## 1. Key 管理

### 1.1 你的 Key（在中国国内使用）

```
sk-cp-...PPLQ
```

**配置方式（D:\hermes\.env）**：

```env
# MiniMax China endpoint (for users in mainland China)
# Get your key at: https://www.minimax.chat
MINIMAX_CN_API_KEY=sk-cp-...PPLQ
MINIMAX_CN_BASE_URL=https://api.minimaxi.com/anthropic
```

### 1.2 Key 分配建议

| 用途 | 变量名 | 备注 |
|------|--------|------|
| TTS 配音 | `MINIMAX_CN_API_KEY` | 用于 t2a_v2 端点 |
| 视觉分析 | `MINIMAX_CN_API_KEY` | 用于 abab6.5s-chat 模型 |
| 图片生成 | `MINIMAX_CN_API_KEY` | 用于文生图 |

---

## 2. TTS 配音端点

### 2.1 请求格式

```
POST https://api.minimaxi.com/v1/t2a_v2
Authorization: Bearer <MINIMAX_CN_API_KEY>
Content-Type: application/json
```

### 2.2 请求体（完整）

```json
{
  "text": "他曾是托尼·史塔克，神盾局最年轻的CEO",
  "model": "speech-02-hd",
  "voice_setting": {
    "voice_id": "audiobook_male_1",
    "speed": 1.0,
    "vol": 1.0,
    "pitch": 0,
    "emotion": "sad"
  },
  "audio_setting": {
    "sample_rate": 44100,
    "format": "mp3",
    "channels": 2
  }
}
```

### 2.3 音色名称对照表（主角音色）

| 角色 | voice_id | 说明 |
|------|----------|------|
| 主角Tony | male-qn-badao | 锁 pitch=0，低沉男声 |
| 旁白 | audiobook_male_1 | 旁白男声 |
| 接待员 | female-chengshu | 前台女声 |
| 客户/女程序员 | female-yujie | 女程序员声 |
| 鹰眼 | presenter_male | 旁白/画外音 |

### 2.4 情绪列表

`surprised` / `sad` / `angry` / `disgusted` / `happy` / `fearful` / `neutral`

### 2.5 关键参数

| 参数 | 推荐值 | 说明 |
|------|--------|------|
| model | speech-02-hd | 高质量语音 |
| speed | 1.0 | 语速（0.5-2.0）|
| vol | 1.0 | 音量 |
| pitch | 0 | 音调（锁0，不可改）|
| sample_rate | 44100 | 必须 44100Hz |
| format | mp3 | 或 wav |
| channels | 2 | 立体声（必须2）|

### 2.6 Python 调用代码

```python
import urllib.request, json

MINIMAX_KEY = 'sk-cp-...PPLQ'  # 你的 key
TTS_URL = 'https://api.minimaxi.com/v1/t2a_v2'

def tts(text, voice_id, emotion='neutral', speed=1.0):
    payload = {
        'text': text,
        'model': 'speech-02-hd',
        'voice_setting': {
            'voice_id': voice_id,
            'speed': speed,
            'vol': 1.0,
            'pitch': 0,
            'emotion': emotion
        },
        'audio_setting': {
            'sample_rate': 44100,
            'format': 'mp3',
            'channels': 2
        }
    }
    req = urllib.request.Request(TTS_URL, data=json.dumps(payload).encode(), method='POST')
    req.add_header('Authorization', f'Bearer {MINIMAX_KEY}')
    req.add_header('Content-Type', 'application/json')
    with urllib.request.urlopen(req, timeout=120) as r:
        resp = json.loads(r.read())
    return resp  # 返回 audio_hex 或 url

# 使用示例
result = tts('他曾是托尼·史塔克', 'audiobook_male_1', emotion='sad')
audio_data = bytes.fromhex(result['data']['audio'])
with open('line_01.mp3', 'wb') as f:
    f.write(audio_data)
```

### 2.7 cURL 测试

```bash
curl -s --noproxy "*" \
  -X POST "https://api.minimaxi.com/v1/t2a_v2" \
  -H "Authorization: Bearer <REDACTED>" \
  -H "Content-Type: application/json" \
  -d '{"text":"测试语音","model":"speech-02-hd","voice_setting":{"voice_id":"male-qn-badao","speed":1,"vol":1,"pitch":0},"audio_setting":{"sample_rate":44100,"format":"mp3","channels":2}}'
```

---

## 3. 视觉分析（Image Understanding）

### 3.1 请求格式

```
POST https://api.minimaxi.com/v1/chat/completions
Authorization: Bearer <MINIMAX_CN_API_KEY>
Content-Type: application/json
```

### 3.2 请求体（多模态）

```json
{
  "model": "abab6.5s-chat",
  "messages": [
    {"role": "user", "content": [
      {"type": "image_url", "image_url": {"url": "data:image/jpeg;base64,...."}},
      {"type": "text", "text": "分析这张图片"}
    ]}
  ]
}
```

### 3.3 图片输入方式

| 方式 | 说明 |
|------|------|
| Base64 内嵌 | `data:image/jpeg;base64,...` |
| URL 直传 | `https://...cdn.../image.jpg` |
| 最大文件 | 20MB |

### 3.4 Python 调用代码

```python
import urllib.request, json, base64

MINIMAX_KEY = 'sk-cp-...PPLQ'
VL_URL = 'https://api.minimaxi.com/v1/chat/completions'

def vision_analyze(image_path, question):
    with open(image_path, 'rb') as f:
        img_b64 = base64.b64encode(f.read()).decode()
    
    payload = {
        'model': 'abab6.5s-chat',
        'messages': [{
            'role': 'user',
            'content': [
                {'type': 'image_url', 'image_url': {'url': f'data:image/jpeg;base64,{img_b64}'}},
                {'type': 'text', 'text': question}
            ]
        }]
    }
    req = urllib.request.Request(VL_URL, data=json.dumps(payload).encode(), method='POST')
    req.add_header('Authorization', f'Bearer {MINIMAX_KEY}')
    req.add_header('Content-Type', 'application/json')
    with urllib.request.urlopen(req, timeout=120) as r:
        return json.loads(r.read())

# 使用
result = vision_analyze('qc/V01/check.jpg', '画面内容？人物脸清晰？')
print(result)
```

### 3.5 返回格式

```json
{
  "choices": [{"message": {"content": "这是一张..."}}],
  "usage": {"total_tokens": 1234}
}
```

---

## 4. 常见问题

| 问题 | 原因 | 解决 |
|------|------|------|
| 语音返回空 | key 多空格/tab | 调用前 `.strip()` |
| 国际版报错 401 | 用了 global endpoint | 用 `api.minimaxi.com`（国内）或 `api.minimax.io`（国际）|
| 图片分析 400 | 图片 URL 不可访问 | 用 base64 内嵌或 raw.githubusercontent |
| 生成失败 | 模型名错 | TTS=`speech-02-hd`, 视觉=`abab6.5s-chat` |

---

## 5. 给另一台 Hermes 完整 .env 配置块

```env
# ===== MiniMax China =====
MINIMAX_CN_API_KEY=sk-cp-...PPLQ
MINIMAX_CN_BASE_URL=https://api.minimaxi.com/anthropic

# 备用国际版（如果国内不可用）
# MINIMAX_API_KEY=<国际版key>
# MINIMAX_BASE_URL=https://api.minimax.io/v1
```

---

## 6. 快速测试脚本（复制即用）

```python
import urllib.request, json
KEY = 'sk-cp-...PPLQ'

# TTS 测试
r = urllib.request.urlopen(urllib.request.Request(
    'https://api.minimaxi.com/v1/t2a_v2',
    data=json.dumps({
        'text': '语音测试',
        'model': 'speech-02-hd',
        'voice_setting': {'voice_id': 'audiobook_male_1', 'speed': 1}
    }).encode(),
    headers={'Authorization': f'Bearer {KEY}', 'Content-Type': 'application/json'}
), timeout=30)
print(json.loads(r.read()))
```
