# MiniMax H3 in ComfyUI: engine + craft rules (generic; from one production node, ~430 generations, 2026-08..10)

Distilled from one production node's measured history (~430 classified generations, 2026-08..10). Every rule carries
its status: PROVEN (measured here, with a control) · MEASURED n=k · DOCUMENTED (vendor docs, not yet measured) ·
COMMUNITY (third-party, unmeasured). A status changes only by a local measurement. Box facts (paths, card, power cap,
never-touch list) live in each node's twin skill; this file has none.

## 0. Before anything
- Most failures are prompting, planning and execution, not the model. A review of 85 classified failures found 0 that
  were "everything right and the model failed"; 31 re-paid for a rule already written down, and reading the rules did
  NOT protect (13 of 16 ignored-rule failures came from sessions that had read them). **Rules that matter belong in a
  pre-submit GATE that refuses, not in prose.** Repeated proven combinations succeed ~79%, first-time ones ~42%: keep a
  capability matrix of what is proven and run anything unproven ONCE, labelled an experiment.
- Licence (§9): testing is fine anywhere the owner allows; PRODUCTION output displayed in an excluded territory is not.

## 1. Two modes (one per generation; the nodes cannot be combined)
| | pinned (`MiniMaxH3ImageToVideo`, fl2va / i2v) | reference (`MiniMaxH3ReferenceToVideo`, ref2va) |
|---|---|---|
| images | `first_frame` and/or `last_frame` rendered LITERALLY | up to 9 `ref_images` (identity/set/props), never rendered; `<Picture N>` |
| voice | cast from the description (and the seed) | up to 3 `ref_audios`, `<Audio N>` |
Text-only = the image node with no frames.
- **Joins between clips:** fl2va with BOTH ends pinned is the transition primitive (bridge: same framing, bodies move;
  PROVEN). A one-end pin ("continue from the last frame") WANDERS: walks to the lens, doubles characters, props appear
  (PROVEN). A 22-frame video tail pinned at the head (`MiniMaxH3AddGuide`, frame 0) buys the JOIN but not the HOLD; the
  set still drifts over 6 s, less (MEASURED n=1). Render the scenes first, then each transition from the rendered frames.
- **Character shots look best as I2V from ONE crisp start still per scene** (PROVEN: reference mode invented every frame
  and softened the face; a face-refine pass on top softened the mouth further).

## 2. Wiring
- Autogrow inputs are DOT-NAMESPACED (`ref_images.ref_image_0`, `ref_audios.ref_audio_0`); a list/dict under the base key
  is SILENTLY DISCARDED (PROVEN). `ref_image_size: match` (`max` saturated 24 GB and hung). Refs <=2048 px short edge,
  never upscaled: identity lives in face pixels.
- `length` = 17k+5 frames at 24 fps (56/124/209/243). The "1344×768" cap is an AREA cap: 9:16 is 768×1344 at the same
  budget, and I2V from a 9:16 plate gives a composed 9:16 video (MEASURED n=1). Never pad-blur a landscape take into 9:16.
- Base graph: text encoder → UNet (GGUF) → `MiniMaxH3SigmaShift` → node → res_multistep / simple / 20 steps.
- Text encoder: H3 ships a Qwen3-VL-32B (~15.7 GB). ClipProj (`NicoLab28/ClipProj-MiniMax-H3`, a fitted projection) lets
  a 4B stand in (~4.6 GB) with no other change; that plus Q4_K GGUF UNets fits H3 on one 24 GB card. **The 32B fixed
  likeness AND exact words** (7/7 exact lines vs 0/6 on the 4B, MEASURED) at the cost of a COLD LOAD (~20 min on a
  24 GB card that spills; the encode itself is seconds warm): keep it cached across shots. `nvfp4` is emulated on Ampere
  and native on Blackwell (sm_120): expect the 32B nvfp4 encoder to be far cheaper on a Blackwell card (unmeasured).
- Sources: UNets `unsloth/MiniMax-H3-GGUF` (Q4_K is the measured default); VAEs + encoders `Comfy-Org/MiniMax-H3`; nodes
  `thaakeno/ComfyUI-MiniMax-H3-Studio`, `city96/ComfyUI-GGUF`, `nicolab28/ComfyUI-ClipProj`.
