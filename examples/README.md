# Examples

Ten 5-second clips, committed so the project can be heard without running anything, and playable on
the [demo page](https://asajid2-cell.github.io/Genre-Remastering-Model/).

| Track | Clips | Producing run |
|---|---|---|
| Codec transfer | `audio/codec_run1055_sample0000_*`, `audio/codec_run1055_sample0018_*` (with inputs) | `saves2/lab3_codec_transfer/run1055` |
| Codec transfer | `audio/codec_run1055_sample0004_src2_tgt1.wav`, `audio/codec_run1055_sample0008_src3_tgt0.wav` (outputs only) | `saves2/lab3_codec_transfer/run1055` |
| Diffusion V2, epoch 6 | `audio/diffusion_v2_run_d002_epoch006_0*_gen.wav` | `saves2/lab3_diffusion/run_d002` |

Two clips ship with their inputs, both CC0 and therefore redistributable here: `sample0000_source.wav`
is a Freesound field recording, and `sample0018_source.wav` is Komiku's "Level 9". The other six are
outputs alone — their inputs come from the XTc Files of Hip Hop sample library and from research
corpora that are not redistributed.

[`metadata.md`](metadata.md) records the model family behind each clip, the run it came from, and the
`srcX_tgtY` genre-index convention.

No training corpora or checkpoints are included. The recipes that reproduce each checkpoint are in
[`docs/howto/reproduce_best_runs.md`](../docs/howto/reproduce_best_runs.md).
