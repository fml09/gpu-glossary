---
title: CUTLASS란 무엇인가?
---

CUTLASS(CUDA Templates for Linear Algebra Subroutines and Solvers)는 [CUDA](/gpu-glossary/device-software/cuda-programming-model) [커널](/gpu-glossary/device-software/kernel) 내부에서 고성능 선형대수 연산을 구현하기 위한 템플릿 기반 추상화 라이브러리입니다.

[cuBLAS](/gpu-glossary/host-software/cublas)와 마찬가지로 CUTLASS의 명칭은 하위 계층 선형대수 연산 루틴 표준인 [BLAS(Basic Linear Algebra Subprograms)](https://netlib.org/blas/blast-forum/)에서 따온 것입니다. 하지만 미리 컴파일된 함수를 그대로 호출하는 형태의 cuBLAS와 달리, CUTLASS는 개발자가 필요에 맞추어 커널을 직접 조립하고 작성할 수 있게 돕는 툴킷입니다. CUTLASS는 BLAS의 3단계 규격인 일반 행렬 곱셈(GEMM, General Matrix Multiplication)과 가장 밀접하게 연결되어 있습니다.

이름에서 알 수 있듯 CUTLASS는 일련의 [CUDA C++](/gpu-glossary/host-software/cuda-c) 템플릿 추상화로 설계되었습니다. [템플릿(Template)](https://en.cppreference.com/cpp/language/templates)은 다른 프로그래밍 언어에서 흔히 [제네릭(Generics)](https://doc.rust-lang.org/rust-by-example/generics.html)으로 불리는 [매개변수 다형성(Parametric polymorphism)](https://bartoszmilewski.com/2014/09/22/parametricity-money-for-nothing-and-theorems-for-free/)을 C++로 구현한 메커니즘입니다. 다형성 함수는 코드를 한 번만 정의하면 다양한 데이터 타입의 입력에 맞추어 유연하게 동작합니다.

현대 CUTLASS의 핵심 기반은 [CuTe](/gpu-glossary/host-software/cute) 라이브러리입니다. CuTe는 [데이터 메모리 계층](/gpu-glossary/device-software/memory-hierarchy)과 [스레드 계층](/gpu-glossary/device-software/thread-hierarchy)의 다차원 텐서를 자유롭게 합성하고 조작할 수 있도록 `Layout`과 `Tensor` 타입을 제공합니다. 이는 파이썬의 도메인 특화 언어(DSL)를 통해 CuTe와 CUTLASS 템플릿을 노출하는 [CuTe DSL](/gpu-glossary/host-software/cute-dsl)과는 구분되는 C++ 라이브러리입니다.

CuTe 위에 구축된 CUTLASS는 헤더 온리(Header-only) 방식의 CUDA C++ 라이브러리로서 세 가지 계층으로 동작합니다. 전체 `device`, 단일 [`kernel`](/gpu-glossary/device-software/kernel), 그리고 [스레드](/gpu-glossary/device-software/thread)들의 집합체(일반적으로 [스레드 블록](/gpu-glossary/device-software/thread-block))인 `collective` 계층이 그것입니다. `collective` 계층에서 행렬 곱셈은 통상 메인루프(Mainloop)와 에필로그(Epilogue)로 구분됩니다. 메인루프는 타일링(Tiling) 기법을 비롯한 핵심 행렬 연산 알고리즘을 표현합니다. 에필로그는 스케일링 계수 적용이나 인공신경망에서 필수적인 스칼라 비선형 활성화 함수 계산 같은 후처리 과정을 정의합니다.

CUTLASS는 최신 [스트리밍 다중처리기(SM) 아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture) 하드웨어에서 초고성능 행렬 곱셈 커널을 직접 구현할 때 가장 널리 사용됩니다. 이러한 커널은 하드웨어 한계에 가까운 피크 [성능](/gpu-glossary/perf)을 이끌어내기 위해 [텐서 코어](/gpu-glossary/device-hardware/tensor-core)를 극도로 정교하게 제어해야 합니다.

CUTLASS는 [GitHub에 오픈 소스로 공개](https://github.com/nvidia/cutlass)되어 있습니다. 라이브러리 내에는 CUTLASS로 작성된 수많은 고성능 오픈 소스 커널 구현체가 포함되어 있어, 다른 오픈 소스 커널을 개발할 때 표준적인 레퍼런스로 활용됩니다. CUTLASS의 핵심 요소를 활용하여 최상의 성능을 이끌어내는 방법을 상세히 다룬 [Colfax International 소속 Jay Shah의 튜토리얼](https://research.colfax-intl.com/)을 읽어보기를 강력히 권장합니다. 다만 일반적인 C++ 템플릿 메타프로그래밍과 마찬가지로 CUTLASS의 학습 곡선과 난이도는 매우 가파른 편입니다.
