---
title: CuTe DSL이란 무엇인가?
---

CuTe DSL은 높은 개발 생산성과 뛰어난 성능을 동시에 확보하면서 GPU [커널](/gpu-glossary/device-software/kernel)을 작성하고 동적으로 컴파일할 수 있도록 지원하는 파이썬 기반 도메인 특화 언어(DSL, Domain-Specific Language)입니다.

CuTe DSL은 [CUDA C++](/gpu-glossary/host-software/cuda-c) 템플릿과 DSL 모음인 [CUTLASS](/gpu-glossary/host-software/cutlass) 프로젝트의 핵심 구성 요소입니다. 자주 사용되는 연산에 대해 완성된 커널을 직접 호출하는 [cuBLAS](/gpu-glossary/host-software/cublas)나 [cuDNN](/gpu-glossary/host-software/cudnn)과 달리, CUTLASS 스택은 개발자가 고성능 커널을 필요에 맞게 조합하여 정의할 수 있는 도구를 제공합니다.

CuTe DSL의 핵심 추상화로는 레이아웃(Layouts), 텐서(Tensors), 하드웨어 아톰(Hardware Atoms), 타일드 연산(Tiled Operations)이 있습니다. 레이아웃은 데이터가 메모리와 스레드에 정렬되는 방식을 나타냅니다. 텐서는 실제 메모리 포인터 또는 반복자(Iterator)를 레이아웃 메타데이터와 결합합니다. 아톰은 행렬 곱셈-누적(MMA, Matrix Multiply-Accumulate)이나 메모리 복사 같은 하드웨어 기본 연산 단위를 의미합니다. 타일드 연산은 이러한 아톰 연산을 [스레드 블록](/gpu-glossary/device-software/thread-block)과 [워프](/gpu-glossary/device-software/warp) 전반에 걸쳐 어떻게 타일 단위로 적용할지 기술합니다. 자세한 내부 원리는 [CuTe](/gpu-glossary/host-software/cute) 문서를 참고하시기 바랍니다.

파이썬 환경에서 CuTe DSL 커널을 실행할 때, 파이썬 애플리케이션은 먼저 `@cute.jit` 함수를 호출하며, 이 함수 내부에서 다시 실제 디바이스 연산을 담당하는 `@cute.kernel` 함수를 실행합니다.

`@cute.jit` 데코레이터는 파이썬 또는 다른 CuTe DSL 함수에서 호출할 수 있는 JIT 컴파일 함수를 선언합니다. `@cute.kernel` 데코레이터는 `@cute.jit` 함수 안에서만 실행 요청을 보낼 수 있는 GPU 커널 함수를 정의합니다. 일반 파이썬 코드에서는 `@cute.kernel` 함수를 직접 호출할 수 없습니다.

