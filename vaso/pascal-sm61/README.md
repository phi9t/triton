# Pascal SM61 patch set

This directory maintains the exact source patch used for the Vaso Pascal
Triton build. Apply it to the pinned Triton **3.8.0** source revision
`675c59878aa2280b31f722aaf42b825fcee21de8`.

The `support/pascal-sm61` branch carries this patch set on top of the fork's
current upstream main. The compiler sources elsewhere on this support branch
are not the qualified Pascal source, and current upstream main has not been
qualified by these results. The original patch commit ID in the manifest
records the patch's provenance; it is not a published Pascal branch reference.

## Apply the patch

From a checkout of this support branch, create a separate build worktree at
the pinned source, then apply the exported commit:

```sh
patch_file="$(git rev-parse --show-toplevel)/vaso/pascal-sm61/0001-Fix-signed-INT8-dot-promotion-on-non-MMA-CUDA-targets.patch"
git worktree add -b build/pascal-sm61 ../triton-pascal-qualified \
  675c59878aa2280b31f722aaf42b825fcee21de8
git -C ../triton-pascal-qualified am "$patch_file"
```

The patch corrects signed INT8-to-INT32 operand promotion on the scalar FMA
path to use integer sign extension. It also supplies a regression covering
negative inputs, a nonzero accumulator, two tile shapes, four/eight warps,
and all visible SM61 devices. This is a correctness fix; it does not claim
a new DP4A optimization.

## Build and qualification identity

`manifest.json` records the source revision, exact exported patch, pinned
LLVM revision, CUDA line, Python ABI, and the qualified source wheel's SHA-256.
The wheel itself is not included here. Build and qualification use the Vaso
toolchain/dependency closure inside standalone Insula; this directory alone
is not a complete capsule assembly recipe.

The recorded source wheel was built with LLVM
`4611156032e1ebac68b2b8f6107b1475a7c60800`, CUDA 12.6, and CPython 3.12.
Both NVIDIA and AMD compiler backends are retained; Proton is disabled.
Runtime JIT qualification uses CUDA 12.6 ptxas, libdevice, and headers from
the capsule plus the admitted native NVIDIA driver projection.

The completed acceptance scope includes source-wheel compiler regressions,
Inkling operator and activation-quantization probes, and a reduced
random-weight Inkling text model's prefill and cached decoding on two
GTX 1070 Ti GPUs. These results do not establish full production checkpoint,
vision, or MTP inference coverage.

Apply future fixes against the pinned source deliberately, update the
manifest with the resulting artifacts, and requalify the changed code on
the intended hardware before changing the maintained source revision.
