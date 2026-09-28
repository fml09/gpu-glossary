---
title: CUDA 프로그래밍 모델이란 무엇인가?
---

CUDA 프로그래밍 모델은 대규모 병렬 프로세서를 프로그래밍하기 위한 프로그래밍 모델입니다.

CUDA는 _Compute Unified Device Architecture_의 약자입니다. 문맥에 따라 'CUDA'는 여러 대상을 가리킬 수 있습니다. [고수준 디바이스 아키텍처](/gpu-glossary/device-hardware/cuda-device-architecture)를 의미하기도 하고, 해당 설계를 갖춘 아키텍처를 위한 병렬 프로그래밍 모델을 뜻하기도 하며, C와 같은 고급 언어를 확장하여 이 프로그래밍 모델을 지원하는 [소프트웨어 플랫폼](/gpu-glossary/host-software/cuda-software-platform)을 나타내기도 합니다.

CUDA의 비전은 [Lindholm et al., 2008](https://www.cs.cmu.edu/afs/cs/academic/class/15869-f11/www/readings/lindholm08_tesla.pdf) 백서에 잘 정리되어 있습니다. 이 논문은 NVIDIA 공식 문서에 등장하는 수많은 핵심 주장, 다이어그램, 특정 기술적 표현의 기원이 된 원전이므로 읽어보기를 강력히 권장합니다.

여기서는 CUDA _프로그래밍 모델_에 초점을 맞추어 설명합니다.

[NVIDIA CUDA C++ 프로그래밍 가이드](https://docs.nvidia.com/cuda/cuda-c-programming-guide/#a-scalable-programming-model)에 따르면, CUDA 프로그래밍 모델은 세 가지 핵심 추상화를 제공합니다.

- [**스레드 그룹 계층 구조(Hierarchy of thread groups)**](/gpu-glossary/device-software/thread-hierarchy): 프로그램은 개별 스레드 단위로 실행되지만, [블록](/gpu-glossary/device-software/thread-block)부터 [그리드](/gpu-glossary/device-software/thread-block-grid)에 이르는 중첩된 계층 구조로 스레드 그룹을 지정하고 참조할 수 있습니다.
- [**메모리 계층 구조(Hierarchy of memories)**](/gpu-glossary/device-software/memory-hierarchy): 계층의 각 수준에 위치한 스레드 그룹은 해당 그룹 내부 통신을 위한 메모리 자원에 접근할 수 있습니다. 메모리 계층 구조의 [최하위 계층](/gpu-glossary/device-software/shared-memory)에 접근하는 속도는 [명령어 하나를 실행하는 것만큼 빨라야 합니다](/gpu-glossary/device-hardware/l1-data-cache).
- **배리어 동기화(Barrier synchronization)**: 스레드 그룹은 배리어(barrier)를 통해 실행 흐름을 서로 맞추고 협력할 수 있습니다.

실행 계층 구조와 메모리 계층 구조, 그리고 이들이 [디바이스 하드웨어](/gpu-glossary/device-hardware)에 매핑되는 방식은 다음 다이어그램에 요약되어 있습니다.

![왼쪽: CUDA 프로그래밍 모델의 추상 스레드 그룹 및 메모리 계층 구조. 오른쪽: 해당 추상화를 구현하는 물리적 하드웨어. NVIDIA의 [CUDA Refresher: The CUDA Programming Model](https://developer.nvidia.com/blog/cuda-refresher-cuda-programming-model/) 및 [CUDA C++ 프로그래밍 가이드](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programming-model) 다이어그램을 바탕으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-cuda-programming-model.svg)

이 세 가지 추상화는 GPU 디바이스의 병렬 실행 하드웨어 자원이 확장됨에 따라 프로그램이 투명하게 확장되도록 유도합니다.

조금 직설적으로 표현하자면, 이 프로그래밍 모델은 프로그래머가 [CUDA 아키텍처 기반](/gpu-glossary/device-hardware/cuda-device-architecture) GPU 프로그램을 작성할 때, 사용자가 더 최신의 NVIDIA GPU를 구매했는데도 성능이 향상되지 않는 상황을 방지해 줍니다.

예를 들어 CUDA 프로그램의 각 [스레드 블록](/gpu-glossary/device-software/thread-block) 내부에서는 긴밀한 협력이 가능하지만, 블록 간의 협력은 엄격히 제한됩니다. 이러한 제약 덕분에 각 블록은 프로그램에서 독립적으로 병렬 처리 가능한 작업 단위가 되며 어떤 순서로든 스케줄링될 수 있습니다. 컴퓨터 구조학 용어로 말하자면, 프로그래머가 컴파일러와 하드웨어에 병렬성을 명시적으로 드러내는(expose) 것입니다. 따라서 더 많은 스케줄링 유닛, 즉 더 많은 [스트리밍 다중처리기(SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)를 탑재한 새로운 GPU에서 프로그램을 실행하면 더 많은 블록이 동시에 병렬로 실행될 수 있습니다.

![8개의 [블록](/gpu-glossary/device-software/thread-block)으로 구성된 CUDA 프로그램이 2개의 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)을 가진 GPU에서는 4단계(웨이브)에 걸쳐 순차 실행되지만, SM 개수가 2배인 GPU에서는 절반의 단계만으로 실행됩니다. [CUDA 프로그래밍 가이드](https://docs.nvidia.com/cuda/cuda-c-programming-guide/)를 바탕으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-wave-scheduling.svg)

CUDA 프로그래밍 모델의 이러한 추상화는 [C++를 확장한 CUDA C++](/gpu-glossary/host-software/cuda-c)와 같이 고급 CPU 프로그래밍 언어의 확장 형태로 프로그래머에게 제공됩니다. 이 프로그래밍 모델은 소프트웨어적으로 가상 명령어 집합 아키텍처인 [PTX(Parallel Thread eXecution)](/gpu-glossary/device-software/parallel-thread-execution)와 저수준 어셈블리어인 [SASS(Streaming Assembler)](/gpu-glossary/device-software/streaming-assembler)를 통해 구현됩니다. 예를 들어 [스레드 계층 구조](/gpu-glossary/device-software/thread-hierarchy)의 [스레드 블록](/gpu-glossary/device-software/thread-block) 수준은 이러한 저수준 언어에서 [협력형 스레드 배열(CTA)](/gpu-glossary/device-software/cooperative-thread-array)로 구현됩니다.