예를 들어 두 개의 1차원 텐서를 원소별로 더하는(Elementwise addition) 단순한 CuTe DSL 커널을 살펴보겠습니다. 이 연산은 [CUDA](/gpu-glossary/device-software/cuda-programming-model)의 탄생을 이끈 선구적 프로젝트인 [Ian Buck의 Brook 프레임워크](https://graphics.stanford.edu/papers/brookgpu/brookgpu.pdf) 시절부터 GPU 프로그래밍의 "Hello World"로 통하는 전통적인 예제입니다. [Modal 노트북 예제](https://modal.com/notebooks/modal-labs/examples/nb-Vnwf5bQck2WSSETJUPk2UD)를 통해 B200 GPU에서 이 커널을 직접 수정하고 실행해 볼 수 있습니다.

```python
import cutlass.cute as cute
import torch

Tensor = cute.Tensor | torch.Tensor


@cute.kernel
def elem_add_kernel(a: cute.Tensor, b: cute.Tensor, out: cute.Tensor):
    block_x, _, _ = cute.arch.block_idx()
    block_dim_x, _, _ = cute.arch.block_dim()
    thread_x, _, _ = cute.arch.thread_idx()

    i = block_x * block_dim_x + thread_x

    if i < out.shape[0]:
        out[i] = a[i] + b[i]


@cute.jit
def elem_add(a: Tensor, b: Tensor, out: Tensor):
    n = out.shape[0]
    threads_per_block = 128
    blocks = (n + threads_per_block - 1) // threads_per_block

    elem_add_kernel(a, b, out).launch(
        grid=(blocks, 1, 1),
        block=(threads_per_block, 1, 1),
    )
```

`elem_add_kernel` 함수는 실제 GPU에서 구동되는 [커널](/gpu-glossary/device-software/kernel)입니다. 각 [스레드](/gpu-glossary/device-software/thread)는 출력 벡터의 원소 하나를 계산합니다. 전역 원소 인덱스 `i`는 [스레드 블록](/gpu-glossary/device-software/thread-block) 인덱스, 블록 내부의 스레드 수, 블록 내 스레드 인덱스를 조합하여 다음과 같이 구합니다.

```python
i = block_x * block_dim_x + thread_x
```

`elem_add` 함수는 전체 출력 텐서를 처리하는 데 필요한 스레드 블록 수를 계산하고, 1차원 [스레드 블록 그리드](/gpu-glossary/device-software/thread-block-grid)를 구성하여 커널 실행을 요청합니다.

이 예제는 최적화보다는 개념 전달을 위한 코드이지만, 기초적인 메모리 접근 패턴을 충실히 보여줍니다. 인접한 스레드가 `a`와 `b`의 인접한 원소를 연속으로 읽고, `out`의 인접한 원소에 연속으로 기록합니다. 이는 [전역 메모리](/gpu-glossary/device-software/global-memory) 접근 시 대역폭을 극대화하는 병합 접근 패턴을 이룹니다. 자세한 원리는 [메모리 병합(Memory Coalescing)](/gpu-glossary/perf/memory-coalescing) 항목을 참조하십시오.

이러한 메모리 레이아웃 제어는 고성능 커널 개발에서 CuTe DSL이 각광받는 주된 이유입니다. 최고 수준의 [연산 성능](/gpu-glossary/perf)을 구현하기 어려운 이유는 어떤 스레드가 어떤 데이터를 담당할지, 메모리를 어떻게 순회할지, 연산 작업을 어떻게 타일링할지, 컴파일된 코드가 어떤 전용 하드웨어 명령어를 호출해야 할지 등 커널을 실제 하드웨어 구조에 정밀하게 일치시켜야 하기 때문입니다. CuTe DSL은 이러한 매핑 관계를 명시적으로 제어하면서도, 동일한 커널 코드 대부분을 다양한 텐서 형상과 [스트리밍 다중처리기(SM) 아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture)에 맞추어 유연하게 재사용할 수 있도록 지원합니다.

성능 최적화를 중시하는 엔지니어에게는 파이썬과 같은 인터프리터 언어로 작성된 프로그램이 어떻게 C++ 등 컴파일 언어로 구현된 코드와 맞먹는 성능을 낼 수 있는지 의아할 수 있습니다.

그 비결은 CuTe DSL 커널이 JIT(Just-In-Time) 방식으로 사전에 컴파일되기 때문입니다. 파이썬 소스 코드는 추상 구문 트리(AST)로 변환된 후 프록시 인자를 통해 트레이싱되고, 최종적으로 최적화된 기계어 형태로 컴파일됩니다. 단, JIT 컴파일되는 코드 블록 내부에서는 파이썬 전체 문법 중 엄선된 일부 하위 기능만 지원됩니다.

현재 CUTLASS 4.x 기준 컴파일 파이프라인은 [MLIR(Multi-Level Intermediate Representation)](https://mlir.llvm.org/)을 거쳐 [PTX](/gpu-glossary/device-software/parallel-thread-execution) 중간 표현으로 변환된 뒤, GPU 디바이스별 전용 기계어인 [SASS](/gpu-glossary/device-software/streaming-assembler)로 컴파일되어 실행됩니다.

실제 대표 사례로 [FlashAttention-4](https://arxiv.org/abs/2603.05451) 커널을 들 수 있습니다. 오픈 소스 코드를 분석한 [Modal의 기술 블로그 글](https://modal.com/blog/reverse-engineer-flash-attention-4)에서는 파이프라인 기반 워프 특화(Warp Specialization), [텐서 코어](/gpu-glossary/device-hardware/tensor-core) 연산, [텐서 메모리(Tensor Memory)](/gpu-glossary/device-hardware/tensor-memory) 및 [텐서 메모리 가속기(TMA)](/gpu-glossary/device-hardware/tensor-memory-accelerator) 제어를 통해 CuTe DSL만으로 어떻게 최고 수준의 최신 성능을 달성했는지 상세히 설명합니다.

CuTe DSL에 대한 더 구체적인 정보는 NVIDIA의 [CuTe DSL 공식 문서](https://docs.nvidia.com/cutlass/4.4.2/media/docs/pythonDSL/cute_dsl.html)와 [CuTe DSL 소개 블로그 기고](https://developer.nvidia.com/blog/achieve-cutlass-c-performance-with-python-apis-using-cute-dsl/)에서 확인할 수 있습니다.
