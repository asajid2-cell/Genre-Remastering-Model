# Deep Generative Genre Remastering (DGGR)

**A four-stage audio pipeline that separates a track into a content vector and a style vector, then
rebuilds it in a target genre. Every stage is graded against thresholds that were written into the
project proposal before any model was trained.**

**[Listen to the remasters](https://asajid2-cell.github.io/Genre-Remastering-Model/)** — five-second
source/remaster pairs and diffusion samples, playable in the browser.
[Final report, 15 pp](DGGR-final-report.pdf) ·
[Proposal, where the targets are fixed](DGGR-proposal.pdf)

![An input spectrogram beside its remaster](docs/media/codec-transfer-spectrogram.png)

*Input from the `cc0_other` bucket on the left, the codec track's remaster of it into
`baroque_classical` on the right. From the [final report](DGGR-final-report.pdf).*

## Samples

Ten clips live in [`examples/audio/`](examples/audio) and play on the
[demo page](https://asajid2-cell.github.io/Genre-Remastering-Model/): eight remasters and the two CC0
inputs they were made from. Two remasters ship with their input, so the change is audible rather than
asserted:

- a CC0 field recording remastered into `lofi_hh_lfbb`
- a CC0 chiptune track remastered into `hiphop_xtc`

The other six are outputs alone. Their inputs come from a commercial sample pack and from research
corpora this repository does not redistribute. [`examples/metadata.md`](examples/metadata.md) records
the run behind each clip.

## Method

Four stages, each consuming the one above.

1. **Deconstruction.** An encoder splits audio into a 128-D content vector and a 128-D style vector
   under an adversarial style probe, with a music/non-music gate.
2. **Target space.** Style vectors from four corpora are concatenated with 32 handcrafted descriptors
   into a 160-D target vector; per-genre centroids become the conditioning signal.
3. **Synthesis.** A codec track translates frozen EnCodec latents straight to waveform; a diffusion
   track generates log-mel with v-prediction and vocodes it with BigVGAN.
4. **Long-form.** Chunked SDEdit-style remastering of a whole track, with prefix-overlap locking and
   periodic re-anchoring.

The [final report](DGGR-final-report.pdf) carries the architecture, and
[`docs/explanation/`](docs/explanation) has a page per stage.

### Dataset confounding

Genre and dataset source are the same variable here. XTC hip-hop, FMA lo-fi and PD-symbolic baroque
arrive from different collections, so a model that appears to transfer style may be learning one
collection's recording chain instead. The pipeline answers with track-ID-grouped splits, multi-source
genre buckets, a leakage audit that re-derives labels from the audio rather than trusting the corpus,
and a style judge built on MERT embeddings. How much of the gap that closes is not settled, and the
audits disagree with each other; see Limitations.

## Results

Targets were fixed in the [proposal](DGGR-proposal.pdf) on 2026-02-10 and did not move when the
results arrived. Every number here is an automated evaluation.

| Stage | Metric | Target | Achieved |
|---|---|---|---|
| Deconstruction | style-probe accuracy | ≥ 0.85 | **0.9417** |
| Deconstruction | content leakage above chance | ≤ 0.15 | **0.1083** |
| Deconstruction | music-gate ROC-AUC | ≥ 0.90 | **0.9299** |
| Target space | silhouette (cosine) | ≥ 0.45 | **0.4939** |
| Synthesis (codec) | melodic preservation | ≥ 0.90 | **0.9565** |
| Synthesis (codec) | target-style confidence | ≥ 0.85 | **0.8940** |

Leakage is the content probe's accuracy above the 0.500 chance baseline, and it is the number that
matters most: it is what separates style from content rather than letting the encoder memorize both.
[`docs/reference/metrics.md`](docs/reference/metrics.md) defines the rest, and
[`docs/explanation/results.md`](docs/explanation/results.md) traces where each number came from.

## Limitations

**The perceptual leg was designed and never run.** The proposal's Lab 5, a blind A/B listening test
plus Fréchet Audio Distance, is the only stage that measures whether a remaster sounds like its
target to a person. Every quality claim above therefore rests on a trained classifier and spectral
diagnostics, and the report's Threats to Validity says the same thing.

That distinction has a price. On the diffusion track, validation loss keeps falling to 0.0386 at
epoch 18, while the checkpoint kept is epoch 6, at 0.0442. Lower loss was not better audio, and
without the listening test there is no measurement that can tell the two apart.

![Training and validation loss over 18 epochs](docs/media/diffusion-loss-curve.png)

- **Long-form drifts past ~160 s.** Boundary statistics are clean over 64 chunks (mean mel MSE
  0.0018347, 2.87 dB mean discontinuity), but what accumulates is warble, not seams. `--t-start`,
  `--source-mel-blend` and `--reanchor-every` trade edit freedom for stability.
- **Two artifacts disagree with the numbers printed beside them.** The selected deconstruction audit
  reports leakage 0.1083; the same run's preflight reports 0.2125 and fails its own gate. The best
  codec run also writes a 0.4 style threshold, the script default, where the design target is 0.85.
  Both achieved values clear the design targets, so no claim rests on either file, but they are in
  the repository. [`docs/explanation/results.md`](docs/explanation/results.md) itemizes them.

## Reproduction

**No checkpoints and no corpora ship with this repository.** It is the code, the report and the
sample clips, not a runnable pretrained demo. Inference needs a compatible checkpoint; retraining
needs the datasets and their manifest CSVs.

```bash
python -m pip install -r requirements.txt
cp .env.example .env      # set DGGR_DATA_ROOT and DGGR_MANIFESTS_ROOT
```

From there: [`docs/howto/01_environment_setup.md`](docs/howto/01_environment_setup.md) for the
environment, [`docs/howto/reproduce_best_runs.md`](docs/howto/reproduce_best_runs.md) for the recipes
that produced each reported checkpoint, and [`docs/reference/cli.md`](docs/reference/cli.md) for the
entry points.

## Code

| Area | Path |
|---|---|
| Deconstruction encoder | `notebooks/01_lab1_deconstruction_encoder.ipynb` |
| Target-vector space | `dggr/lab2_pipeline.py` |
| Codec-latent transfer | `dggr/lab3_codec_train.py`, `lab 3/run_lab3_codec.py` |
| Style judge and metrics | `dggr/lab3_codec_judge.py` |
| Diffusion branch | `dggr/lab3_diffusion_train.py` |
| Long-form coherence | `lab 3/run_lab4_longform_coherence.py` |
| Source-leakage audit | `lab 3/run_lab3_quality_audit.py` |

`dggr/` is the canonical package. `lab 2/src/` and `lab 3/src/` hold one-line re-export shims so the
original run scripts' imports keep working.

## Authors and licence

Built for CMPUT 414 at the University of Alberta, Winter 2026, in a group of three. The code, this
repository and the audits are mine; the report and the proposal are co-authored with Sahara Kaul and
Kelsey Pattison. [MIT](LICENSE) covers the code and documentation, and the two co-authored PDFs are
carved out in [`NOTICE`](NOTICE).
