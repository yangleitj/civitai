# MiniMax H3 first + last frame, 10 seconds

This is the exact ComfyUI path that wrote `MoodyH3_FL2V_10s_00001_.mp4` on 2026-09-18.

## 1. Stills

| Role | Local file | Size |
| --- | --- | --- |
| First | `C:\\Users\\yangl\\OneDrive\\Desktop\\首帧.png` | 1024x1536, curled sideways |
| Last | `C:\\Users\\yangl\\OneDrive\\Desktop\\尾帧.png` | 1024x1536, facing camera |

Copy into the Comfy input folder (LoadImage only lists this directory):

```
C:\\Users\\yangl\\AppData\\Local\\Comfy-Desktop\\ComfyUI-Shared\\input\\first_frame.png
C:\\Users\\yangl\\AppData\\Local\\Comfy-Desktop\\ComfyUI-Shared\\input\\last_frame.png
```

H3 native canvas is a 768px short edge, multiples of 32. Scale both stills to **768x1152**, lanczos, center crop.

## 2. Open the graph

In Comfy Desktop, open **video_minimax_h3_i2v_HERETIC**.

`MiniMaxH3ImageToVideo`:

- first frame only = I2V
- **first + last** = interpolate A to B (this job)

Models already on this box:

| Slot | File |
| --- | --- |
| UNet | `minimax_h3_fl2va_pruned_int8_convrot.safetensors` |
| CLIP | `qwen3vl_32b_h3_ultra_uncensored_heretic_int8_convrot.safetensors` (**type = minimax**) |
| Video VAE | `minimax_h3_video_vae_fp16.safetensors` |
| Audio VAE | `minimax_h3_audio_vae_fp32.safetensors` |
| Turbo LoRA | `minimax_h3_fl2v_turbo_8step_v1.0_comfyui_bf16.safetensors` at strength 1.0 |

## 3. Wire it

1. LoadImage `first_frame.png` -> ImageScale 768x1152 -> **first_frame**
2. LoadImage `last_frame.png` -> ImageScale 768x1152 -> **last_frame**
3. Enable turbo, **8 steps**
4. Width/height **768 / 1152** (not the template 1:1)
5. Duration **10 seconds**

H3 snaps length to 24 fps on a `17k+5` grid:

```
10 * 24 = 240
240 % 17 = 2
240 + (5 - 2) = 243 frames
```

Set `length = 243`.

Sampler used:

- `res_multistep`
- scheduler `simple`
- steps 8, denoise 1.0
- seed `100010`
- CreateVideo fps 24
- SaveVideo prefix `video/MoodyH3_FL2V_10s`

Paste [prompts/fl2v_10s.txt](../prompts/fl2v_10s.txt). Locked camera, no push-in, face stays in frame.

Queue with Ctrl+Enter. This run took about 5 minutes on RTX 5090.

## 4. Node chain

```
LoadImage(first) -> ImageScale 768x1152 -> first_frame \
LoadImage(last)  -> ImageScale 768x1152 -> last_frame  +-> MiniMaxH3ImageToVideo
CLIP (minimax) ------------------------------------------+
Video VAE -----------------------------------------------+

UNet fl2va -> Turbo LoRA -> BasicScheduler (simple, 8)
                         -> BasicGuider <- positive from MiniMaxH3ImageToVideo

RandomNoise + KSamplerSelect(res_multistep)
        -> SamplerCustomAdvanced(latent from MiniMaxH3ImageToVideo)
        -> VAEDecode (video VAE) + VAEDecodeAudio (audio VAE)
        -> CreateVideo 24fps -> SaveVideo
```

API-format graph: [workflows/h3_fl2v_10s_api.json](../workflows/h3_fl2v_10s_api.json). POST that to `http://127.0.0.1:8188/prompt`.

## 5. Result

`C:\\Users\\yangl\\AppData\\Local\\Comfy-Desktop\\ComfyUI-Shared\\output\\video\\MoodyH3_FL2V_10s_00001_.mp4`

Checked frames 0 / 81 / 162 / 242: curled start -> speaking and opening legs -> front-on spread -> last-frame hand pose. Face stayed in the shot.
