# llama.cpp on Direct3D 11 and Direct3D 12

This repository is llama.cpp with two native Windows GPU backends that share one set of HLSL
kernels (`ggml/src/ggml-d3d12/hlsl/`). Neither needs CUDA, ROCm, Vulkan or a vendor SDK.

| | Direct3D 12 | Direct3D 11 |
|---|---|---|
| needs | D3D12 feature level 11_0, shader model 6.0 | D3D11 feature level 11_0 |
| shader compiler | DXC (`dxcompiler.dll` + `dxil.dll`, shipped next to the DLL) | FXC (`d3dcompiler_47.dll`, part of Windows) |
| default | on | on in a D3D11-only build; with D3D12 built too, off until `GGML_D3D11_ENABLE=1` |
| for | most GPUs, the faster of the two | GPUs whose D3D12 driver is missing or broken |

Status: experimental. Ops a backend does not support run on the CPU. Compiled shaders are cached
on disk next to the backend DLL.

## Build

Cross compile from Linux with mingw-w64 (the tested build):

    cmake -B build-win -DCMAKE_TOOLCHAIN_FILE=cmake/x86_64-w64-mingw32.cmake \
          -DGGML_D3D12=ON -DGGML_D3D11=ON -DGGML_BACKEND_DL=ON -DGGML_CPU_ALL_VARIANTS=ON
    cmake --build build-win -j

Native Windows (not tested yet):

    cmake -B build -DGGML_D3D12=ON -DGGML_D3D11=ON -DGGML_BACKEND_DL=ON -DGGML_CPU_ALL_VARIANTS=ON
    cmake --build build --config Release

Leave out either `-DGGML_D3D12=ON` or `-DGGML_D3D11=ON` to build one backend only.

The D3D12 backend loads `dxcompiler.dll` and `dxil.dll` at startup. Put both next to
`ggml-d3d12.dll` (take them from the `bin/x64` folder of a
[DirectXShaderCompiler release](https://github.com/microsoft/DirectXShaderCompiler/releases)).
The D3D11 backend needs nothing extra.

## Run

Direct3D 12 (default):

    llama-cli -m model.gguf -ngl 99

Direct3D 11 (a D3D11-only build uses it without any setting; with both built, turn it on):

    set GGML_D3D11_ENABLE=1
    llama-cli -m model.gguf -ngl 99

With both built, `GGML_D3D12_DISABLE=1` hides D3D12 and D3D11 takes its place.

Useful switches (D3D12 names; most exist as `GGML_D3D11_*` too):

| variable | effect |
|---|---|
| `GGML_D3D12_STATS=1` | dispatch counts, timings, and ops that fell back to the CPU |
| `GGML_D3D12_DEBUG=1` | list each GPU and why it was skipped |
| `GGML_D3D12_ONLY=<text>` | use only the GPUs whose name contains `<text>` |
| `GGML_D3D12_DRED=1` | on a GPU crash, report which shader was running (D3D12 only) |

Pre-built Windows zips of the D3D12 backend, with a step-by-step guide:
[`README-D3D12.md`](https://github.com/koza-coder/llama.cpp-d3d11-d3d12/blob/main/README-D3D12.md).

## Tested

`test-backend-ops`, all op groups:

| GPU | Direct3D 12 | Direct3D 11 |
|---|---|---|
| AMD Radeon AI PRO R9700 | pass | pass |
| Moore Threads MTT S80 | pass | pass (previous build; latest build partly run) |

Text generation also checked on Intel Iris Xe (both backends).

Speed on the R9700, 0.6B Q4_0 model, tokens per second:

| | prompt 512 | generation |
|---|---|---|
| Direct3D 12 | 6922 | 129.0 |
| Direct3D 11 | 7267 | 96.1 |

Known limits of D3D11: flash attention with head size 192 or more is left to the CPU on AMD GPUs;
some quantized KV cache types are not supported in flash attention.

## Upstream

Everything else is upstream llama.cpp: <https://github.com/ggml-org/llama.cpp>. Its own README
is `README.md` in the repository root.
