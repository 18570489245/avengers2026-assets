# 🎆 Agnes API 完整使用指南 — 跨机器部署版

> **适用**：另一台 Hermes 也用同样的方式调用 Agnes
> **测试通过**：2026-07-11 你给的 key 完全可用
> **关键发现**：Agnes 必须用 `video_id` 查询（不是 task_id），某些模型名不对会 400

---

## 1. 三个 Key（顺序重要）

```
# Key 0：主力（负责 img2img + 视频派发）
sk-VBSU7KzWmAd7ViGIEcl9OclTCwGCEjnF02D6Z8QqgtoVzbtW

# Key 1、Key 2：你原本的 3 个 key（上一台 Hermes 里的）
# 如果其他机器用不了上面那个 key，就用原来的 3 个 key round-robin
```

---

## 2. 正确的模型名（必对）

| 用途 | 模型名 | 错的（会 400） |
|------|--------|---------------|
| 视频生成 | `agnes-video-v2.0` | `agnes-video` / `v2.0` / `video-v2` |
| 图生图 | `agnes-image-2.1-flash` | `agnes-image` / `image-2.1` |

---

## 3. 请求格式（完整版）

### 3.1 视频派发

```bash
curl -s --noproxy "*" -X POST \
  "https://apihub.agnes-ai.com/v1/videos" \
  -H "Authorization: Bearer sk-VBSU7KzWmAd7ViGIEcl9OclTCwGCEjnF02D6Z8QqgtoVzbtW" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-video-v2.0",
    "prompt": "9:16 vertical video full body visible. Navy suit red tie. Original subject on wet ground. Action: pushes arms up struggles to stand. Camera: medium low angle. Lighting: cool night rain. Audio: rain intensifies. Constraints: full body visible do not alter face identity no text no logo.",
    "width": 720,
    "height": 1280,
    "num_frames": 145,
    "frame_rate": 24,
    "seed": 42,
    "stream": false,
    "extra_body": {
      "mode": "keyframes",
      "image": [
        "https://cdn.jsdelivr.net/gh/18570489245/avengers2026-assets@main/avengers2026/AV-KF-01.jpg",
        "https://cdn.jsdelivr.net/gh/18570489245/avengers2026-assets@main/avengers2026/seg4_v4-tail.jpg"
      ]
    }
  }'
```

**返回**：
```json
{
  "video_id": "task_xxx",
  "status": "queued",
  "task_id": "task_xxx"
}
```

### 3.2 查询进度（两种方式都可用）

```bash
# 方式 1：用 video_id（推荐）
curl -s --noproxy "*" \
  -H "Authorization: Bearer sk-VBSU7KzWmAd7ViGIEcl9OclTCwGCEjnF02D6Z8QqgtoVzbtW" \
  "https://apihub.agnes-ai.com/agnesapi?video_id=task_xxx"

# 方式 2：用 task_id（兼容）
curl -s --noproxy "*" \
  -H "Authorization: Bearer sk-VBSU7KzWmAd7ViGIEcl9OclTCwGCEjnF02D6Z8QqgtoVzbtW" \
  "https://apihub.agnes-ai.com/v1/videos/task_xxx"
```

**返回**：
```json
{
  "status": "completed",
  "url": "https://xxx.mp4",
  "video_id": "task_xxx",
  "size": "704x1280",
  "seconds": 6.04
}
```

### 3.3 图生图（生关键帧）

```bash
curl -s --noproxy "*" -X POST \
  "https://apihub.agnes-ai.com/v1/images/generations" \
  -H "Authorization: Bearer sk-VBSU7KzWmAd7ViGIEcl9OclTCwGCEjnF02D6Z8QqgtoVzbtW" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-image-2.1-flash",
    "prompt": "9:16 vertical portrait full body visible. Young Asian man navy suit red tie. On wet ground looking up. Night rain city. NO text.",
    "size": "768x1024",
    "extra_body": {
      "image": ["https://cdn.jsdelivr.net/gh/18570489245/avengers2026-assets@main/avengers2026/AV-KF-01.jpg"],
      "response_format": "url"
    }
  }'
```

**返回**：
```json
{
  "data": [{"url": "https://platform-outputs.agnes-ai.space/images/xxx.png"}]
}
```

---

## 4. 限流与轮询

### 4.1 单 Key 限流规则

| 规则 | 说明 |
|------|------|
| 同一 key 调用间隔 | ≥ 60s |
| 推荐派发间隔 | ≥ 180s（3 分钟）|

