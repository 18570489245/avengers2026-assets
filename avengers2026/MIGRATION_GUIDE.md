# 🎬 复仇者联盟2026 — AI短剧生产系统 完整迁移手册

> **目标**：让另一台 Hermes 实例完整接手 EP01 生产，达到与当前实例同等熟悉程度。
> **更新日期**：2026-07-11
> **兼容 Hermes**：v2026+ (OpenAI browser backend)

---

## 1. 项目全局

```
D:\复仇者联盟2026-漫剧\
├── 00-参考素材\        ← 主角脸 ref 图、关键帧原图
├── 01-剧本与分镜\
│   └── EP01_完整分镜.md   (278行完整分镜：V01-V20)
├── 02-关键帧\          ← 上传用 KF (含 AV-KF-01.jpg 等)
├── 03-视频段\
│   ├── raw\            ← 原始生成视频 (旧版+新版混存)
│   └── normalized\     ← 60fps 修复后视频 (待建)
├── 04-TTS\
│   └── lines\          ← 22句 mp3 (已生成)
├── 05-BGM\
│   └── 励志奋斗风.mp3   ← 已选定 BGM
├── 06-后期模板\
├── 09-夜审\             ← 用户审核记录
└── D:\tmp\            ← 脚本目录 (派发/合并脚本)
```

---

## 2. 硬件与存储规则

| 规则 | 说明 |
|------|------|
| **C盘只剩 452MB** | 禁止写任何数据到 C: |
| **D盘为工作盘** | 所有视频/图片/脚本都在 `D:\` |
| **Hermes 迁移** | `D:\hermes` 真实路径，`C:\Users\Jack\AppData\Local\hermes` 是 junction |
| **环境变量** | MINIMAX_CN_API_KEY / GH_TOKEN 在 `D:\hermes\.env` |
| **github CDN** | `curl -sk --noproxy "*"` 绕过代理直连（jsdelivr CDN 可用） |

---

## 3. EP01 分镜时间线（114s 全片）

| 段 | 时间 | 时长 | 画面 | 台词 |
|---|------|------|------|------|
| V01 | 0-7.77s | 7.77s | 办公室转身 → 紫光爆脸 → 跌坐 | 前CEO/刚送外卖/想回家 |
| V02a | 7.77-10.74s | 2.97s | 外卖小哥路过无视 | - |
| V02b | 10.74-13.71s | 2.97s | 二次撞车倒地 | - |
| V02c | 13.71-16.68s | 2.97s | 起身直视握拳 | - |
| V03 | 16.68-23.65s | 6.97s | 雨中独白 | 拍vlog/打工人/100万赞 |
| V04 | 23.65-28.69s | 5.04s | 任务卡弹出 | 神盾局通知 |
| V05 | 28.69-33.73s | 5.04s | OS反应 | 拍vlog？我？ |
| V06a | 33.73-36.70s | 2.97s | 西装进写字楼 | 工牌 |
| V06b | 36.70-39.70s | 3.0s | 红酒进电梯 | 不能带酒 |
| V06c | 39.70-42.70s | 3.0s | 保姆装擦桌子 | 搞笑 |
| V07 | 42.70-44.58s | 1.88s | 手机3个失败通知 | - |
| V08 | 44.58-49.62s | 5.04s | 外卖工服骑电动车 | 我不装了 |
| V09 | 49.62-54.66s | 5.04s | 5城快剪 | 特种兵citywalk |
| V10 | 54.66-59.70s | 5.04s | 送螺蛳粉客户对话 | 你也加班/如何呢 |
| V11 | 59.70-62.25s | 2.55s | 雨中独白 | 我们都是打工人 |
| V12 | 62.25-67.29s | 5.04s | 外滩夜景 | 薪资基础... |
| V13 | 67.29-72.33s | 5.04s | 头盔内刻字 | 托尼的心 |
| V14 | 72.33-77.37s | 5.04s | 摘头盔金句 | 爱你老己 |
| V15 | 77.37-82.41s | 5.04s | 数据飙升 | ❤️23.4万 |
| V16 | 82.41-87.45s | 5.04s | 时空裂缝 | 小辣椒 |
| V17 | 87.45-92.49s | 5.04s | 鹰眼语音 | 还差5个人 |
| V18-19 | 92.49-97.53s | 5.04s | 六画面快闪 | 我一个人演完 |
| V20 | 97.53-100.50s | 2.97s | 黑屏金句 | 爱你老己/第2集 |

---

## 4. 当前素材状态（截至 2026-07-11 02:35）

### ✅ 已完成视频（raw/ 目录）

| 文件 | 说明 | 分辨率 | 帧率 | 来源 |
|------|------|--------|------|------|
| `seg1_720x1280.mp4` | V01第1段 | 704×1280 | 24fps | 旧版提示词 |
| `seg2_720x1280.mp4` | V01第2段 | 704×1280 | 24fps | 旧版提示词 |
| `seg3_720x1280.mp4` | V01第3段 | 704×1280 | 24fps | 旧版提示词 |
| `seg4_720x1280.mp4` | V01第4段 | 704×1280 | 24fps | 旧版提示词 |
| `seg1_v2_seedance.mp4` | V01 Seedance重做 | 704×1280 | 24fps | **Seedance方法论** ✅ |
| `seg2_v2_seedance.mp4` | V01 Seedance重做 | 704×1280 | 24fps | **Seedance方法论** ✅ |
| `seg3_v2_seedance.mp4` | V01 Seedance重做 | 704×1280 | 24fps | **Seedance方法论** ✅ |
| `seg4_v2_seedance.mp4` | V01 Seedance重做 | 704×1280 | 24fps | **Seedance方法论** ✅ |
| `v02a_720x1280.mp4` | V02a竖屏 | 704×1280 | 24fps | 旧版提示词 |
| `V02b_720x1280.mp4` | V02b竖屏 | 704×1280 | 24fps | 顺序派发守护 |
| `seg1_kf_seedance.mp4` | V1 **关键帧版** 1段 | 704×1280 | 24fps | **KF+Seedance** ✅ |
| `seg2_kf_seedance.mp4` | V1 **关键帧版** 2段 | 704×1280 | 24fps | **KF+Seedance** ✅ |

### ❌ 待生成/待重做（v02c-v20 / TTS 合成 / 字幕）

V02c / V03 / V04 / V05 / V06a / V06b / V06c / V07 / V08 / V09 / V10 / V11 / V12 / V13 / V14 / V15 / V16 / V17 / V18-19 / V20

---

## 5. 核心工作流（铁律）

### 5.1 视频生成流程

```
Step 1: 抽取上段尾帧 (上段视频尾帧 → CDN)
        ffmpeg -y -sseof -1 -i V01.mp4 -frames:v 1 V01_last.jpg
        curl -sk --noproxy "*" -X PUT ... → GitHub CDN

