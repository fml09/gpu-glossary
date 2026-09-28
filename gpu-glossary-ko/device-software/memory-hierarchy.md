---
title: CUDA 메모리 계층 구조란 무엇인가?
---

![[공유 메모리](/gpu-glossary/device-software/shared-memory)와 [전역 메모리](/gpu-glossary/device-software/global-memory)는 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)의 메모리 계층 구조(왼쪽)에서 두 계층을 형성하며, 각각 [L1 데이터 캐시](/gpu-glossary/device-hardware/l1-data-cache) 및 [GPU RAM](/gpu-glossary/device-hardware/gpu-ram)에 매핑됩니다. NVIDIA의 [CUDA Refresher: The CUDA Programming Model](https://developer.nvidia.com/blog/cuda-refresher-cuda-programming-model/) 및 [CUDA C++ 프로그래밍 가이드](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programming-model) 다이어그램을 바탕으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-cuda-programming-model.svg)

[CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)에서는 [스레드 계층 구조](/gpu-glossary/device-software/thread-hierarchy)의 각 수준에 속한 그룹 내 모든 [스레드](/gpu-glossary/device-software/thread)가 함께 공유하는 개별 메모리 영역에 접근할 수 있으며, 이를 '메모리 계층 구조(Memory Hierarchy)'라고 부릅니다. 이 메모리는 스레드 간 협력 및 통신에 사용되며, 하드웨어나 런타임이 아닌 프로그래머가 직접 관리합니다.

[스레드 블록 그리드](/gpu-glossary/device-software/thread-block-grid) 수준에서 공유되는 메모리는 [GPU RAM](/gpu-glossary/device-hardware/gpu-ram)에 위치하며 [전역 메모리](/gpu-glossary/device-software/global-memory)로 알려져 있습니다. 원자적 연산과 배리어를 통해 이 메모리에 대한 접근을 조율할 수 있지만, [스레드 블록](/gpu-glossary/device-software/thread-block) 간의 실행 순서는 정해져 있지 않습니다.

개별 [스레드](/gpu-glossary/device-software/thread) 수준에서 사용하는 메모리는 [스트리밍 다중처리기(SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)의 [레지스터 파일](/gpu-glossary/device-hardware/register-file)의 일부분입니다. [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)의 원래 시맨틱에 따르면 이 메모리는 단일 [스레드](/gpu-glossary/device-software/thread) 전용의 프라이빗 메모리이지만, [텐서 코어](/gpu-glossary/device-hardware/tensor-core) 기반의 행렬 곱셈을 위해 [PTX](/gpu-glossary/device-software/parallel-thread-execution) 및 [SASS](/gpu-glossary/device-software/streaming-assembler)에 추가된 특정 명령어들은 여러 [스레드](/gpu-glossary/device-software/thread)에 걸쳐 입력과 출력을 공유하기도 합니다.

그 중간 단계로서, 스레드 계층 구조의 [스레드 블록](/gpu-glossary/device-software/thread-block) 수준을 위한 [공유 메모리](/gpu-glossary/device-software/shared-memory)는 각 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)의 [L1 데이터 캐시](/gpu-glossary/device-hardware/l1-data-cache)에 저장됩니다. 새로운 데이터를 읽어오기 전까지 [최대한 많은 산술 연산을 수행할 수 있도록(연산 집약도 극대화)](/gpu-glossary/perf/arithmetic-intensity) 공유 메모리에 데이터를 효율적으로 로드하는 등, 이 캐시를 세심하게 관리하는 것이야말로 [고성능](/gpu-glossary/perf) CUDA [커널](/gpu-glossary/device-software/kernel) 설계의 핵심입니다.
