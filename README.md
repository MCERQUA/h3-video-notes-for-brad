# Local video generation with MiniMax H3: what we actually know

A short, honest reference written 2026-11-24 for Brad and his agent. It is not a full workflow pack:
we have not yet logged detailed settings, and this repo says so rather than guessing.

## What H3 is, and why not Wan

- "H3" is **MiniMax H3**, our team's standing local video-generation model. **Hunyuan Video 1.5** is
  the tested alternative.
- **Wan is ruled out.** Mike's words (2026-10-25): *"wan video is crap and we should not use it."*
  This is a standing team decision, not a benchmark result. Do not use or recommend Wan.
- Images are a separate model, not video: **FLUX.3 Image Klein**, 5.6B params, int8 about 5.8 GB
  (about 11 GB with its Qwen3-VL 4B text encoder).

## Confirmed

- 2026-11-22: a member of our voice room said Mike runs MiniMax H3 locally for video and called it
  the most productive video model they had seen in that setup. Secondhand praise, no numbers.
- 2026-10-30: H3 formalized as our video model, Hunyuan 1.5 as the tested alternative, Wan ruled out.
  Policy, not a workflow writeup.
- General tip, not H3-specific: on multi-GPU rigs, models that overflow VRAM run far slower
  (more than half the speed was lost in one test). Leave VRAM headroom before tuning video settings.

## Not yet documented

No ComfyUI workflow JSON, node graphs, sampler/step/resolution settings, LoRA names or hard
performance numbers (seconds per frame, VRAM used) are written down here for H3 or Hunyuan 1.5.
Anything more specific would be a guess from public knowledge, not our tested experience.

## Contents

- `README.md`: this file.
- `SCRIPT-WRITING.md`: what we have for video script writing, and what we do not.

## Key

- **Confirmed**: cited from our team's own notes or a named experience.
- **Policy**: a standing team rule, binding for us, not evidence about quality.
- **Not yet documented**: anything we have not written down.