- **Tokenizer check after every ComfyUI upgrade:** before ComfyUI PR #15808 (2026-08-22) `<d>` encoded as TWO tokens and
  corrupted dialogue; dialogue reports older than that are suspect. Verify `<d>`→151669, `</d>`→151670.
- Suppress H3's own music with the documented literal last line `Non-diegetic music: N/A` (prose "no music" still
  produced stings, PROVEN). Exclusions are written POSITIVELY ("the camera holds a perfectly static shot throughout"):
  H3 is CFG-distilled and has no negative-prompt path (DOCUMENTED). Prompt cap 7,000 characters.

## 3. Dialogue and voices
- **Spoken words go in double quotes, in their OWN sentence** (`He says exactly this and nothing else: "…"` then `He
  speaks no other words in this shot.`). Direction in the same clause as the line gets VOICED (PROVEN). A pause is an
  ACTION between two quoted lines, never a phrase inside one. Every line `<d>[English] … </d>`, delivery text outside it.
- **Any speech sentence after the last line is voiced** (a speaker tag, "X speaks in the voice of", a name + verb):
  put the speaker→voice map BEFORE the beats and end on the last line + a nameless room-tone tail (PROVEN).
- **One speaker: NO `(S1)` tag** (it cost ~60 Hz of register, MEASURED n=3). Two or more: number `(S1)` as whoever speaks
  FIRST (the documented cause of swapped voices), every ID has >=1 line (a labelled mute steals lines), never tell a
  SPEAKER to be silent, identify by screen POSITION, one beat per speaker in order, a silent tail beat after the last
  line (or the last word clips). Two described same-gender voices in one take: 3 pass / 1 swap; in a chained episode use
  ONE speaker per pinned clip.
- **The first and last word are the unreliable positions:** final word off-script 10.1%, prepended filler 4.3% (n=188).
  Never leave a word the client must hear at either end; name both ends in the beat; after take 3, move it to a card.
- **Pace:** H3 talks across the whole clip. Flag rate by scripted density: <1 wps 33% · 1–2 wps 30% · 2–3 wps 4%
  (n=77). Gate <2.0 or >3.3 wps; fewer sentence breaks (the model inserts invented words AT a break when there is room).
  Give the clip a little more time than the speech; a silent reveal before a line is its own shot.
- **In pinned mode the SEED is the casting:** same description, different seed = a different actor (150 vs 194 Hz).
  Lock a cast seed per character. Prose does not move REGISTER (only "brighter/higher" moves pitch, upward). A pinned
  chain re-casts the voice every clip (cosine vs the locked ref 0.72/0.62/0.39): measure each spoken clip and convert to
  the ref in post; a re-render is another roll.
