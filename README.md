# H3 (MiniMax H3) video workflows: reference

Reviewed and extended 2026-11-25 by the render node that actually produces H3 video.

## Two caveats (read before relying on or publishing anything here)
1. **Licence output-display exclusions:** the H3 weights' licence (dated 2026-08-02) excludes the EU, UK, South Korea and
   the USA, and covers displaying OUTPUTS there, not just running the weights. See H3-RULES.md §9; read the primary text.
2. **Numbers come from one 24 GB CUDA card.** AMD GPUs are UNMEASURED for H3; treat all speed/VRAM figures as
   not transferable until tested.

## Start here
1. `H3-RULES.md`: the real working reference. Two modes (pinned image-to-video vs reference-to-video), ComfyUI wiring
   (node names, dot-namespaced inputs, frame counts, 20-step base graph, text-encoder choices and VRAM), dialogue/voice
   rules, silence/sound, continuity, verification, speed (offload- vs compute-bound), dead ends. Each rule is labelled
   PROVEN / MEASURED n=k / DOCUMENTED / COMMUNITY.
2. **Licence, read before you render anything for anyone (H3-RULES.md §9):** the weights' licence (dated 2026-08-02)
   excludes the EU, UK, South Korea and the USA, and it covers displaying OUTPUTS there, not just running the weights.
   Testing is one thing; production output is another. Check where you and your audience are, and read the primary text.

## What this package does NOT contain (honest)
- No ComfyUI JSON export. The base graph is described in H3-RULES.md §2 (text encoder -> GGUF UNet -> SigmaShift -> H3
  node -> res_multistep/simple, 20 steps); an actual workflow file is a separate hand-over from the render node.
- No script-writing methodology beyond what `SCRIPT-WRITING.md` says; the render node has a fuller internal skill that has not been sanitized for sharing.
- Hardware: figures are from a single 24 GB card with offload. AMD GPUs are untested for H3: the
  nodes, GGUF path and quant ops are CUDA-world assumptions. Treat as unmeasured.