Step 2: 用用户主角脸 ref 图 + 尾帧 → img2img 生成新关键帧
        ref 图始终用: https://cdn.jsdelivr.net/gh/18570489245/avengers2026-assets@main/avengers2026/ref-v01-frame.jpg

Step 3: 派发视频 (keyframes 模式)
        Agnes POST /v1/videos
        {"model":"agnes-video-v2.0","prompt":"...","width":720,"height":1280,
         "num_frames":121,"frame_rate":24,"extra_body":{"mode":"keyframes",
         "image":[LAST_CDN_URL, NEW_CDN_URL]}}

        ⚠️ 关键尺寸：width=720, height=1280（输出704×1280）
        ❌ width=1080 height=1920 会被规范成横屏！

Step 4: 轮询任务 → 下载视频
        GET /agnesapi?video_id=<XXX>
        → status=completed → url 字段 → 下载 mp4

Step 5: 抽取新尾帧 → 循环到 Step 1
```

### 5.2 视频转码（24fps → 60fps）

```bash
ffmpeg -y -i input_24fps.mp4 -vf "fps=60" -c:v libx264 -crf 18 -pix_fmt yuv420p output_60fps.mp4
```

### 5.3 多段拼接

```bash
# 制作 list.txt
echo "file 'seg1.mp4'" > list.txt
echo "file 'seg2.mp4'" >> list.txt
echo "file 'seg3.mp4'" >> list.txt