- **Casting that works:** describe the voice, render pinned, have a human judge, then capture 8–10 s of SPEECH-ONLY audio
  (pcm_s16le/44.1k/mono) as the reference and use reference mode from then on. A delivered clip is a bad pitch instrument
  (H3's sub-80 Hz bed pins f0 trackers to their floor).
- **Same-gender voice convergence is an open upstream bug** (MiniMax-H3 #17, ComfyUI #15454): describe RANGE and TEXTURE,
  not just pitch; there is no prompt fix.
- **Memorised identities (famous characters):** the NAME carries look, voice and mannerisms (MEASURED n=1). One name-only
  take to discover the look, then lock it: reference the SET, name the FACES, text-lock the CLOTHES. Face crops from your
  own output as refs made likeness WORSE. A wardrobe line summons its character into the shot; mention absent characters
  not at all. Check new names against a known-identity collision list. **A real person's likeness, or an IP character,
  never reaches a client deliverable.**
- **A real person from one photo (consented):** ref2va, `<Picture 1>` a set plate with an EMPTY mark and nobody in the
  subject's clothes, `<Picture 2>` the photo (three named features, their own clothes), `<Audio 1>` ~14 s of their REAL
  recording (never a TTS clone). The identity ref binds to whoever in the plate wears its clothes.
- **Text conditioning moves the AUDIO:** one mis-bound sentence of another character's business degraded likeness AND
  voice; fixed by naming, with seed and refs unchanged (PROVEN). A bare pronoun binds to whoever was named LAST: lead every
  per-character sentence with the name.
- **Head-of-clip junk burst:** ~1 in 4 renders opens with a 0.1–0.8 s junk burst before digital silence. Mute up to the
  start of that silence gap, never a fixed trim; a continuous bed is CANNOT-TELL. Re-check after every re-cut.
- Singing: lyrics in `<d>` under "sings", the strum/mouth as the motion spine (as an adjective the hands froze). H3 raps
  slower than the record: cut phrase pieces onto the track's word times. A sung line must be the TRACK'S words at that
  time. `AddGuide.audio` anchors the soundtrack, not the motion (mouth-to-audio sync fell); a masked AV-context audio-lock
  node holds the supplied track (MEASURED n=6).

## 4. Silence and sound
- **A front-facing closed mouth will not hold, and instructions make it worse** (motion 7.17 → 7.45 with "mouth CLOSED",
  vs 4.66 when genuinely speaking; PROVEN). Fix the FRAME: rear view, over the shoulder, or far enough that the mouth is
  a few pixels (profile still reads). On a pinned take a reframe is a PLATE change; rewriting the camera string over an
  unchanged plate changes nothing.
- **Verify mouths by MOTION, never stills:** a mouth opens and closes in ~6 frames. Compare against a shot where the same
  character IS speaking; an absolute threshold means nothing.
- **No-speech shots:** fill the audio space (a soundscape paragraph: ambience + the physical sounds of every beat + a
  non-verbal human sound), put "silently" on each non-speaking action verb, close every voice path ("No dialogue. No
  voiceover. No singing…"). "Nobody speaks" alone is a coin flip. Professionals replace generation audio in post anyway.
- **A word anywhere in the prompt is spoken by the visible face** (off-screen voice-over and soundscape forms both FAILED
  n=2): an unseen voice goes in post, or in its own shot.
- A foley named only in the closing tail was not heard: write it IN the beat as a sound with a time. Indoor vs outdoor is
  REVERB: state the room's acoustic as a physical fact (decay 275 ms vs 185 ms passed where EQ'd beds failed). An
  off-screen soundscape renders at the level the PICTURE implies (−60 dB behind an empty-looking room): put the source on
  camera. An absence is not renderable ("no cicadas" gave crickets): state the positive fact.

## 5. Blocking, frame 0, continuity
- **The model holds NOTHING between generations.** A prompt may not refer to another shot ("same framing as…" is a pointer
  into a void). Byte-identical camera-geometry + set blocks are NECESSARY, not sufficient: in text-to-video every take
  re-rolls the set. Only a pinned `first_frame` or wired `ref_images` hold a set; stage the second shot's frame 0 from a
  real frame of the first.
- **On a pinned take the plate's subject SCALE is ground truth:** a framing word that contradicts it ADDS a character (a
  second, larger copy appeared at the requested scale; PROVEN with control). Crop the plate instead.
- **Camera distance is not a crop:** "five metres back" and "six metres back" both gave a wide with the subject at ~5%;
  describing the frame's CONTENTS at the size they occupy gave exactly that crop (PROVEN). Shot-size labels are the same
  abstraction. Framing is a DESTINATION unless you also say the camera does not move.
- **Style needs objects:** a characterless shot under a flat-cartoon preamble came back photoreal; weld the style to every
  object, negate photoreal cues by name, AND say surfaces are FILLED (negations alone converge on wireframe). A 2D look →
  pinned from a 2D plate (reference mode drifts toward 3D).
- **A pinned shot cannot pay off with a grade** (the first frame fixes exposure): make the payoff an ACTION.
- Every visible character needs an action in every beat or it freezes. An entering character starts OFF frame, the door
  stated CLOSED, its opening an EVENT. Never restate a wired set reference in words. A FULL-LENGTH identity ref locks
  wardrobe (chest-up refs re-roll the lower body). Plan a scale ladder; three wides in a row reads as no planning.

## 6. Verification
- Transcript PRECISION, not just recall: `off_script_frac` (share of spoken content words not in the script); CHECK at
  >=0.15 (rejected 0.23–0.38 vs passed 0.00–0.09). Beam search + timestamps; never split on whisper punctuation; whisper
  is not deterministic.
- Lip-sync as a named check (mouth-ROI motion in speech vs silence: MOUTH-FLAP / STATIC-MOUTH / pass / n.a.).
- Identity, voice identity, who-spoke and motion direction are HUMAN-ONLY; embeddings are a check, never a verdict.
  >=3 seeds for any continuous metric; a disconnect control for any new reference type.
- A completed job proves nothing: status success AND the file. Watch and listen before anything ships. A verdict on a clip
  in isolation can flip in the cut. Calibrate any comparison meter on a known-identical and a known-different pair first.

## 7. Speed and hardware
- Know which regime you are in: OFFLOAD-bound (weights stream; GPU util is a sawtooth; RAM/residency is the lever) or
  COMPUTE-bound (util 78–99%; step count is the lever). One `nvidia-smi` sample cannot tell "wedged" from "rendering":
  sample 4× over ~30 s. Low GPU util with one pegged CPU core = launch-bound, not "GPU too small".
- Turbo v4 (`larryvrh_v4_step600`) wired the author's way (`MiniMaxH3TurboLoRA` + `MiniMaxH3TurboSampler`, 6 steps, no
  SigmaShift) = ×2.75–2.92 wall-clock when compute-bound (n=3). **Drafts only**: finals stay full encoder + 20 steps.
  The old 4-step file ranks BELOW baseline in a 21k-vote blind arena.
