---
title: 스레드 블록 그리드란 무엇인가?
---

![스레드 블록 그리드는 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)의 스레드 그룹 계층 구조에서 최상위 수준에 해당하며(왼쪽), 여러 [스트리밍 다중처리기](/gpu-glossary/device-hardware/streaming-multiprocessor)에 매핑됩니다(오른쪽 아래). NVIDIA의 [CUDA Refresher: The CUDA Programming Model](https://developer.nvidia.com/blog/cuda-refresher-cuda-programming-model/) 및 [CUDA C++ 프로그래밍 가이드](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programming-model) 다이어그램을 바탕으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-cuda-programming-model.svg)

CUDA [커널](/gpu-glossary/device-software/kernel)이 실행(launch)되면 스레드 블록 그리드(Thread Block Grid)라고 하는 [스레드](/gpu-glossary/device-software/thread) 집합이 생성됩니다. 그리드는 1차원, 2차원, 또는 3차원 구조로 구성할 수 있으며 여러 [스레드 블록](/gpu-glossary/device-software/thread-block)으로 이루어집니다.

이에 대응하는 [메모리 계층 구조](/gpu-glossary/device-software/memory-hierarchy)의 계층은 [전역 메모리](/gpu-glossary/device-software/global-memory)입니다.

[스레드 블록](/gpu-glossary/device-software/thread-block)은 실질적으로 독립적인 연산 단위입니다. 이들은 비결정적인 순서에 따라 동시(concurrent)에 실행되며, 단일 [스트리밍 다중처리기(SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)만 탑재된 GPU에서는 완전히 순차적으로 실행되는 것부터, 모든 블록을 동시에 실행할 수 있는 충분한 하드웨어 자원을 갖춘 GPU에서는 완전한 병렬(parallel)로 실행되는 것까지 다양한 방식으로 처리됩니다.
