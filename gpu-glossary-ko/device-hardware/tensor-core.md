---
title: 텐서 코어란 무엇인가?
---

텐서 코어(Tensor Core)는 명령어 하나로 행렬 전체를 연산하는 GPU [코어](/gpu-glossary/device-hardware/core)입니다.

![H100 SM 내부 아키텍처. 텐서 코어의 크기가 더 크고 개수가 적은 점에 주목하십시오. NVIDIA의 [H100 백서](https://modal-cdn.com/gpu-glossary/gtc22-whitepaper-hopper.pdf)를 기반으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-gh100-sm.svg)

단일 명령어 인출(instruction fetch)로 더 많은 데이터를 처리하면 전력 소모를 획기적으로 줄일 수 있어 성능 향상의 폭이 크게 넓어집니다(NVIDIA 수석 과학자 Bill Dally의 [강연](https://youtu.be/kLiwvnr4L80?t=868) 참조). 볼타(Volta) [스트리밍 다중처리기(SM) 아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture) 세대에서 처음 도입된 이후, 텐서 코어는 NVIDIA GPU에서 최고 수준의 [연산 처리량](/gpu-glossary/perf/arithmetic-bandwidth)을 달성할 수 있는 유일한 수단이 되었습니다. [CUDA 코어](/gpu-glossary/device-hardware/cuda-core)보다 초당 100배 더 많은 부동소수점 연산을 제공합니다.

예를 들어 `HMMA16.16816.F32` [SASS](/gpu-glossary/device-software/streaming-assembler) 명령어는 행렬 A, B, C, D에 대해 D = AB + C를 계산합니다(여기서 C는 종종 D와 물리적으로 동일한 행렬입니다). `MMA`는 '행렬 곱셈 및 누적(Matrix Multiply and Accumulate)'을 의미합니다. `HMMA16`은 입력이 반정밀도(`16`비트)임을 나타내고, `F32`는 출력이 `32`비트(단정밀도) 부동소수점으로 누적됨을 의미합니다.

중간에 위치한 `16816`은 16,000을 넘는 하나의 숫자가 아닙니다. 숫자열 `16`, `8`, `16`은 연산 대상 행렬의 차원을 나타냅니다. NVIDIA는 [PTX](/gpu-glossary/device-software/parallel-thread-execution) 명령어 등에서 이 차원을 일반적으로 `m`, `n`, `k`로 명명합니다. 행렬 A와 B의 외부 차원인 `m`과 `n`이 먼저 오고, 누적을 위한 공유 내부 차원 `k`가 뒤따릅니다. 이들을 곱해보면 `HMMA16.16816.F32` 명령어 하나가 16 × 8 × 16 = 2,048회의 곱셈-누적(MAC, Multiply-Accumulate) 연산을 수행함을 알 수 있습니다.

주의할 점은 단일 [스레드](/gpu-glossary/device-software/thread)의 명령어 하나가 행렬 곱셈 전체를 단독으로 수행하지 않는다는 것입니다. 대신 하나의 [워프](/gpu-glossary/device-software/warp)를 이루는 32개 스레드가 함께 명령어를 실행하여 협력적으로 결과를 산출합니다. 명령어당 소비되는 전력 오버헤드의 대부분은 디코딩 단계에서 발생하는데, [워프 스케줄러](/gpu-glossary/device-hardware/warp-scheduler) 덕분에 이 디코딩 비용을 [워프](/gpu-glossary/device-software/warp) 전체가 공유합니다. 32개 스레드로 나누어 보더라도 명령어당 2,048 ÷ 32 = 64회의 MAC 연산이 수행됩니다.

이러한 이유로 텐서 코어나 Google TPU(Tensor Processing Unit)의 시스톨릭 어레이(systolic array) 같은 유사 하드웨어를 일종의 [복합 명령어 집합 컴퓨터(CISC, Complex Instruction Set Computer)](https://www.omgwiki.org/ddsf/doku.php?id=ddsf:public:guidebook:06_append:glossary:c:cisc) 하드웨어 형태로 이해하는 것이 유용합니다. TPU에 이러한 관점을 적용한 더 자세한 설명은 컴퓨터 구조학자 David Patterson의 [강연](https://youtu.be/fhHAArxwzvQ?t=2072)을 참고하시기 바랍니다. 참고로 Patterson 교수는 [CISC와 RISC라는 용어를 고안](https://www.semanticscholar.org/paper/4d3a941a5749dbf0dd39554f12597c449c3c07ff)한 인물입니다.

이러한 어셈블러 수준 명령어는 컴파일러가 `wmma`([공식 문서](https://docs.nvidia.com/cuda/archive/12.8.0/parallel-thread-execution/index.html#warp-level-matrix-instructions) 참조)와 같은 [PTX 수준](/gpu-glossary/device-software/parallel-thread-execution) 행렬 곱셈 및 누적 명령어를 구현하는 과정에서 생성될 수 있습니다. 해당 명령어들 역시 행렬 A, B, C, D에 대해 D = AB + C를 계산하지만, 일반적으로 더 작은 행렬을 다루는 여러 개의 개별 [SASS](/gpu-glossary/device-software/streaming-assembler) 텐서 코어 명령어로 컴파일됩니다.

[PTX](/gpu-glossary/device-software/parallel-thread-execution) 명령어 집합 아키텍처(ISA)의 이러한 명령어들은 상위 수준 언어인 [CUDA C++ 프로그래밍 언어](/gpu-glossary/host-software/cuda-c)에서 내장 함수(intrinsic) 형태로 노출됩니다.

역순으로 살펴보면, 두 개의 16×16 행렬 곱셈 `C = A @ B`를 작성한 [CUDA C++](/gpu-glossary/host-software/cuda-c) 코드 한 줄은 다음과 같을 수 있습니다.

```cpp
wmma::mma_sync(c, a, b, c);
```

여기서 `c`는 모든 요소가 0으로 초기화된 상태이며, 첫 번째 인자로 전달된 것은 이 변수가 출력 대상임을 나타냅니다. 이 코드는 [`nvcc`](/gpu-glossary/host-software/nvcc)를 통해 다음과 같은 [PTX](/gpu-glossary/device-software/parallel-thread-execution) 중간 표현으로 컴파일될 수 있습니다.

```ptx
wmma.mma.sync.aligned.col.row.m16n16k16.f32.f32 {%f2, %f3, %f4, %f5, %f6, %f7, %f8, %f9}, {%r2, %r3, %r4, %r5, %r6, %r7, %r8, %r9}, {%r10, %r11, %r12, %r13, %r14, %r15, %r16, %r17}, {%f1, %f1, %f1, %f1, %f1, %f1, %f1, %f1};
```

그런 다음 최종적으로 `ptxas`에 의해 다음과 같은 [SASS](/gpu-glossary/device-software/streaming-assembler) 코드로 컴파일됩니다.

```sass
HMMA.1688.F32 R20, R12, R11, RZ   // 1
HMMA.1688.F32 R24, R12, R17, RZ   // 2
HMMA.1688.F32 R20, R14, R16, R20  // 3
HMMA.1688.F32 R24, R14, R18, R24  // 4
```

각 `HMMA` 명령어의 피연산자는 순서대로 `D = A @ B + C`로 읽을 수 있습니다. 예를 들어 3번 명령어는 출력 `D`에 [레지스터](/gpu-glossary/device-hardware/register-file) 20을 사용하고, 입력 `A`와 `B`에 각각 레지스터 14와 16을 사용하며, 입력 `C`로 레지스터 20을 재사용하여 `C += A @ B` 연산을 수행합니다.

이 프로그램은 전체 16×16 정사각 행렬 곱셈을 4개의 개별 명령어로 분할하며, 각 명령어는 16×8 행렬과 8×8 행렬의 곱셈을 수행합니다. 마찬가지로 대규모 행렬 곱셈을 실행하는 프로그램도 지금 분석하고 있는 `mma_sync` 호출처럼 작업을 더 작은 행렬 곱셈 단위로 쪼개어 처리해야 합니다. 아래에서 이 프로그램의 세부 동작을 살펴보겠습니다.

![C = A @ B 연산을 위한 텐서 코어 MMA의 레지스터 사용 방식. R11, R17, R16, R18 레지스터는 각각 1, 2, 3, 4번 명령어에서 사용됩니다. 세부 내용은 본문 설명을 참고하십시오.](https://modal-cdn.com/gpu-glossary/light-tensor-core-mma.svg)

처음 두 명령어는 `R12`에 담긴 입력 `a`의 처음 8개 열과, `R11` 및 `R17`에 담긴 입력 `b`의 처음 8개 행 간의 행렬 곱을 계산하여 16×16 행렬을 만든 뒤 `R20`과 `R24`에 저장합니다. 이는 길고 좁은 행렬과 짧고 넓은 행렬의 곱이라는 점에서 일종의 "외적(outer product)" 형태를 띱니다(`RZ`는 값 `0(Zero)`을 담고 있는 특수 목적 "레지스터"입니다).

뒤이어 나오는 두 명령어는 `a`의 다음 8개 열과 `b`의 다음 8개 행에 대해 유사한 "외적"을 계산하고, 이를 앞선 두 명령어의 출력 결과와 누적하여 `c`의 최종값을 산출합니다.

다른 방식으로 설명하자면 다음과 같습니다. 행렬 B의 8개 행과 8개 열로 구성된 블록 내부, 그리고 행렬 A의 열 전체에 걸쳐, 명령어 관점에서 텐서 코어 내부에서 수많은 곱셈과 덧셈이 동시에 일어나 행렬 곱셈을 구현합니다. 각 명령어는 B에서 주어진 행과 열 블록에 대해 A의 모든 `m`개 행을 처리하며, 이들이 결합하여 완전한 행렬 곱셈을 완성합니다.

더 깊이 있는 분석을 원하시면 [Godbolt 컴파일러 출력 결과](https://godbolt.org/z/e6cqn8491)를 살펴보시기 바랍니다. 다만 이 코드는 텐서 코어를 사용한 [처리율 최적화](https://modal.com/blog/gpu-utilization-guide) 행렬 곱셈과는 거리가 멉니다. 고도로 최적화된 구현에 대해서는 [Pranjal Shandkar의 작업 로그](https://cudaforfun.substack.com/p/outperforming-cublas-on-h100-a-worklog)를 참고하시기 바랍니다.

호퍼(Hopper) 및 블랙웰(Blackwell) 텐서 코어에서 최대 성능을 끌어내기 위해서는 순수 [CUDA C++](/gpu-glossary/host-software/cuda-c)만으로는 한계가 있으며, 연산과 메모리 양쪽 모두에서 [PTX](/gpu-glossary/device-software/parallel-thread-execution) 내장 함수를 활용해야 합니다. 따라서 직접 작성하기보다는 [cuBLAS](/gpu-glossary/host-software/cublas)와 같은 기존 커널 라이브러리를 사용하거나, C++ 기반의 [CUTLASS](/gpu-glossary/host-software/cutlass) 또는 Python 기반의 [CuTe DSL](/gpu-glossary/host-software/cute-dsl)처럼 상위 수준 커널 프로그래밍 인터페이스를 활용하는 것이 일반적입니다.

텐서 코어는 [CUDA 코어](/gpu-glossary/device-hardware/cuda-core)보다 훨씬 크기가 크고 개수가 적습니다. H100 SXM5의 경우 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)당 수백 개의 [CUDA 코어](/gpu-glossary/device-hardware/cuda-core)가 탑재된 반면, 텐서 코어는 SM당 4개, 즉 [워프 스케줄러](/gpu-glossary/device-hardware/warp-scheduler)당 1개만 탑재되어 있습니다. 텐서 코어는 [텐서 메모리](/gpu-glossary/device-hardware/tensor-memory)를 소비하고 생성하는 주체입니다.

텐서 코어는 V100 GPU에서 처음 도입되었으며, 이는 NVIDIA GPU가 대규모 신경망 워크로드에 최적화되는 중요한 전환점이 되었습니다. 자세한 내용은 [V100 소개 NVIDIA 백서](https://images.nvidia.com/content/volta-architecture/pdf/volta-architecture-whitepaper.pdf)를 참고하시기 바랍니다.

텐서 코어의 내부 구조는 대외적으로 공개되지 않았으며, [SM 아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture) 세대마다 다를 가능성이 높습니다. TPU처럼 시스톨릭 어레이 구조일 것으로 널리 추정되고 있으나, 마이크로벤치마크 관련 문헌에서도 명확히 합의된 바는 없습니다.
