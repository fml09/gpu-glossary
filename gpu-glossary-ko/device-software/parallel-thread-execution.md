---
title: 병렬 스레드 실행(PTX)이란 무엇인가?
abbreviation: PTX
---

병렬 스레드 실행(Parallel Thread eXecution, PTX)은 병렬 프로세서(거의 대부분 NVIDIA GPU)에서 실행될 코드를 위한 중간 표현(IR)입니다. 이는 [NVIDIA CUDA 컴파일러 드라이버](/gpu-glossary/host-software/nvcc)인 `nvcc`가 출력하는 파일 형식 중 하나입니다. NVIDIA 엔지니어들은 주로 "피텍스"라고 발음하며, 그 외 사람들은 대부분 "피-티-엑스"라고 읽습니다.

NVIDIA 문서에서는 PTX를 '가상 머신(virtual machine)'이자 '명령어 집합 아키텍처(ISA)'라고 설명합니다.

프로그래머 관점에서 PTX는 가상 머신 모델을 대상으로 프로그래밍하기 위한 명령어 집합입니다. PTX를 생성하는 프로그래머나 컴파일러는 아직 출시되지 않은 차세대 기기를 포함하여 서로 다른 수많은 물리적 기기에서도 프로그램이 동일한 시맨틱으로 실행될 것임을 신뢰할 수 있습니다. 이런 점에서 PTX는 [x86_64](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html), [aarch64](https://developer.arm.com/documentation/ddi0487/latest/), [SPARC](https://www.gaisler.com/doc/sparcv8.pdf)와 같은 CPU 명령어 집합 아키텍처와도 유사합니다.

하지만 이러한 하드웨어 ISA와 달리, PTX는 LLVM-IR처럼 본질적으로 [중간 표현(Intermediate Representation, IR)](https://en.wikipedia.org/wiki/Intermediate_representation)에 해당합니다. [CUDA 바이너리](/gpu-glossary/host-software/cuda-binary-utilities)에 포함된 PTX 구성 요소는 실행 시점에 호스트의 [CUDA 드라이버](/gpu-glossary/host-software/nvidia-gpu-drivers)에 의해 디바이스별 실행 코드인 [SASS](/gpu-glossary/device-software/streaming-assembler)로 JIT(Just-In-Time) 컴파일됩니다.

NVIDIA GPU에서 PTX는 전방향 호환성(forward compatibility)을 갖습니다. 이러한 JIT 컴파일 메커니즘 덕분에, 해당 버전과 일치하거나 더 높은 [컴퓨트 성능](/gpu-glossary/device-software/compute-capability) 버전을 갖춘 GPU라면 문제없이 해당 프로그램을 실행할 수 있습니다. 이를 통해 PTX는 하드웨어와 소프트웨어의 세계를 깔끔하게 분리하는 [모래시계의 잘록한 허리(narrow waist)](https://www.oilshell.org/blog/2022/02/diagrams.html) 역할을 합니다.

대표적인 PTX 예제는 다음과 같습니다.

```ptx
.reg .f32 %f<7>;
```

- PTX-to-[SASS](/gpu-glossary/device-software/streaming-assembler) 컴파일러를 위한 지시문으로, 해당 커널이 7개의 32비트 부동소수점 [레지스터](/gpu-glossary/device-software/registers)를 사용함을 명시합니다. 레지스터는 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)의 [레지스터 파일](/gpu-glossary/device-hardware/register-file)에서 [스레드](/gpu-glossary/device-software/thread) 그룹인 [워프](/gpu-glossary/device-software/warp) 단위로 동적 할당됩니다.

```ptx
fma.rn.f32 %f5, %f4, %f3, 0f3FC00000;
```

- 혼합 곱셈-덧셈(Fused Multiply-Add, `fma`) 연산을 적용하여 레지스터 `f3`과 `f4`의 값을 곱한 뒤 상수 `0f3FC00000`을 더하고 결과를 `f5`에 저장합니다. 모든 숫자는 32비트 부동소수점 형식입니다. FMA 연산의 `rn` 접미사는 부동소수점 반올림 모드를 기본값인 [IEEE 754 "짝수 반올림(round even)"](https://en.wikipedia.org/wiki/IEEE_754)으로 설정합니다.

```ptx
mov.u32 %r1, %ctaid.x;
mov.u32 %r2, %ntid.x;
mov.u32 %r3, %tid.x;
```

- 협력형 스레드 배열 인덱스(`ctaid`), 협력형 스레드 배열 차원 크기(`ntid`), 스레드 인덱스(`tid`)의 `x`축 값을 3개의 `u32` 레지스터 `r1` ~ `r3`으로 각각 이동(`mov`)합니다.

PTX 프로그래밍 모델은 여러 단계의 병렬성을 프로그래머에게 노출합니다. 이러한 단계는 아래 다이어그램에 나타난 PTX 머신 모델을 통해 하드웨어에 직접 매핑됩니다.

![PTX 머신 모델. [PTX 공식 문서](https://docs.nvidia.com/cuda/parallel-thread-execution/#ptx-machine-model)를 바탕으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-ptx-machine-model.svg)

이 머신 모델에서 주목할 만한 점은 여러 프로세서를 위해 단 하나의 명령어 유닛만 존재한다는 것입니다. 각 프로세서가 단일 [스레드](/gpu-glossary/device-software/thread)를 실행하지만, 이들 스레드는 동일한 명령어를 함께 실행해야 합니다. 이것이 바로 '병렬 스레드 실행(Parallel Thread eXecution)', 즉 PTX라는 이름이 붙은 이유입니다. 이 스레드들은 [공유 메모리](/gpu-glossary/device-software/shared-memory)를 통해 서로 협력하며, 각자 독립적인 [레지스터](/gpu-glossary/device-software/registers)를 활용하여 서로 다른 결과를 도출합니다.

최신 버전의 PTX 문서는 NVIDIA [공식 사이트](https://docs.nvidia.com/cuda/parallel-thread-execution/)에서 확인할 수 있습니다. PTX 명령어 집합은 '[컴퓨트 성능](/gpu-glossary/device-software/compute-capability)'이라는 번호로 버전이 관리되며, 이는 '최소 지원 [스트리밍 다중처리기 아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture) 버전'과 같은 의미입니다.

인라인 PTX를 직접 손으로 작성하는 일은 극도의 극한 성능을 추구하는 경우가 아니라면 흔하지 않습니다. 이는 분석용 데이터베이스의 고성능 벡터화 쿼리 연산자나 운영체제 커널의 성능 민감한 부분에서 인라인 `x86_64` 어셈블리를 작성하는 것과 비슷합니다. 이 글을 작성하는 시점 기준으로, [Flash Attention 3](https://arxiv.org/abs/2407.08608)나 [Machete w4a16 커널](https://youtu.be/-4ZkpQ7agXM)처럼 `wgmma` 및 `tma` 명령어와 같은 Hopper 아키텍처 특화 하드웨어 기능을 활용하려면 인라인 PTX를 사용하는 것이 유일한 방법입니다. [CUDA C/C++](/gpu-glossary/host-software/cuda-c), [SASS](/gpu-glossary/device-software/streaming-assembler), [PTX](/gpu-glossary/device-software/parallel-thread-execution)를 한눈에 나란히 비교해 보는 기능은 [Godbolt](https://godbolt.org/z/5r9ej3zjW)에서 지원합니다. 자세한 내용은 [NVIDIA "Inline PTX Assembly in CUDA" 가이드](https://docs.nvidia.com/cuda/inline-ptx-assembly/)를 참조하십시오.
