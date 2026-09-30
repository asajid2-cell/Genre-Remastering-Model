# Documentation (DGGR)

The tree follows Diataxis: tutorials teach, how-tos solve a task, reference states facts, and
explanation covers why the design is what it is. Pick by what you are trying to do.

## Start here

- [`tutorials/01_quickstart_longform.md`](tutorials/01_quickstart_longform.md) - run one end-to-end
  long-form remaster and read the coherence diagnostics it produces.
- [`howto/reproduce_best_runs.md`](howto/reproduce_best_runs.md) - the exact recipes behind the best
  checkpoints, including the knobs to turn when you hear warble.
- [`explanation/architecture.md`](explanation/architecture.md) - how the four stages fit together.

## Reference

- [`reference/cli.md`](reference/cli.md) - entry points and flags.
- [`reference/data_formats.md`](reference/data_formats.md) - manifests, caches, and run artifacts.
- [`reference/metrics.md`](reference/metrics.md) - what each gate metric means.
- [`reference/env_vars.md`](reference/env_vars.md) - `DGGR_DATA_ROOT`, `DGGR_MANIFESTS_ROOT`.

## Explanation

- [`explanation/results.md`](explanation/results.md) - measured outcomes, the artifact each comes
  from, and the known discrepancies between run files. Start here if you are checking a number.
- [`explanation/lab1_deconstruction_encoder.md`](explanation/lab1_deconstruction_encoder.md)
- [`explanation/lab2_target_vector_space.md`](explanation/lab2_target_vector_space.md)
- [`explanation/lab3_codec_transfer.md`](explanation/lab3_codec_transfer.md)
- [`explanation/lab3_diffusion.md`](explanation/lab3_diffusion.md)
- [`explanation/lab4_longform.md`](explanation/lab4_longform.md)
- [`explanation/codec_vs_diffusion.md`](explanation/codec_vs_diffusion.md)
- [`explanation/longform_coherence.md`](explanation/longform_coherence.md)

## Decisions

[`decisions/`](decisions/README.md) holds the architecture decision records, including why the
canonical package is `dggr/` and why `lab */src/` survives as shims.
