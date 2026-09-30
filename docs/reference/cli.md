# CLI Reference

Runnable scripts and what they do. The `lab 2/` and `lab 3/` scripts are thin wrappers around the
`dggr/` package, where the implementations live.

Conventions:
- Run commands from the repo root unless noted.
- Large artifacts are not committed (`saves/`, `saves2/` are gitignored).
- Prefer setting `DGGR_MANIFESTS_ROOT` and `DGGR_DATA_ROOT` for portability (see
  `docs/reference/env_vars.md`).

## Lab 2

### `lab 2/run_lab2.py`

Purpose:
- Harvest Lab 1 embeddings (`z_content`, `z_style`) for curated manifests.
- Build 160D target vectors and validate target-space separability.

Key flags:
- `--checkpoint`: frozen Lab 1 checkpoint (`saves/.../latest.pt`).
- `--manifests-root`: path to cleaned manifests (defaults via `DGGR_MANIFESTS_ROOT` fallback).
- `--manifest-files`: list of CSVs to include.
- `--per-genre-samples`: sample cap per genre.
- `--output-dir`: where to write artifacts (`saves/lab2_calibration/...`).
- `--reuse-artifacts-dir` + `--projection lda`: re-warp an earlier harvest without re-running it.

Outputs:
- `validation_summary.json`
- centroid CSVs / JSON exports (depends on run mode)

## Lab 3 (mel-target decoder)

### `lab 3/run_lab3.py`

Purpose:
- Stage 1 self-reconstruction, then stage 2 genre-shift synthesis in mel space.
- The first synthesis path tried; the codec and diffusion tracks below carry the reported results.

Key flags:
- `--mode fresh|resume`, `--resume-dir`: run control.
- `--run-name`, `--strict-run-naming`: enforce `run1`, `run2`, ... under `saves2/`.
- `--genre-schema {default4,binary_acoustic_beats}`, `--balance-sources-within-genre`,
  `--require-min-sources-per-genre`, `--require-is-music`: anti-leakage controls.
- `--chunks-per-track`, `--min-start-sec`, `--max-start-sec`: multi-chunk cache sampling.

Outputs:
- `saves2/lab3_synthesis/runN/` with `run_state.json`, `history.csv`,
  `checkpoints/stage{1,2}_latest.pt`, `lab3_exit_audit.json`, and an exported
  `samples/posttrain_samples/` pack.

## Lab 3 (codec-latent transfer)

### `lab 3/run_lab3_codec.py`

Purpose:
- Style transfer by translating EnCodec embeddings `q_src -> q_hat`.
- Content preservation enforced via Lab 1 `z_content` cosine similarity.
- Style control driven by a conditioning embedding (Lab 1 / codec judge / MERT probe).

Key flags:
- `--style-cond-source`: where the style embedding comes from.
  - `mert_probe_embed` is the best-performing conditioning in current results.
- `--style-loss-mode`: how style loss is computed.
- `--translator-direct-output`: removes the residual leash; enabled in the best run (`run1055`).
- `--gate-multi-pass`: apply translator multiple times at eval for stronger style shift (optional).

Outputs:
- `codec_gate_eval.json` (MPS, style confidence, style accuracy, collapse proxy)
- optional exported sample WAVs

### `lab 3/scripts/*.ps1`

The strong-schema and auto-genre drivers. They chain cache build, style bank, training, and audit
into one command; `run_codec_strong_schema_smoke.ps1` is the quick sanity run and
`run_codec_audit_latest.ps1` audits the latest run for source leakage.

## Lab 3 (diffusion)

### `lab 3/run_lab3_diffusion_v2.py`

Purpose:
- Train diffusion V2 (v-prediction UNet) in mel space with EMA + CFG dropout.

Key flags:
- `--cache-dir`: diffusion cache (mel/chroma/onset + z_content/z_style).
- `--out-dir`: run folder for checkpoints + history.
- `--epochs`, `--lr`, `--ema-decay`, `--cfg-dropout-p`

Outputs:
- `v2_config.json`, `v2_history.json`
- `checkpoints/epoch_*.pt`
- `epoch_samples/` (if enabled)

### `lab 3/run_lab3_diffusion_v3.py`

Purpose:
- Fine-tune diffusion V2 with a mel discriminator (hinge GAN + feature matching).

Notes:
- In current runs, V3 did not beat V2 on validation, but the scaffold exists for future tuning.

### `lab 3/run_lab3_diffusion.py`

The first diffusion entry point, superseded by the V2 script above.

## Lab 4 (long-form coherence)

### `lab 3/run_lab4_longform_coherence.py`

Purpose:
- Long-form chunked generation with *coherence-first* constraints:
  - SDEdit-style anchoring to the source mel at timestep `t_start`.
  - Prefix-locking overlap frames at every DDIM step to keep seams consistent.
  - Drift controls (re-anchoring, mel smoothing, HF source blend).

Key flags (highest impact):
- `--t-start`, `--t-start-end`: how far to diffuse the source before denoising (edit magnitude).
- `--prefix-blend`: overlap locking strength (cohesion).
- `--reanchor-every`, `--reanchor-t-start`: periodically reset drift.
- `--style-strength`: mix between source and target style embeddings.
- `--source-mel-blend`, `--hf-source-blend`: anti-warble stabilization.
- `--assemble-domain`: `mel` (one-shot vocoding) vs `audio` (per-chunk vocoding + crossfade).

Outputs:
- `longform_coherent.wav`
- per-chunk WAVs
- `coherence_metrics.json` (boundary diagnostics)

## Labelling and auditing

### `lab 3/run_lab3_auto_genre.py`

CLAP zero-shot labelling, for when corpus labels are really dataset-source labels.
`--labels` is the label set and `--out-csv` the labelled manifest; `--min-conf` pushes weak items
to `unassigned`. Defaults to `laion/clap-htsat-fused` at 48 kHz.

### `lab 3/run_lab3_auto_genre_lab2cluster.py`

The model-free alternative: cluster `target160 = [z_style, descriptor32]` into `--n-clusters`
buckets and write `genre=cluster_i`. No text model involved.

### `lab 3/run_lab3_quality_audit.py`

Re-derives labels from audio and reports where a run keys on corpus source rather than style.
Takes `--runs` and writes `quality_audit_summary.{csv,json}`.

### `lab 3/run_lab3_target_vector_audit.py`

Probe audit of a finished run's target-vector conditioning: linear probe, centroid distances, and
neighbour hits, written as JSON/CSV.

## Listening and triage

### `lab 3/run_lab3_clip_picker.py`

Interactive accept/reject pass over generated clips (`a` accept, `r` reject, `s` skip, `o` replay,
`q` quit), writing the accepted and rejected lists into a named session.

### `lab 3/run_lab3_human_panel.py`

Builds a blind A/B panel from several runs (`prepare`) and aggregates the ratings plus the audit
metrics (`summarize`). This is the harness for the proposal's Lab 5 listening test; the test itself
has never been run.

## Evaluation helpers

### `lab 3/quick_eval_diffusion.py`
Quickly vocodes a small number of diffusion samples at different guidance scales.

### `lab 3/quick_eval_sdedit.py` and `lab 3/quick_eval_sdedit_v2.py`
Quickly tests SDEdit-style transfer and envelope-transfer variants for timbral shift.

## Data utilities

### `scripts/render_phase1_symbolic.py`
Renders the symbolic classical/baroque pool to audio, using `pretty_midi`.
