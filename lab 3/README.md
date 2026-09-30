# Lab 3 - Synthesis

Lab 3 turns the analysis stack into a generative one. Both tracks take Lab 1's 128-D `z_content`
plus a Lab 2 target vector, and both are scored on melodic preservation, target-style confidence,
and spectral continuity.

| Track | What it does | Entry point |
|---|---|---|
| Codec-latent transfer | Translates frozen EnCodec latents and decodes straight to waveform | `lab 3/run_lab3_codec.py` |
| Diffusion | Generates log-mel with v-prediction (UNet, EMA, CFG dropout) and vocodes with BigVGAN | `lab 3/run_lab3_diffusion_v2.py` |

A third path, `lab 3/run_lab3.py`, reconstructs log-mel directly in two stages - self-reconstruction,
then genre shift. It came first and does not carry the reported numbers.

## Quick start

```powershell
python "lab 3/run_lab3_codec.py" --smoke
python "lab 3/run_lab3.py" --smoke
```

## Exit gates

- `MPS` (melodic preservation): cosine(`z_content`, `z_content'`) >= 0.90
- `SF` (stylistic fidelity): judge confidence in the target genre >= 0.85
- Spectral continuity: multi-resolution STFT score, lower is better

What each one means is in `docs/reference/metrics.md`; the values reached, and the artifacts behind
them, are in `docs/explanation/results.md`.

## Genre labels are dataset labels

Every public corpus in this project couples genre to dataset source, so a model can learn one
source's recording chain instead of a transferable style. Without new data, the countermeasure is to
keep each label bucket multi-source and balanced:

```powershell
python "lab 3/run_lab3_codec.py" `
  --genre-schema binary_acoustic_beats `
  --balance-sources-within-genre `
  --require-min-sources-per-genre 2 `
  --require-is-music
```

`scripts/run_codec_strong_schema_smoke.ps1` and `run_codec_strong_schema_full.ps1` wrap cache build,
style bank, training, and audit into one command; `scripts/run_codec_audit_latest.ps1` audits the
latest run for leakage.

For labels that are not source buckets at all, two unpaired options are provided: CLAP zero-shot
prompts (`lab 3/run_lab3_auto_genre.py`) and Lab 2-style clustering of `[z_style, descriptor32]`
(`lab 3/run_lab3_auto_genre_lab2cluster.py`).

## Run artifacts

Runs land in `saves2/lab3_synthesis/runN/`, with strict run naming by default: `run_state.json`,
`history.csv`, `checkpoints/`, `lab3_exit_audit.json`, and a `samples/posttrain_samples/` pack
exported on completion. Resume with `--mode resume --resume-dir <run_dir>`.

To audition the exports by hand, `lab 3/run_lab3_clip_picker.py` gives an accept/reject pass over
them (`a`, `r`, `s`, `o`, `q`).

## Reading more

- Design: `docs/explanation/lab3_codec_transfer.md`, `docs/explanation/lab3_diffusion.md`
- Flags: `docs/reference/cli.md` · Metrics: `docs/reference/metrics.md`
- Best-checkpoint recipes: `docs/howto/reproduce_best_runs.md`
- Final report: `DGGR-final-report.pdf`
