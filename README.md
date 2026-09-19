# civitai

Private ComfyUI notes for this machine. The working pipeline that produced `MoodyH3_FL2V_10s_00001_.mp4` is documented here.

## Output

`C:\Users\yangl\AppData\Local\Comfy-Desktop\ComfyUI-Shared\output\video\MoodyH3_FL2V_10s_00001_.mp4`

- 768x1152, 243 frames, 24 fps, ~10s, AAC stereo
- MiniMax H3 first + last frame (`fl2va`), turbo 8 steps
- Locked camera, face in frame the whole clip

Full UI steps: [docs/h3-first-last-10s.md](docs/h3-first-last-10s.md)

API graph: [workflows/h3_fl2v_10s_api.json](workflows/h3_fl2v_10s_api.json)

Prompt: [prompts/fl2v_10s.txt](prompts/fl2v_10s.txt)

First/last stills and the mp4 are **not** in this repo. Keep them local.
