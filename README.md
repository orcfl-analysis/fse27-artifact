# Replication package (anonymized for review)

A whole-program flows-to analysis for indirect-call resolution over
LLVM IR, its Lean model, and the scripts that reproduce the paper's
numbers. Everything external is fetched from pinned sources and
verified (git commits by SHA, files by sha256): the Linux 5.18 source
(kernel.org), the SoK-MLTA dataset and scripts, SVF, Lotus, GraCFL and
the Graspan graphs, and the APR libraries. Nothing is prebuilt.

## Layout
- `src/` the analysis (C++17, LLVM 18); `src/gracfl/` bundled CFL engine
- `func_summaries.txt` library transfer summaries
- `proof/lean/` Lean model and proofs (`lake build`); `GAPS.md` lists
  the open obligations
- `eval/` experiment scripts, numbered by stage; each documents its
  inputs, outputs and environment variables in its header
- `tools/` scoring (`sok-persite.py`, `sok-report.py`) and the kernel
  ground-truth matcher (`gt-match.py`)
- `test/cfl-smoke.sh` smoke test

## Build
    make BUILD_DIR=release LLVM_BUILD=/usr/lib/llvm-18
    test/cfl-smoke.sh            # expect four PASS lines

## Reproduce
Kernel (Linux 5.18, x86-64 defconfig + Clang ThinLTO):

    eval/59-kernel-corpus.sh           # kernel.org source -> bitcode corpus
    KA_KERNEL_BCLIST=$KA_WORK/linux-5.18.bclist eval/60-kernel-fse.sh
    KA_KERNEL_BCLIST=$KA_WORK/linux-5.18.bclist eval/61-kernel-fs-endpoints.sh

The kernel recall numbers use fuzzing call traces from a third party,
which we cannot redistribute. `eval/60-kernel-fse.sh` runs without
them; with a trace file in the same format, set `KA_GT=<file>`.

User-space programs (httpd, PostgreSQL):

    eval/10-fetch-corpora.sh && eval/20-build-corpora.sh && eval/64-usermode-fse.sh

SoK-MLTA comparison with all baselines (needs docker):

    export KA_BIGBOX_WORK=/big/disk/sok
    eval/69-sok-bigbox.sh setup        # dataset, SoK scripts, SVF, Lotus, APR
    eval/69-sok-bigbox.sh all          # -> $KA_BIGBOX_WORK/report.md

CFL engines on the Graspan graphs:

    eval/63-gracfl-graspan.sh

`eval/env.sh` holds the shared defaults (`KA_WORK` and toolchain).