# 拼接 (重新编码保证一致性)
ffmpeg -y -f concat -safe 0 -i list.txt -c:v libx264 -crf 18 -pix_fmt yuv420p -r 60 -movflags +faststart output.mp4
```

### 5.4 CDN 上传

```bash
python -c "
import json, base64
from pathlib import Path
img = Path('D:/path/to/image.jpg').read_bytes()
b64 = base64.b64encode(img).decode()
Path('D:/tmp/body.json').write_text(json.dumps({'message':'upload','content':b64}))
"
curl -sk --noproxy "*" -X PUT \
  "https://api.github.com/repos/18570489245/avengers2026-assets/contents/avengers2026/FILENAME.jpg" \
  -H "Authorization: token $GH_TOKEN" \
  -H "Content-Type: application/json" \
  -d @D:/tmp/body.json
```

CDN URL 格式：`https://cdn.jsdelivr.net/gh/18570489245/avengers2026-assets@main/avengers2026/FILENAME`

---

## 6. 用户工作习惯（Jack）

| 习惯 | 说明 |
|------|------|
| **"进度"** | 汇报当前完成情况 |
| **"发给我看看"** | 上传 CDN 后发 MEDIA: 链接 |
| **"可以"** | 确认通过 |
| **"不对"** | 需要重新做 |
| **"要先给我审核 不行还要换"** | 视频/关键帧必须先发给用户审 |
| **微信语音输入** | 错字是正常语音识别误写，不纠正 |
| **每 3-5 分钟推一次进度** | 不能沉默超过 5 分钟 |
| **"先出效果再确认"** | 先做出来让用户看到 |
| **禁止慢倍速延长视频** | 宁可重新生成视频，也不能慢放 |

---

## 7. 项目特定规则

| 规则 | 说明 |
|------|------|
| **视频不加字幕** | 中文文字=质量杀手，后期用 FFmpeg drawtext 叠加 |
| **BGM** | 已选定励志奋斗风，vol≥0.55 |
| **TTS 配音** | 22句 mp3 已生成在 `04-TTS/lines/` |
| **主角脸** | 始终用 `ref-v01-frame.jpg` 作为 reference_image |
| **1人6角** | Jack 亲自演全部角色（AI 性转女角） |
| **抖音目标** | 100万赞，数据驱动，严苛评审 |

---

## 8. MiniMax TTS 配置

```
Endpoint: https://api.minimaxi.com/v1/t2a_v2
Key:      MINIMAX_CN_API_KEY (D:\hermes\.env, 需要 .strip())

音色分配:
  旁白     = audiobook_male_1
  主角     = male-qn-badao (pitch锁定0)
  接待员   = female-chengshu
  客户     = female-yujie
  鹰眼     = presenter_male
```

---

## 9. Seedance 视频提示词方法论

**来源**：Emily2040/seedance-2.0 v6.6.0

### 9.1 导演公式

```
Subject + Action + Scene + Camera + Lighting/Style + Audio + Constraints
```

### 9.2 模式门

| 模式 | 规则 |
|------|------|
| **T2V** | 完整描写 7 要素 |
| **I2V** | 从 @Image1 开始，只加动作/镜头/时长/音频/保留约束 |
| **FLF2V** | @Image1→@Image2，只描述过渡过程 |

### 9.3 反垃圾词汇

```
❌ 空洞词: highly detailed, 8k, masterpiece, best quality
✅ 具体器材: Sony A7S3, ARRI Alexa, iPhone 15 Pro, 35mm film
```

### 9.4 保留约束铁律

```
必须写: NO change + 具体约束
例: "do not alter face identity, suit color, or glasses"
```

### 9.5 完整提示词示例

```
9:16 vertical video, full body visible. Young Asian man navy suit red tie.
Original subject facing camera. Action: purple energy blasts face, body flies
backward into electric scooter. Camera: medium shot, slow push-in.
Lighting: purple energy flash rim light, dark ambient. Audio: bass boom, metal
crash. Constraints: full body visible, do not alter face identity, suit color,
or glasses.
```

---

## 10. Agnes API 速查

| 参数 | 必填 | 说明 |
|------|------|------|
| `model` | ✅ | `agnes-video-v2.0` |
| `prompt` | ✅ | 视频提示词 |
| `width` | 否 | 竖屏用 **720**（❌用1080变横屏） |
| `height` | 否 | 竖屏用 **1280**（❌用1920变横屏） |
| `num_frames` | 否 | ≤441 且 8n+1 |
| `frame_rate` | 否 | 24或30或60 |
| `extra_body.image` | 关键帧模式 | [首帧URL, 尾帧URL] 数组 |
| `extra_body.mode` | 关键帧模式 | `keyframes` |

