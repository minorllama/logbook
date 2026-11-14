## Numerical methods. 

* [Strassen multiplication](https://github.com/minorllama/numerical_algorithms/blob/main/recipes/linear_algebra/src/fast.rs).
*  
* [PDEs](https://github.com/minorllama/numerical_algorithms/tree/main/recipes/pdes): Numerical recipes for PDEs.

## Machine learning. 

* [A basic neural net with rust](https://github.com/minorllama/numerical_algorithms/tree/main/ml/two_class_nn-rust_burn): machine learning with rust/burn.  

* [MLPdes](https://github.com/minorllama/mlpde): Machine learning and PDEs - diffusion models and PINNs. 

* [MLearn](https://github.com/minorllama/mlearn): Machine learning for everything but PDEs. 

## Write-ups and small side projects. 

* [Burn RK4 Example for SHO](burn_rk4/README.md): Writing Python extensions using [Rust/burn](https://github.com/tracel-ai/burn).
  
* [Julia, CUDA.jl, CUDA C](julia_cuda_extension/README.md): Writing CUDA extensions for Julia using CUDA.jl to set up context for a CUDA C extension. 
This was initially written before there was support for `cuBLASDx` in Julia. [cuTile.jl](https://github.com/JuliaGPU/cuTile.jl) is a better option without descending into `CUDA C`.

* Getting [started](setup_cuda_oxide/README.md) with NVIDIA's experimental Rust to CUDA compiler, cuda-oxide. 

* A RAII wrapped [CuBuffer](https://github.com/minorllama/cubuffer-rs) for GPU arrays in rust.

* A hackable GUI minimal [texteditor](https://github.com/minorllama/gbuf-rs).

* [Pythonic Julia](log/pythonic_julia.md): Object-dot-method syntax with Julia.


