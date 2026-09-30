# Examples

Seven generated clips, committed so the project can be heard without running anything: three codec
transfers and four diffusion samples.

| Track | Clips | Producing run |
|---|---|---|
| Codec transfer | `audio/codec_run1055_sample000*_src*_tgt*.wav` | `saves2/lab3_codec_transfer/run1055` |
| Diffusion V2, epoch 6 | `audio/diffusion_v2_run_d002_epoch006_0*.wav` | `saves2/lab3_diffusion/run_d002` |

[`metadata.md`](metadata.md) records the model family behind each clip, the run it came from, and the
`srcX_tgtY` genre-index convention.

No source audio, training corpora, or checkpoints are included. The recipes that reproduce each
checkpoint are in [`docs/howto/reproduce_best_runs.md`](../docs/howto/reproduce_best_runs.md).
