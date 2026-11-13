## Numerical methods. 

### Linear algebra recipes.

* [Strassen multiplication](https://github.com/minorllama/numerical_algorithms/blob/main/recipes/linear_algebra/src/fast.rs) 

## Write-ups and small side projects. 

* Writing Python extensions using [Rust/burn](https://github.com/tracel-ai/burn): [Burn RK4 Example for SHO](burn_rk4/README.md).
* Writing CUDA extensions for Julia using CUDA.jl to set up context for a CUDA C extension: [Julia, CUDA.jl, CUDA C](julia_cuda_extension/README.md). 
This was initially written before there was no support for `cuBLASDx` in Julia. [cuTile.jl](https://github.com/JuliaGPU/cuTile.jl) is a better option without descending into `CUDA C`.
* Getting [started](setup_cuda_oxide/README.md) with NVIDIA's experimental Rust to CUDA compiler, cuda-oxide. 
* A RAII wrapped [CuBuffer](https://github.com/minorllama/cubuffer-rs) for GPU arrays in rust.
* A hackable GUI barebones [texteditor](https://github.com/minorllama/gbuf-rs)


