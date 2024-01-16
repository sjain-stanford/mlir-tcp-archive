Tensor Compute Primitives
=========================

Mid-level intermediate representation for machine learning programs.

[![Bazel Build and Test (mlir-tcp)](https://github.com/cruise-automation/mlir-tcp/actions/workflows/bazelBuildAndTestTcp.yml/badge.svg)](https://github.com/cruise-automation/mlir-tcp/actions/workflows/bazelBuildAndTestTcp.yml)

:construction: **This project is under active development (WIP).**

## Project Communication

- For general discussion use `#mlir-tcp` channel on the [LLVM Discord](https://discord.gg/xS7Z362)
- For feature request or bug report file a detailed [issue on GitHub](https://github.com/cruise-automation/mlir-tcp/issues)

## Developer Guide

To build TCP using Bazel, follow these steps:

1. (Optional) For a quick start, launch an interactive docker container with clang (and lld) pre-installed:
```shell
./docker/run_docker.sh
```

2. You can now build `tcp-opt` by running:
```shell
bazel build --config=clang_linux //:tcp-opt
```
(replace `linux` with `osx` for Mac)

3. To run TCP lit and aot compile tests:
```shell
bazel test --config=clang_linux //test/...
```

We welcome contributions to `mlir-tcp`. If you do contribute, please finalize your PR with clang-format and bazel buildifier to ensure the C++ sources and BUILD files are formatted consistently:
```shell
# clang-format
find . -type f -name "*.cpp" -o -name "*.h" | xargs clang-format -i

# buildifer
bazel run --config=clang_linux //:buildifier
```

When bumping upstream dependencies (LLVM, Torch-MLIR, StableHLO), you may validate the set of "green commits" by running the corresponding third-party tests:
```shell
bazel test --config=clang_linux @llvm-project//mlir/...
bazel test --config=clang_linux @torch-mlir//...
bazel test --config=clang_linux @stablehlo//...
```

The following CI workflows are automatically triggered anytime upstream dependencies (`deps.bzl`) are updated:
- [![Bazel Build and Test (llvm-project)](https://github.com/cruise-automation/mlir-tcp/actions/workflows/bazelBuildAndTestLlvm.yml/badge.svg)](https://github.com/cruise-automation/mlir-tcp/actions/workflows/bazelBuildAndTestLlvm.yml)
- [![Bazel Build and Test (torch-mlir)](https://github.com/cruise-automation/mlir-tcp/actions/workflows/bazelBuildAndTestTorchmlir.yml/badge.svg)](https://github.com/cruise-automation/mlir-tcp/actions/workflows/bazelBuildAndTestTorchmlir.yml)
- [![Bazel Build and Test (stablehlo)](https://github.com/cruise-automation/mlir-tcp/actions/workflows/bazelBuildAndTestStablehlo.yml/badge.svg)](https://github.com/cruise-automation/mlir-tcp/actions/workflows/bazelBuildAndTestStablehlo.yml)

## Debugging Guide

Debug prints
```C++
llvm::errs() << "Operand Type: " << operandType << "\n";
op.emitWarning() << "Operand Type: " << operandType << "\n";
```

```shell
bazel build --config=clang_linux //:tcp-opt
bazel-bin/tcp-opt --some-pass foo.mlir
```


https://llvm.org/docs/ProgrammersManual.html#the-llvm-debug-macro-and-debug-option
```C++
#include "llvm/Support/Debug.h"
#define DEBUG_TYPE "mydebugtag"

LLVM_DEBUG(llvm::dbgs() << "This log will only show up when -debug or -debug-only=mydebugflag flags are present\n");
```

GDB
```shell
bazel build --config=clang_linux --config=gdb //:tcp-opt
```
Then run:

$ gdb --args bazel-bin/tcp-opt 

For help with gdb commands please refer to this gdb cheat sheet.
https://gist.github.com/rkubik/b96c23bd8ed58333de37f2b8cd052c30

https://mlir.llvm.org/getting_started/Debugging/


LLVM Symbolizer
If you get a stack dump upon crash like this:
```
Stack dump without symbol names (ensure you have llvm-symbolizer in your PATH or set the environment var `LLVM_SYMBOLIZER_PATH` to point to it):
0  tcp-opt   0x000055ac1c9c0c1d
1  tcp-opt   0x000055ac1c9c110b
2  tcp-opt   0x000055ac1c9be846
3  tcp-opt   0x000055ac1c9c1855
4  libc.so.6 0x00007f7011c6a520
```

```shell
bazel build --config=clang_linux @llvm-project//llvm:llvm-symbolizer
export LLVM_SYMBOLIZER_PATH=`pwd`/bazel-bin/external/llvm-project/llvm/llvm-symbolizer
```