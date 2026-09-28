---
title: CUDA 스레드란 무엇인가?
---

![스레드는 스레드 그룹 계층 구조의 최하위 수준에 해당하며(왼쪽 위), [스트리밍 다중처리기](/gpu-glossary/device-hardware/streaming-multiprocessor)의 [코어](/gpu-glossary/device-hardware/core)에 매핑됩니다. NVIDIA의 [CUDA Refresher: The CUDA Programming Model](https://developer.nvidia.com/blog/cuda-refresher-cuda-programming-model/) 및 [CUDA C++ 프로그래밍 가이드](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programming-model) 다이어그램을 바탕으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-cuda-programming-model.svg)

_실행 스레드(Thread of execution)_(줄여서 "스레드")는 GPU 프로그래밍의 가장 작은 단위이자 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)의 [스레드 계층 구조](/gpu-glossary/device-software/thread-hierarchy)를 이루는 기본 원자입니다. 스레드는 자신만의 전용 [레지스터](/gpu-glossary/device-software/registers)를 갖지만, 그 외 전용 하드웨어 자원은 거의 갖지 않습니다.

[SASS](/gpu-glossary/device-software/streaming-assembler)와 [PTX](/gpu-glossary/device-software/parallel-thread-execution) 프로그램은 모두 스레드를 대상으로 작성됩니다. 이는 하나 이상의 스레드로 구성된 프로세스를 대상으로 작성되는 POSIX 환경의 전형적인 C 프로그램과 대조적입니다. 또한 POSIX 스레드와 달리 [CUDA](/gpu-glossary/device-software/cuda-programming-model) 스레드는 시스템 콜(syscall)을 호출하는 데 사용되지 않습니다.

CPU 스레드와 마찬가지로 GPU 스레드 역시 독립적인 명령어 포인터(Instruction Pointer)/프로그램 카운터(Program Counter)를 가질 수 있습니다. 하지만 성능상의 이유로, GPU 프로그램은 통상 [워프(Warp)](/gpu-glossary/device-software/warp) 내의 모든 스레드가 동일한 명령어 포인터를 공유하며 보조를 맞추어(lock-step) 명령어를 함께 실행하도록 작성됩니다([워프 스케줄러](/gpu-glossary/device-hardware/warp-scheduler) 참조).

CPU 스레드와 마찬가지로 GPU 스레드도 방출된 레지스터(spilled registers)와 함수 호출 스택을 저장하기 위해 [전역 메모리](/gpu-glossary/device-hardware/gpu-ram)에 스택을 둡니다. 그러나 고성능 [커널](/gpu-glossary/device-software/kernel)에서는 통상 이 둘의 사용을 최대한 억제합니다.

단일 [CUDA 코어](/gpu-glossary/device-hardware/cuda-core)는 단일 스레드의 명령어를 실행합니다.
