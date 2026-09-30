# Deep Generative Genre Remastering (DGGR)

An audio research project investigating genre transfer while preserving a track's melodic content, using codec-latent translation and diffusion.

[Listen to samples](examples/README.md) · [Final report (15 pages)](DGGR-final-report.pdf) · [Project proposal](DGGR-proposal.pdf)

## Samples

Seven output clips are included; no training or source audio is distributed, so these are not paired listening comparisons.

| Model | Outputs | Producing run |
|---|---|---|
| EnCodec latent translation | [Other → lo-fi](examples/audio/codec_run1055_sample0000_src1_tgt3.wav) · [Hip-hop → other](examples/audio/codec_run1055_sample0004_src2_tgt1.wav) · [Lo-fi → baroque](examples/audio/codec_run1055_sample0008_src3_tgt0.wav) | `run1055` |
| Diffusion V2 + BigVGAN | [Sample 1](examples/audio/diffusion_v2_run_d002_epoch006_00_gen.wav) · [Sample 2](examples/audio/diffusion_v2_run_d002_epoch006_01_gen.wav) · [Sample 3](examples/audio/diffusion_v2_run_d002_epoch006_02_gen.wav) · [Sample 4](examples/audio/diffusion_v2_run_d002_epoch006_03_gen.wav) | `run_d002`, epoch 6 |

[Sample metadata](examples/metadata.md) records model provenance and genre indices.

![Input and codec-transfer output spectrograms](docs/media/codec-transfer-spectrogram.png)

*An input from the `cc0_other` bucket (left) and its codec remaster into `baroque_classical` (right), reproduced from the final report.*

## Method

The pipeline has four stages:

1. **Deconstruction:** encode audio into separate 128-D content and style vectors, with an adversarial style probe and a music/non-music gate.
2. **Target space:** combine style vectors with 32 handcrafted descriptors to form 160-D genre targets.
3. **Synthesis:** translate frozen EnCodec latents to waveform, or generate log-mel spectrograms with v-prediction diffusion and vocode them with BigVGAN.
4. **Long-form processing:** remaster overlapping chunks with SDEdit-style diffusion, prefix locking, and periodic re-anchoring.

### Dataset confounding

Genre labels are correlated with dataset source: hip-hop, lo-fi, and baroque examples come from different collections. A style classifier can therefore reward recording characteristics rather than musical genre. The pipeline uses track-grouped splits, balanced sampling, and source-leakage audits to investigate this confound. The effect of the corrective measures on transfer quality has not been established.

The [final report](DGGR-final-report.pdf) describes the architecture and experiments; [Lab 2](lab%202/README.md) and [Lab 3](lab%203/README.md) document the target-space and synthesis paths.

## Results

The [proposal](DGGR-proposal.pdf) records these targets before training. These are automated evaluations, not listening-test results.

| Stage | Metric | Target | Reported result |
|---|---|---|---|
| Deconstruction | Style-probe accuracy | ≥ 0.85 | 0.9417 |
| Deconstruction | Content leakage above chance | ≤ 0.15 | 0.1083* |
| Deconstruction | Music-gate ROC-AUC | ≥ 0.90 | 0.9299 |
| Target space | Silhouette (cosine) | ≥ 0.45 | 0.4939 |
| Synthesis (codec) | Melodic preservation score | ≥ 0.90 | 0.9565 |
| Synthesis (codec) | Target-style confidence | ≥ 0.85 | 0.8940 |

*Content leakage is probe accuracy above the 0.500 chance baseline. The selected audit reports 0.1083; the same run's preflight reports 0.2125, which fails the target. These measurements do not establish complete content/style disentanglement.*

[Metric definitions and run provenance](docs/explanation/results.md) detail the evaluations and artifact discrepancies. The codec run also records a script-default style threshold of 0.4 rather than the proposal's 0.85; its achieved confidence of 0.8940 clears both.

## Limitations

- **No perceptual validation:** the proposed blind A/B listening study and Fréchet Audio Distance evaluation were not run. Classifier scores do not establish that listeners perceive the target genre.
- **Dataset-source confounding remains unresolved:** leakage audits investigate it, but the project's results do not demonstrate that it has been eliminated.
- **Long-form output accumulates warble:** a 160-second run across 64 chunks reports mean boundary mel MSE of 0.0018347 and discontinuity of 2.87 dB. These boundary measurements do not establish perceptual stability. `--t-start`, `--source-mel-blend`, and `--reanchor-every` control the edit/stability tradeoff.
- **Lower diffusion loss did not track informal listening judgments:** validation loss fell from 0.0442 at epoch 6 to 0.0386 at epoch 18, but epoch 6 was selected for perceived quality without a controlled listening study.

## Reproduction

**Checkpoints and training corpora are not distributed.** Running inference requires a compatible trained checkpoint; reproducing training also requires the datasets and manifests. This is not a ready-to-run pretrained demo.

- [Installation and environment setup](docs/howto/01_environment_setup.md)
- [Recipes for the reported codec, diffusion, and long-form runs](docs/howto/reproduce_best_runs.md)
- [Command-line reference](docs/reference/cli.md)

## Code

| Area | Entry point |
|---|---|
| Deconstruction encoder | [`notebooks/01_lab1_deconstruction_encoder.ipynb`](notebooks/01_lab1_deconstruction_encoder.ipynb) |
| Target-vector space | [`dggr/lab2_pipeline.py`](dggr/lab2_pipeline.py) |
| Codec-latent transfer | [`dggr/lab3_codec_train.py`](dggr/lab3_codec_train.py) |
| Style judge and metrics | [`dggr/lab3_codec_judge.py`](dggr/lab3_codec_judge.py) |
| Diffusion synthesis | [`dggr/lab3_diffusion_train.py`](dggr/lab3_diffusion_train.py) |
| Long-form processing | [`lab 3/run_lab4_longform_coherence.py`](lab%203/run_lab4_longform_coherence.py) |
| Source-leakage audit | [`lab 3/run_lab3_quality_audit.py`](lab%203/run_lab3_quality_audit.py) |

## Authors and license

Built for CMPUT 414 at the University of Alberta, Winter 2026, in a group of three. Code and audits are by Ahmed Sajid; the report and proposal are co-authored with Sahara Kaul and Kelsey Pattison.

[MIT](LICENSE) covers the code and documentation. The two co-authored PDFs are excluded from that grant; see [NOTICE](NOTICE).
