# tollgate-native

Prebuilt `llama-server` binaries for [Tollgate](https://github.com/Vibhinn/Tollgate)'s
local routing-intelligence classifier. Kept in a separate repo because these
are large, platform-specific compiled artifacts, not source code that
belongs alongside Tollgate's own history.

## What this builds

`llama-server` (from [ggerganov/llama.cpp](https://github.com/ggerganov/llama.cpp)),
compiled with `GGML_CPU_ALL_VARIANTS` - one CPU-optimized kernel per known
microarchitecture, with runtime dispatch picking the best match for whatever
machine actually runs it. One build per (OS, architecture) pair covers every
CPU generation within that pair at native performance - no per-machine
compilation needed, and no "-march=native tied to one build machine" crash
risk on different hardware. See `.github/workflows/build-llama-server.yml`
for the exact flags.

Currently covers: macOS arm64 (with Metal GPU offload), Linux x86_64, Linux
arm64, Windows x86_64.

## Releasing a new build

1. If bumping the pinned llama.cpp commit or changing build flags, update
   `LLAMA_CPP_REF` (and anything else) in the workflow first, and validate
   the change locally before tagging - a bad pin here breaks Tollgate's
   classifier for everyone who fetches it.
2. `git tag llama-server-v<N>` (increment N) and `git push --tags`.
3. The workflow builds all four platforms and attaches each archive to the
   GitHub Release created for that tag automatically.

`workflow_dispatch` is also enabled for a manual test run without cutting a
real tag.

## How Tollgate consumes this

Tollgate's setup step downloads the archive matching the running machine's
platform from this repo's latest release, extracts it into
`src/app/intelligence/server/`, and spawns the binary as a subprocess. See
`cli/steps/download_server.py` in the Tollgate repo.
