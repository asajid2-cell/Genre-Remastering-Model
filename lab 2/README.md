# Lab 2 - Genre Target Vector Space

Lab 2 turns the frozen Lab 1 encoder into a style-harvesting system. Each clip becomes a 160-D
target vector - 128-D `z_style` from Lab 1 concatenated with 32 handcrafted log-mel descriptors -
and the per-genre centroids of those vectors are the style blueprint Lab 3 conditions on.

## Run it

```powershell
python "lab 2/run_lab2.py" --smoke
python "lab 2/run_lab2.py" --per-genre-samples 1200
```

The encoder defaults to `saves/lab1_run_combo_af_gate_exit_v2/latest.pt`; override it with
`--checkpoint`.

## Inputs

Four genre buckets - `baroque_classical`, `hiphop_xtc`, `lofi_hh_lfbb`, `cc0_other` - built from
four cleaned manifests (`xtc_audio_clean.csv`, `hh_lfbb_audio_clean.csv`, `cc0_audio_clean.csv`, and
optionally `phase1_symbolic_audio_manifest.csv`). They are read from `DGGR_MANIFESTS_ROOT`, falling
back to `Z:/DataSets/_lab1_manifests` or `data/_lab1_manifests`.

Neither the manifests nor the source audio ship with this repository, so a bare clone cannot run
this stage; you need the corpora and the Lab 1 checkpoint first (`docs/howto/02_run_lab2.md`).

## Outputs

Written to `saves/lab2_calibration/<timestamp>/`:

- `validation_summary.json`, `lab2_exit_checklist.json`
- `centroids_160d.csv`, `target_centroids.json`, `centroid_distances.csv`
- `embeddings.npz`, `embeddings_index.csv`, `genre_samples.csv`
- `vector_variance_report.csv`, `inter_centroid_separation.csv`,
  `centroid_stability_trials.csv`, `centroid_stability_report.json`
- `neighbor_audit.csv`, `neighbor_audit_summary.csv`
- `global_genre_map_tsne.csv`, `global_genre_map_tsne.png`

## Gates

Linear-probe genre accuracy, nearest-centroid accuracy, silhouette on cosine distance, pairwise
centroid separation at >= 3 sigma, and 5/5 neighbour hits per genre. Thresholds are set on the CLI
(`--silhouette-threshold`, `--sigma-multiplier`, `--neighbor-top-k`); what each metric means is in
`docs/reference/metrics.md`, and the values reached are in `docs/explanation/results.md`.

## Re-warping an existing harvest

A supervised projection can be applied to embeddings you already harvested, without re-running the
encoder:

```powershell
python "lab 2/run_lab2.py" `
  --reuse-artifacts-dir "saves/lab2_calibration/lab2_20260211_015118" `
  --projection lda `
  --zstyle-weight 2.0 `
  --descriptor-weight 1.0 `
  --output-dir "saves/lab2_calibration/lab2_20260211_015118_lda"
```

## Reading more

- Design: `docs/explanation/lab2_target_vector_space.md`
- Flags: `docs/reference/cli.md` · Metrics: `docs/reference/metrics.md`
- Final report: `DGGR-final-report.pdf`

## Note

`FPR@0.5` from Lab 1 is a threshold-calibration artifact, not an embedding-rank problem.
