# 用 ComfyUI 做 10 秒首尾帧视频（MiniMax H3）

这份说明对应本机实际跑通的那条片子：

`C:\Users\yangl\AppData\Local\Comfy-Desktop\ComfyUI-Shared\output\video\MoodyH3_FL2V_10s_00001_.mp4`

规格：768×1152，243 帧，24fps，大约 10 秒，带立体声。镜头锁定，脸全程在画面里。

不要跳步。按下面从 1 做到 12。

---

## 1. 确认 ComfyUI 已经开着

1. 打开 `C:\Users\yangl\AppData\Local\Programs\Comfy Desktop\Comfy Desktop.exe`。
2. 等实例起来。浏览器或本机访问 `http://127.0.0.1:8188` 能打开才算好。
3. 队列必须是空的：左侧 Queue 里没有正在跑的任务。

数据目录：

- 输入：`C:\Users\yangl\AppData\Local\Comfy-Desktop\ComfyUI-Shared\input`
- 输出：`C:\Users\yangl\AppData\Local\Comfy-Desktop\ComfyUI-Shared\output`
- 工作流：Comfy Desktop 左侧列表，来自 live install 的 user workflows，不是 Documents\ComfyUI。

---

## 2. 确认模型文件在

在 Comfy 的模型目录里（Shared models 或 MiniMax-H3-Setup）必须能看到这些文件。缺一个都不要排队。

| 放哪一类 | 文件名 |
| --- | --- |
| diffusion_models | `minimax_h3_fl2va_pruned_int8_convrot.safetensors` |
| text_encoders | `qwen3vl_32b_h3_ultra_uncensored_heretic_int8_convrot.safetensors` |
| vae | `minimax_h3_video_vae_fp16.safetensors` |
| vae | `minimax_h3_audio_vae_fp32.safetensors` |
| loras | `minimax_h3_fl2v_turbo_8step_v1.0_comfyui_bf16.safetensors` |

CLIP 的 type 必须是 **minimax**，不能选 krea2、ltxv 或默认 SD。

---

## 3. 准备首帧、尾帧

本机当时用的两张图：

| 角色 | 文件 | 原尺寸 | 画面 |
| --- | --- | --- | --- |
| 首帧 | `C:\Users\yangl\OneDrive\Desktop\首帧.png` | 1024×1536 | 灰沙发上抱膝侧坐 |
| 尾帧 | `C:\Users\yangl\OneDrive\Desktop\尾帧.png` | 1024×1536 | 正对镜头、腿分开 |

要求：

- 两张都是同一个人、同一套衣服、同一个房间。
- 都是竖图。横图不要硬塞进这套 768×1152 设置。
- 不要用带中文名的文件直接给 LoadImage，先复制并改名。

---

## 4. 拷进 Comfy 输入目录并改名

在资源管理器里复制：

1. 把 `首帧.png` 复制到  
   `C:\Users\yangl\AppData\Local\Comfy-Desktop\ComfyUI-Shared\input\first_frame.png`
2. 把 `尾帧.png` 复制到  
   `C:\Users\yangl\AppData\Local\Comfy-Desktop\ComfyUI-Shared\input\last_frame.png`

LoadImage 只扫 input 目录。文件还在桌面上、或还在 output 里，节点下拉列表里是找不到的（除非带 `[output]` 后缀，不推荐）。

---

## 5. 打开工作流

1. Comfy Desktop 左侧 Workflows 点 **video_minimax_h3_i2v_HERETIC**。
2. 这是 MiniMax H3 的 Image to Video 图，核心节点是 `MiniMaxH3ImageToVideo`。
3. 接法：
   - 只接 first_frame → 普通图生视频
   - **first_frame + last_frame 都接** → 从 A 姿势插值到 B（这条片子用的就是这个）

不要打开 LTX 的 Image to Video，也不要打开 `video_minimax_h3_t2v_HERETIC`。

---

## 6. 接图：首帧

1. 找到 `LoadImage`。
2. 图片下拉选 `first_frame.png`。
3. 后面接 `ImageScale`（有的图画里叫 Upscale Image）：
   - upscale_method：`lanczos`
   - width：`768`
   - height：`1152`
   - crop：`center`
4. ImageScale 的 IMAGE 输出接到 `MiniMaxH3ImageToVideo` 的 **first_frame**。

为什么要缩：H3 原生短边 768，边长必须是 32 的倍数。1024×1536 直接塞进去又慢又容易糊。768×1152 是 1024×1536 的等比缩小。

---

## 7. 接图：尾帧

1. 再放一个 `LoadImage`，选 `last_frame.png`。
2. 同样接一个 `ImageScale`：lanczos，768×1152，center。
3. 接到 `MiniMaxH3ImageToVideo` 的 **last_frame**。

两个 ImageScale 的宽高必须完全一样。一边 768×1152、一边还是 1024×1536，会失败或变形。

---

## 8. 模型节点核对（不要改文件名）

在 H3 子图 / 模型组里确认：

1. **UNETLoader**  
   `minimax_h3_fl2va_pruned_int8_convrot.safetensors`  
   weight_dtype：`default`
2. **CLIPLoader**  
   `qwen3vl_32b_h3_ultra_uncensored_heretic_int8_convrot.safetensors`  
   type：`minimax`  
   device：`default`
3. **VAELoader（视频）**  
   `minimax_h3_video_vae_fp16.safetensors`
4. **VAELoader（音频）**  
   `minimax_h3_audio_vae_fp32.safetensors`
5. **LoraLoaderModelOnly（Turbo）**  
   `minimax_h3_fl2v_turbo_8step_v1.0_comfyui_bf16.safetensors`  
   strength：`1.0`