### 轮询查询（推荐）
```
GET https://apihub.agnes-ai.com/agnesapi?video_id=<VIDEO_ID>
Authorization: Bearer <KEY>
```

### 兼容旧版查询
```
GET https://apihub.agnes-ai.com/v1/videos/<TASK_ID>
Authorization: Bearer <KEY>
```

### 返回字段
- `status`: queued / in_progress / completed / failed
- `url`: 视频下载链接（仅 completed）
- `size`: 如 "704x1280"
- `seconds`: 视频时长

---

## 11. 派送模式选择

| 模式 | 脚本 | 适用 |
|------|------|------|
| **顺序单 key 派发** | `dispatch_720x1280.py` | 单 key，每段 sleep 3min |
| **多 key 3-min 间隔** | `dispatch_multi_key.py` | 3 key 并行，间隔 3min |
| **用户审核版** | 手动逐段 | 每段审 KF → 审视频 → 确认 |

### 推荐并发脚本结构
```python
key_order = [0, 1, 2, 0, 1]  # round-robin
for i, task in enumerate(tasks):
    dispatch(task, key_idx=key_order[i])
    if i < len(tasks) - 1:
        time.sleep(180)  # 3 minutes between dispatches
```

---

## 12. TTS 时长匹配铁律

| 规则 | 说明 |
|------|------|
| **先测 TTS 总长** | 派发生成前，实测所有 mp3 的时长之和 |
| **TTS > 视频时长** | 必须重新生成视频更长，**禁止慢倍速** |
| **TTS ≤ 8s** | Agnes 1080p ≈ 241帧@30fps 上限 |
| **TTS > 8s** | 拆分为多个子段（V01a/V01b/V01c 模式） |

---

## 13. 守护进程管理

| 守护 | 任务 | 状态 |
|------|------|------|
| `proc_8d8a047a872d` | V02b→V20 顺序派发（旧版提示词） | 🔄 运行中 |
| `proc_d86cbeea2bbe` | V1 关键帧版 Seedance 重做 | 🔄 运行中 |

### 守护管理命令
```
process(action="list")                              ← 列出所有守护
process(action="poll", session_id="proc_xxx")       ← 查输出
process(action="wait", session_id="proc_xxx", timeout=600)  ← 等完成
process(action="kill", session_id="proc_xxx")       ← 强制终止
```

---

## 14. 常见 Pitfalls（2026-07-10 教训）

| 问题 | 解决 |
|------|------|
| Agnes 输出横屏 | ✅ `width=720, height=1280` 直接生成竖屏 |
| CDN 超时 | ✅ 用 `raw.githubusercontent.com`，不用 jsdelivr |
| GitHub API SSL EOF | ✅ Python urllib 失败 → 改用 `curl -sk --noproxy "*"` |
| 脸变形 | ✅ 始终用 `ref-v01-frame.jpg` 作为 reference_image |
| 文字乱入 | ✅ prompt 加 `NO text, NO subtitles, NO watermarks` |
| concat filter 失败 | ✅ 改用 `-f concat` demuxer 或 filter_complex concat |
| 找不到 .env | ✅ 在 `D:\hermes\.env`（junction 透明） |

---

## 15. 快速启动命令

```bash
# 查看当前状态
cd /d/复仇者联盟2026-漫剧/03-视频段/raw && ls -la *.mp4 | wc -l

# 查看所有关键帧
ls -la /d/复仇者联盟2026-漫剧/02-关键帧/

# 检查 TTS
ls -la /d/复仇者联盟2026-漫剧/04-TTS/lines/

# 看守护日志
process(action="log", session_id="proc_8d8a047a872d", limit=20)

# 上传新预览
# (参见第 5.4 节 CDN 上传)
```

---

## 16. 下一步行动计划

1. **等 V1 关键帧版** 守护完成 → 用户审核
2. **全段 Seedance 重做** — V02c → V20（按用户选择用KF版或纯Seedance版）
3. **TTS 合成** — 22句 mp3 已就绪，等视频全齐
4. **字幕 burn** — FFmpeg drawtext 后期叠加
5. **EP01 最终版** — 拼接全部 22 段 + TTS + 字幕 + BGM
6. **EP02-06** — 等 EP01 数据验证后启动

---

**文档版本**：v1.0 | **生成时间**：2026-07-11 02:35 | **维护者**：Jack + Hermes
