# llama.cpp on Direct3D 11

This repository is llama.cpp with a native **Direct3D 11** compute backend (and the Direct3D 12
backend it shares its kernels with). It runs on any Windows GPU whose driver supports D3D11
feature level 11_0 - no CUDA, no ROCm, no Vulkan, no vendor SDK.

Status: experimental.

## How it works

The kernels are HLSL (in `ggml/src/ggml-d3d12/hlsl/`), embedded as source in the DLL and compiled
at runtime to `cs_5_0` with `d3dcompiler_47.dll`, which ships with Windows. Compiled shaders are
cached on disk. Ops the backend does not support run on the CPU.

## Build

Cross compile from Linux with mingw-w64 (the tested build):

    cmake -B build-win -DCMAKE_TOOLCHAIN_FILE=cmake/x86_64-w64-mingw32.cmake \
          -DGGML_D3D11=ON -DGGML_BACKEND_DL=ON -DGGML_CPU_ALL_VARIANTS=ON
    cmake --build build-win -j

Native Windows (not tested yet):

    cmake -B build -DGGML_D3D11=ON -DGGML_BACKEND_DL=ON -DGGML_CPU_ALL_VARIANTS=ON
    cmake --build build --config Release

Add `-DGGML_D3D12=ON` to build the Direct3D 12 backend as well.

## Run

The D3D11 backend is off unless asked for, so a machine with both backends keeps using D3D12:

    set GGML_D3D11_ENABLE=1
    llama-cli -m model.gguf -ngl 99

`GGML_D3D12_DISABLE=1` also turns D3D11 on, in place of D3D12.

## Tested

`test-backend-ops` (all op groups) on AMD Radeon AI PRO R9700 and Moore Threads MTT S80;
flash attention and matrix multiplication on Intel Iris Xe.

Known limits: flash attention with head size 192 or more is left to the CPU on AMD GPUs;
some quantized KV cache types are not supported in flash attention.

## Upstream

Everything else is upstream llama.cpp: <https://github.com/ggml-org/llama.cpp>. Its own README
is `README.md` in the repository root.
