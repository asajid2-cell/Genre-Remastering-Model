# Reproduce Best Runs (How-To)

Copy-paste recipes for the best observed checkpoints and settings. They assume the caches and
checkpoints are present locally, and that you run from the repo root.

## 1) Codec transfer (best short-form style metrics)

Best run so far: `saves2/lab3_codec_transfer/run1055`, with `style_cond_source=mert_probe_embed`
and `translator_direct_output=true`.

```powershell
python "lab 3/run_lab3_codec.py" `
  --mode fresh `
  --style-cond-source mert_probe_embed `
  --style-loss-mode mert_probe_ce `
  --translator-direct-output `
  --per-genre-samples 600
```

Score a trained run by reading `codec_gate_eval.json` in its run folder.

## 2) Diffusion V2 (best perceived quality checkpoint)

Best subjective checkpoint: `saves2/lab3_diffusion/run_d002/checkpoints/epoch_006.pt`.

```powershell
python "lab 3/run_lab3_diffusion_v2.py" `
  --cache-dir "saves2/lab3_diffusion/run_d001/cache" `
  --out-dir "saves2/lab3_diffusion/run_d002" `
  --epochs 60
```

## 3) Long-form coherence (full-song test)

```powershell
python "lab 3/run_lab4_longform_coherence.py" `
  --cache-dir "saves2/lab3_diffusion/run_d001/cache" `
  --checkpoint "saves2/lab3_diffusion/run_d002/checkpoints/epoch_006.pt" `
  --out-dir "saves2/lab4_longform_coherence/repro" `
  --source-audio "PATH_TO_AUDIO_FILE" `
  --source-genre hiphop_xtc `
  --target-genre baroque_classical `
  --t-start 350 `
  --prefix-blend 1.0 `
  --style-strength 0.75
```

If you hear accumulating warble or static, lower `--t-start`, raise `--source-mel-blend` and
`--hf-source-blend`, or set `--reanchor-every` to 8-16.
