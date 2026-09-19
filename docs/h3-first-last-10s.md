# MiniMax H3 first + last frame, 10 seconds

完整逐步说明（中文、按点击顺序）在仓库根目录：

**[../README.md](../README.md)**

That README is the source of truth: Comfy Desktop start, model files, copy `首帧.png` / `尾帧.png` into Shared input, open `video_minimax_h3_i2v_HERETIC`, scale 768x1152, turbo 8, length 243, locked-camera prompt, Ctrl+Enter, output path.

Prompt file: [../prompts/fl2v_10s.txt](../prompts/fl2v_10s.txt)

API graph to POST at `http://127.0.0.1:8188/prompt`:
[../workflows/h3_fl2v_10s_api.json](../workflows/h3_fl2v_10s_api.json)

Output of the run this documents:

`C:\Users\yangl\AppData\Local\Comfy-Desktop\ComfyUI-Shared\output\video\MoodyH3_FL2V_10s_00001_.mp4`
