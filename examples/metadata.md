# Example Metadata

## Genre index convention

- 0: `baroque_classical`
- 1: `cc0_other`
- 2: `hiphop_xtc`
- 3: `lofi_hh_lfbb`

Filenames read `srcX_tgtY`: source genre index `X`, target genre index `Y`. `cc0_other` is the bucket
of CC0-licensed material whose content does not fit the three named genres.

## Codec transfer examples

Run: `saves2/lab3_codec_transfer/run1055`. Model family: EnCodec latent translation with the EnCodec
decoder. Every clip is 5.0 s, mono, 24 kHz, 16-bit PCM.

| Input | Remaster | Direction | Input shipped |
|---|---|---|---|
| `audio/codec_run1055_sample0000_source.wav` | `audio/codec_run1055_sample0000_src1_tgt3.wav` | `cc0_other` → `lofi_hh_lfbb` | yes |
| `audio/codec_run1055_sample0018_source.wav` | `audio/codec_run1055_sample0018_src1_tgt2.wav` | `cc0_other` → `hiphop_xtc` | yes |
| — | `audio/codec_run1055_sample0004_src2_tgt1.wav` | `hiphop_xtc` → `cc0_other` | no |
| — | `audio/codec_run1055_sample0008_src3_tgt0.wav` | `lofi_hh_lfbb` → `baroque_classical` | no |

The two shipped inputs are CC0:

- `sample0000` — `493858__bashrambali__atmo-antigua_semana-santa`, Freesound.
- `sample0018` — Komiku, "Level 9 — One more step till the fall of patriarchy".

The two inputs that are not shipped are from the XTc Files of Hip Hop sample library and from the
project's lo-fi beat corpus.

## Diffusion examples

Files: `audio/diffusion_v2_run_d002_epoch006_0{0..3}_gen.wav`

Run: `saves2/lab3_diffusion/run_d002`, epoch 6 — the checkpoint kept for perceived quality. Model
family: v-prediction log-mel diffusion with BigVGAN vocoding. Clips are 2.97 s, mono, 22.05 kHz,
16-bit PCM. Target genre `hiphop_xtc`.

The run's reference clips are XTC recordings and are not redistributed, so these are generated
outputs with no comparison included. The same run's validation loss keeps falling past epoch 6 while
perceived quality does not; see the README's Limitations section.