6. 打开 **turbo_mode**（模板里的 Boolean / Enable Lightning LoRA）。关掉的话步数会走 20，更慢。

---

## 9. 分辨率和时长

在 `MiniMaxH3ImageToVideo` 或外面的 ResolutionSelector 上：

1. 不要用模板默认的 `1:1 (Square)`。
2. 写成：
   - width：`768`
   - height：`1152`
3. 时长填 **10 秒**。

H3 按 24fps、网格 `17k+5` 取整，不会恰好 240 帧：

```
10 × 24 = 240
240 ÷ 17 余 2
240 + (5 - 2) = 243 帧
```

所以 `length` 必须是 **243**。自己乱填 240 或 241 会被节点改掉或对不齐。

---

## 10. 采样设置

和当时跑通的参数保持一致：

| 项 | 值 |
| --- | --- |
| Sampler | `res_multistep` |
| Scheduler | `simple` |
| Steps | `8`（turbo 开着） |
| Denoise | `1.0` |
| Seed | `100010`（要复现就锁这个；要新结果就 randomize） |
| CreateVideo fps | `24` |
| SaveVideo 前缀 | `video/MoodyH3_FL2V_10s` |
| format / codec | `auto` |

镜头约束写在提示词里，不要再加推镜、变焦节点。

---

## 11. 提示词（整段贴进去）

打开 `MiniMaxH3ImageToVideo` 的 prompt 框，**先清空模板里那套鼠标广告**，再整段粘贴：

```
Keep the exact woman from the first frame and last frame: same face always fully visible, same long light-brown wavy hair and bangs, same black dress with large white neck bow and gold buttons, same dark chain ankle boots, same gray armchair. Start exactly on image 1 (curled sideways hugging knees). End exactly on image 2 (sitting facing camera, knees spread wide, no underwear, vulva exposed, her hand touching her labia).

Locked camera. Do not push in. Do not zoom. Do not crop the face out. Keep a medium-wide shot of the full seated body plus face in frame for the entire 10 seconds. One continuous shot, no cuts.

Timeline:
She looks at her husband off-camera, flushed and teasing. She slowly unhooks her arms, opens her knees, the short dress hem rides up. She turns more front-on in the chair, spreads her legs wider until they match the last frame, then reaches down and strokes her labia with her fingers, arriving on the last-frame pose. Breathing heavier, small moans, face stays in the upper third of the frame.

Audio, Mandarin, close intimate adult female voice, breathy, teasing her husband:
"老公...嗯..."
"快来啪啪我吧...等不及了..."
soft wet moan
"嗯啊...老公...快点..."
Quiet indoor room tone, fabric rustle, no music, no other speakers.
```

独立文件：[prompts/fl2v_10s.txt](prompts/fl2v_10s.txt)

写提示词时记住：

- 不要再描写五官和衣服，图已经定了。
- 必须写 **Locked camera / Do not push in**，否则后半段会推到脸上、头出画。
- 对白写在 Audio 段，用中文引号括起来，H3 才会尝试出声。

---

## 12. 排队、等、看结果

1. 快捷键 **Ctrl+Enter**（Queue Prompt）。
2. 看 Queue：running 一条。RTX 5090 上大约 **5 分钟**。
3. 不要中途点 Interrupt，除非你确认提示词还是鼠标广告那套。
4. 完成后输出在：

`C:\Users\yangl\AppData\Local\Comfy-Desktop\ComfyUI-Shared\output\video\MoodyH3_FL2V_10s_00001_.mp4`

（如果已经有 `_00001_`，下次会变成 `_00002_`。）

5. 用播放器打开，检查三件事：
   - 第 1 帧是不是抱膝那张
   - 最后一帧是不是分开腿那张
   - 中间脸有没有出画
   - 中文对白清不清（H3 有时会含糊）

当时抽过 0 / 81 / 162 / 242 帧：抱膝 → 开口、腿开始打开 → 正对镜头分开 → 落到尾帧手势。脸一直在。

---

## 用 API 重跑（可选，不点界面）

图已经拷进 input 之后，把 [workflows/h3_fl2v_10s_api.json](workflows/h3_fl2v_10s_api.json) POST 到：

```
http://127.0.0.1:8188/prompt
```

这个 JSON 里已经写好节点、768×1152、243 帧、turbo 8 步、seed 100010 和上面那段 prompt。

---

## 常见失败

| 现象 | 原因 | 处理 |
| --- | --- | --- |
| LoadImage 列表里没有图 | 文件还在桌面 | 拷到 Shared\input 并改成英文名 |
| 报错 CLIP / type | CLIP type 不是 minimax | 改成 minimax |
| 画面是正方形、人被裁 | 还在用 1:1 0.4MP | 改 768×1152 |
| 后半段只剩身体没有脸 | 提示词里有 push-in | 删掉推进，写 Locked camera |
| 还在播鼠标广告旁白 | 没清模板 prompt | 整段换成 fl2v_10s.txt |
| 只有 2～5 秒 | duration 还是模板的 2 或 5 | 改 10 秒，length=243 |
| OOM | 没缩小、或时长更长 | 先保证 768 短边 |

---

## 仓库里还有什么

- [docs/h3-first-last-10s.md](docs/h3-first-last-10s.md) — 英文对照版
- [prompts/fl2v_10s.txt](prompts/fl2v_10s.txt) — 提示词原文
- [workflows/h3_fl2v_10s_api.json](workflows/h3_fl2v_10s_api.json) — API 图

首尾帧 png 和成品 mp4 **不进 git**，只留在你电脑上。
