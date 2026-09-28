---
title: CuTe란 무엇인가?
---

CuTe(CUDA Templates)는 [CUTLASS](/gpu-glossary/host-software/cutlass) 내에 포함된 헤더 온리(Header-only) [CUDA C++](/gpu-glossary/host-software/cuda-c) 라이브러리로, [데이터 메모리 계층](/gpu-glossary/device-software/memory-hierarchy)과 [스레드 계층](/gpu-glossary/device-software/thread-hierarchy)의 다차원 텐서를 체계적으로 기술하고 조작하기 위해 만들어졌습니다.

이름에서 알 수 있듯 CuTe는 CUDA C++ [템플릿(Templates)](https://en.cppreference.com/cpp/language/templates)을 광범위하게 활용합니다. 템플릿은 다른 언어의 [제네릭(Generics)](https://doc.rust-lang.org/rust-by-example/generics.html)에 대응하는 [매개변수 다형성(Parametric polymorphism)](https://bartoszmilewski.com/2014/09/22/parametricity-money-for-nothing-and-theorems-for-free/)을 C++로 구현한 문법입니다. 다형성 함수는 코드를 한 번만 작성해도 다양한 데이터 타입에 걸쳐 동일하게 동작합니다. CuTe는 파이썬 도메인 특화 언어(DSL)를 통해 CuTe 및 CUTLASS 템플릿을 노출하는 [CuTe DSL](/gpu-glossary/host-software/cute-dsl)과는 구분되는 C++ 라이브러리입니다.

CuTe 타입 시스템의 핵심은 `Layout`(레이아웃)입니다. `Layout`은 CuTe의 `Tensor`(텐서)에 접근하는 규칙적인 패턴을 정의합니다. `Tensor`는 `Layout`과 실제 [메모리](/gpu-glossary/device-software/memory-hierarchy) 포인터를 결합한 객체입니다. 특히 `Layout`은 자유롭게 합성할 수 있다는 중요한 특성을 지닙니다. 수학적으로 [하나의 범주(Category)](https://arxiv.org/abs/2601.05972)를 형성하며 [풍부한 대수적 구조](https://arxiv.org/abs/2603.02298)를 제공하므로, 강력한 표현력과 엄밀한 논리 구조를 동시에 갖추고 있습니다. `Layout` 자체는 메모리 크기와 탐색 방식을 정의하는 `Shape`(형상) 및 `Stride`(보폭) 튜플의 조합으로 구성됩니다.

CuTe는 타입 시스템을 활용하여 메모리 배치, 스트라이드 접근 패턴, 타일링과 같은 프로그램의 핵심 메타데이터를 정적으로 인코딩합니다. 이를 통해 [컴파일러](/gpu-glossary/host-software/nvcc)가 코드의 정확성을 미리 검증하고 최적화를 적용하는 동안 불변 조건(Invariants)을 유지할 수 있도록 돕습니다. 그 결과 연산 성능의 저하 없이 매우 높은 수준의 [커널](/gpu-glossary/device-software/kernel) 메타프로그래밍이 가능해집니다. 예를 들어 동일한 템플릿 코드를 여러 세대의 [스트리밍 다중처리기(SM) 아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture)에 맞추어 고도로 최적화된 개별 커널로 컴파일할 수 있습니다. 레이아웃 계산이 컴파일 시점에 완전히 해결되므로 런타임 오버헤드가 전혀 발생하지 않으며, 이는 [메모리 제약(Memory-bound)](/gpu-glossary/perf/memory-bound) 워크로드의 [성능](/gpu-glossary/perf)을 극대화하는 데 결정적인 역할을 합니다.

자세한 내용은 NVIDIA의 [공식 CuTe 문서](https://docs.nvidia.com/cutlass/4.4.2/media/docs/cpp/cute/index.html)에서 확인할 수 있습니다.

아래 소개하는 CuTe 기반 행렬 전치(Transpose) 커널은 [Colfax International의 기술 기고문](https://research.colfax-intl.com/tutorial-matrix-transpose-in-cutlass/)에 소개된 기본 구현을 바탕으로 작성되었으며, CuTe의 핵심 구성 요소인 템플릿, 형상(Shape), 레이아웃(Layout), 텐서(Tensor)의 조작 방식을 명확하게 보여줍니다. 이 코드는 [Modal 노트북 예제](https://modal.com/notebooks/modal-labs/examples/nb-owEUD0kdSVeL4KeEX5sjh1)를 통해 H100 GPU에서 직접 실행해 볼 수 있습니다.

```cpp
// one CuTe trick: transpose a row-major matrix just using Layouts
template <typename T>
__global__ void transpose_kernel(const T* __restrict__ d_S,
                                 T* __restrict__ d_D,
                                 int M, int N)
{
    // define the Shape of tiles worked on by thread blocks
    using b = Int<32>;
    auto block_shape = make_shape(b{}, b{});

    // define the Shape of input/output Tensors
    auto tensor_shape = make_shape(M, N);

    // define the Layout of the input and output Tensors in global memory
    auto gmemLayoutS  = make_layout(tensor_shape, GenRowMajor{}); // input:  row-major
    auto gmemLayoutDT = make_layout(tensor_shape, GenColMajor{}); // output: col-major

    // construct the Tensors
    auto tensor_S  = make_tensor(make_gmem_ptr(d_S), gmemLayoutS);
    auto tensor_DT = make_tensor(make_gmem_ptr(d_D), gmemLayoutDT);

    // define a tile-ing of the Tensors (as a "Tensor of Tensors")
    auto tiled_tensor_S  = tiled_divide(tensor_S,  block_shape);
    auto tiled_tensor_DT = tiled_divide(tensor_DT, block_shape);

    // pull out the tiles this thread block will be working on
    auto tile_S  = tiled_tensor_S (make_coord(_, _), blockIdx.x, blockIdx.y);
    auto tile_DT = tiled_tensor_DT(make_coord(_, _), blockIdx.x, blockIdx.y);

    // create a Layout for threads in the thread block
    auto thr_layout = make_layout(
        make_shape(Int<8>{}, Int<32>{}),
        GenRowMajor{}
    );

    // pull out the tile this thread will work on
    auto thr_tile_S  = local_partition(tile_S,  thr_layout, threadIdx.x);
    auto thr_tile_DT = local_partition(tile_DT, thr_layout, threadIdx.x);

    // define a "Tensor" in register memory
    auto rmem = make_tensor_like<T>(thr_tile_S);

    // copy tile into registers
    copy(thr_tile_S, rmem);
    // copy tile out of registers as though it were column-major
    copy(rmem, thr_tile_DT);
}
```
