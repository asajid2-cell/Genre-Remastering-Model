# Observed Results (Labs 1-4)

Best quantitative outcomes, with the artifact each one comes from. Run outputs live under `saves/`
and `saves2/`, which are gitignored — this page records the paths so a number can be traced back to
the file that produced it.

## Lab 1 - Deconstruction Encoder

Checkpoint: `saves/lab1_run_combo_af_gate_exit_v2/latest.pt`
Artifacts: `saves/lab1_run_combo_af_gate_exit_v2/audits_confidence/`

| Metric | Value | Threshold |
|---|---|---|
| `style_probe_accuracy` | 0.9417 | >= 0.85 |
| `content_leakage_above_baseline` | 0.1083 | <= 0.15 |
| `gate ROC-AUC` | 0.9299 | >= 0.90 |

`z_style` is strongly style-informative, `z_content` is substantially style-suppressed, and the music
gate ranks music against non-music reliably. Leakage is the content probe's accuracy above its 0.500
chance baseline, measured on 600 embeddings balanced strictly 300/300 across source classes.

## Lab 2 - Target Vector Space

Run: `saves/lab2_calibration/lab2_20260211_015118_lda_cleanup_v2/validation_summary.json`

| Metric | Value | Threshold |
|---|---|---|
| `silhouette` (cosine) | 0.4939 | >= 0.45 |
| `linear_probe_acc` | 0.8554 | - |
| `nearest_centroid_acc` | 0.8514 | - |

4,011 embeddings over 4 genres, LDA projection. The 160-D space is separable by genre and usable as a
conditioning blueprint, which is all Lab 3 needs from it.

## Lab 3 - Codec Latent Translation

Best run: `saves2/lab3_codec_transfer/run1055`
Gate metrics: `codec_gate_eval.json`

| Metric | Value | Design target |
|---|---|---|
| `mps` (melodic preservation) | 0.9565 | >= 0.90 |
| `style_conf` | 0.8940 | >= 0.85 |
| `style_acc` | 0.9492 | >= 0.85 |

This is the strongest synthesis path in the project.

## Lab 3/4 - Diffusion Branch

Run: `saves2/lab3_diffusion/run_d002`

- Best validation loss (epoch 18): **0.0386**
- Selected quality checkpoint (epoch 6): **0.0442**

Later epochs improve the number while perceived quality peaks earlier, which is mode averaging
rather than progress. Loss is not a proxy for the thing the report is judged on.

## Lab 4 - Long-Form Coherence

Source: `saves2/lab4_longform_coherence/fullsong_test/coherence_metrics.json`

| Metric | Value |
|---|---|
| `n_chunks` | 64 |
| `duration_sec` | 160.0 |
| `boundary_mel_mse_mean` | 0.0018347 |
| `boundary_mel_mse_p95` | 0.0051191 |
| `boundary_disc_db_mean` | 2.8691 |
| `boundary_disc_db_p95` | 4.9909 |

Boundary metrics give a handle for coherence tuning. What remains is accumulating warble and static
rather than hard seams, so these numbers bound the seam problem without bounding the perceptual one.

## Provenance and known discrepancies

Four places where a number above and a file in the repo do not say the same thing. Each is recorded
here so it is found deliberately rather than accidentally.

**Leakage has two values in one run.** `exit_run_summary.json` reports `content_leakage_above_baseline`
as 0.1083 in its `confidence` block and 0.2125 in its `preflight` block, where it also sets
`leakage_ok: false` and `pass: false`. The preflight is a coarse gate over a smaller slice; the
confidence audit is the balanced 300/300 measurement, and it is the source of the 0.1083 used above
and in the report. The same file disagrees on two other Lab 1 metrics for the same reason:
style-probe accuracy is 0.9417 (confidence) against 0.95 (preflight), and gate AUC is 0.9299 against
0.9395.

**The style gate was configured below its design value.** `lab 3/run_lab3_codec.py` defaults
`--gate-min-style-conf` and `--gate-min-style-acc` to 0.40, so run1055's `codec_gate_eval.json`
records 0.4 while the proposal specifies 0.85. The achieved 0.8940 and 0.9492 clear both, so the
threshold a reader sees in that file is the script default rather than the project target.

**The music gate ranks well but is not calibrated at its operating point.** `gate_summary.json` gives
0.9299 ROC-AUC with `fpr_pass: false`: at threshold 0.5 the false-positive rate is 0.136, and reaching
the 2% target needs a threshold of 0.890, which drops music recall to 0.558. The ranking is the useful
part; the threshold is an unresolved tradeoff.

**No perceptual or distributional metric exists.** The proposal's Lab 5 — a blind A/B listening test
plus Frechet Audio Distance — was designed and never run. Every quality claim in this repository rests
on a trained classifier and spectral diagnostics rather than listeners.
