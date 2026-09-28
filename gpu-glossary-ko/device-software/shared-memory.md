---
title: 공유 메모리란 무엇인가?
---

![공유 메모리는 CUDA 스레드 그룹 계층 구조(왼쪽)의 [스레드 블록](/gpu-glossary/device-software/thread-block) 수준(왼쪽 가운데)에 대응하는 추상 메모리입니다. NVIDIA의 [CUDA Refresher: The CUDA Programming Model](https://developer.nvidia.com/blog/cuda-refresher-cuda-programming-model/) 및 [CUDA C++ 프로그래밍 가이드](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programming-model) 다이어그램을 바탕으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-cuda-programming-model.svg)

공유 메모리(Shared Memory)는 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)의 [스레드 계층 구조](/gpu-glossary/device-software/thread-hierarchy)에서 [스레드 블록](/gpu-glossary/device-software/thread-block) 수준에 대응하는 [메모리 계층 구조](/gpu-glossary/device-software/memory-hierarchy)의 한 계층입니다. 공유 메모리는 [전역 메모리](/gpu-glossary/device-software/global-memory)에 비해 용량은 훨씬 작지만 처리량과 지연 시간 면에서 훨씬 빠릅니다.

따라서 전형적인 [커널](/gpu-glossary/device-software/kernel)의 동작 흐름은 대략 다음과 같은 구조를 이룹니다.

- [전역 메모리](/gpu-glossary/device-software/global-memory)에서 공유 메모리로 데이터를 로드합니다.
- [CUDA 코어](/gpu-glossary/device-hardware/cuda-core) 및 [텐서 코어](/gpu-glossary/device-hardware/tensor-core)를 통해 해당 데이터를 대상으로 일련의 산술 연산을 수행합니다.
- 연산을 수행하는 동안 필요에 따라 배리어(barrier)를 활용하여 [스레드 블록](/gpu-glossary/device-software/thread-block) 내부의 [스레드](/gpu-glossary/device-software/thread)들을 동기화합니다.
- 처리된 데이터를 다시 [전역 메모리](/gpu-glossary/device-software/global-memory)에 기록하며, 필요한 경우 원자적 연산(atomic)을 활용하여 [스레드 블록](/gpu-glossary/device-software/thread-block) 간의 경쟁 상태(race condition)를 방지합니다.

공유 메모리는 GPU의 [스트리밍 다중처리기(SM)](/gpu-glossary/device-hardware/streaming-multiprocessor) 내에 위치한 [L1 데이터 캐시](/gpu-glossary/device-hardware/l1-data-cache)에 저장됩니다.
