# References and Attributions

Where the documentation structure and templates came from.

## Documentation structure

- Diataxis (tutorials / how-to / reference / explanation): https://diataxis.fr/ (Daniele Procida).
  Gives the information architecture for `docs/`.

## Documentation templates

- The Good Docs Project: https://thegooddocsproject.dev/ and
  https://github.com/thegooddocsproject/templates. Source of the tutorial, how-to, reference, and
  explanation page shapes.

## Decision records

- MADR (Markdown Architectural Decision Records): https://adr.github.io/madr/. Format used in
  `docs/decisions/`.

## Models and data

- BigVGAN (`nvidia/bigvgan_v2_22khz_80band_256x`) - mel vocoder used by the diffusion branch.
- EnCodec - waveform codec whose latents the codec track translates.
- MERT - embedding model behind the style judge.
- Source corpora and their licences are recorded in the manifest CSVs rather than here.