### 4.2 Round-Robin 轮询（推荐）

```python
KEYS = ['sk-KEY0', 'sk-KEY1', 'sk-KEY2']
key_order = [0, 1, 2, 0, 1]

for i, task in enumerate(tasks):
    dispatch_video(task, key_idx=key_order[i])
    if i < len(tasks) - 1:
        time.sleep(180)
```

---

## 5. 另一台 Hermes 用不了这个 Key 的原因排查

| 现象 | 原因 | 解决 |
|------|------|------|
| HTTP 401 | Key 错/过期 | 复制粘贴时漏字符 |
| HTTP 400 | 模型名错 | 用 `agnes-video-v2.0` |
| HTTP 400 | 不能传 width/height/frame_rate | 这是旧文档的坑 |
| HTTP 429 | 限流 | 加 3 分钟间隔 |
| HTTP 503 | 服务暂不可用 | 重试（代码里要 backoff）|
| 横屏输出 | width/height 没传或传反 | 720×1280 |
| vision_analyze 失败 | 本地路径不行 | 用 CDN URL |

---

## 6. 快速测试脚本（复制即运行）

```python
import urllib.request, json

KEY = 'sk-VBSU7KzWmAd7ViGIEcl9OclTCwGCEjnF02D6Z8QqgtoVzbtW'
VID_URL = 'https://apihub.agnes-ai.com/v1/videos'
IMG_URL = 'https://apihub.agnes-ai.com/v1/images/generations'

# 测试 1：生图
payload = {'model':'agnes-image-2.1-flash','prompt':'red circle on white','size':'512x512','return_base64':True}
req = urllib.request.Request(IMG_URL, data=json.dumps(payload).encode(), method='POST')
req.add_header('Authorization', f'Bearer {KEY}')
req.add_header('Content-Type', 'application/json')
with urllib.request.urlopen(req, timeout=120) as r:
    resp = json.loads(r.read())
print('生图:', 'OK' if resp.get('data') else f'FAIL: {resp}')

# 测试 2：视频
payload = {'model':'agnes-video-v2.0','prompt':'9:16 video test','width':720,'height':1280,'num_frames':49,'frame_rate':24,'seed':42,'stream':False}
req = urllib.request.Request(VID_URL, data=json.dumps(payload).encode(), method='POST')
req.add_header('Authorization', f'Bearer {KEY}')
req.add_header('Content-Type', 'application/json')
with urllib.request.urlopen(req, timeout=120) as r:
    resp = json.loads(r.read())
print('视频:', f"OK v={resp.get('video_id')}" if resp.get('video_id') else f"FAIL: {resp}")
```

---

## 7. 给另一台 Hermes 的完整 setup

1. **在 `.env` 里加**：
```
AGNES_API_KEY_0=sk-VBSU7KzWmAd7ViGIEcl9OclTCwGCEjnF02D6Z8QqgtoVzbtW
```

2. **或者在代码里直接用**：
```python
from agnes_pool import AgnesPool
pool = AgnesPool()
# 修改 agnes_pool.py 里的 KEYS 列表，把你新 key 放进去
```

3. **调用方式**：
```python
pool = AgnesPool()
result = pool.dispatch({
    'model': 'agnes-video-v2.0',
    'prompt': '...',
    'width': 720, 'height': 1280,
    'num_frames': 145, 'frame_rate': 24,
    'extra_body': {'mode': 'keyframes', 'image': [first_url, tail_url]}
})
```

---

## 8. 关键经验（踩过的坑）

| ❌ 错误 | ✅ 正确 |
|---------|---------|
| `GET /v1/videos/{video_id}` | `GET /agnesapi?video_id={video_id}` |
| `GET /agnesapi?task_id=xxx` | `GET /agnesapi?video_id=xxx` |
| `prompt` 里只写 "9:16 video..." | 用完整 Seedance 导演公式 |
| 尾帧传到首帧位置做参考 | 首帧永远用 AV-KF-01.jpg，尾帧用 img2img 生成 |
| 每段连续派发 | round-robin + sleep 3min |
| 用 jsdelivr 做 video 输入 | 用 raw.githubusercontent.com |

---

## 9. 参考

- [Agnes Video V2.0 文档](https://www.agnes-ai.com/zh-Hans/docs/agnes-video-v20.md)
- [Seedance Prompt 方法论](https://github.com/Emily2040/seedance-2.0/tree/main/skills/seedance-prompt)
```