- **Draft → final = upscale the APPROVED take (e.g. SeedVR2), never a re-render**: the same seed at another resolution is
  a different take. Put every performance setting in the draft.
- More watts can be SLOWER (hotspot throttling); a new quant format is only a speed lever if its op is on ComfyUI's
  native list (others are emulated; re-read after upgrades). On Windows, a spill to shared GPU memory is invisible to
  nvidia-smi (use the `GPU Process Memory` counters), and an idle co-tenant app holding VRAM can starve the encoder.
- H3 output is single-GOP: re-encode (`-g 24`) before any edit. A byte-identical resubmit returns from cache: change the
  seed. ComfyUI history is in memory: reconcile before a restart.

### Delivery gate: keyframe-gap check
Single-GOP output cannot be trimmed by an editor, and "re-encode with `-g 24`" sat as prose for 7 weeks before it was a
gate. Fail any render whose LONGEST keyframe gap is over 12 s, counting the tail (last keyframe to end of file), so a
one-keyframe clip fails too (gap = full duration). 12 s because x264's default is 10.4 s at 24 fps, so no encoder default
trips it. Measure with ffprobe packet flags:
`ffprobe -v error -select_streams v:0 -show_entries packet=pts_time,flags -of csv=p=0 f.mp4` -> keep `pts_time` where
flags start with `K`, add the stream duration as a final point, take the max difference. Positive control: a
one-keyframe clip must be refused (20 s clip -> gap 20.0); replay over real renders before enabling (60 renders: max 8.33 s).
Drive the finishing step on an excerpt of a REAL render, not a synthetic pattern.

## 8. Dead ends (each with the question it closed)
FastH3 v1 (multi-GPU Blackwell; v2 is single-GPU, unmeasured) · RAVEN LoRA (~160 GB RAM) · `ref_image_size: max` (hangs) ·
Mac MPS upscale (~37 min/clip). "Turbo gives no speed" was true only of an offload-bound box. When a measurement reverses
a rule, rewrite every line that states it, not just a correction below.

## 9. Licence (MiniMax H3, licence dated 2026-08-02; read the primary text, not a summary)
Use is granted "solely within the Applicable Territory", and you may not use or **display** the works **"or any of their
Outputs"** outside it. Excluded Territories: the EU, the UK, the Republic of Korea and the USA. So it binds OUTPUTS and
their display, not just where the weights run. A commercial product using H3 must prominently display "MiniMax H3". The
hosted API is not territorially restricted the way the weights are. Each node's twin records its location and the
owner's ruling (e.g. testing allowed; production output needs the owner's OK).
