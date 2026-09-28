---
title: CUDA 스레드 블록이란 무엇인가?
---

![스레드 블록은 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)의 스레드 그룹 계층 구조에서 중간 수준에 해당합니다(왼쪽). 스레드 블록은 단일 [스트리밍 다중처리기](/gpu-glossary/device-hardware/streaming-multiprocessor)에서 실행됩니다(오른쪽 중간). NVIDIA의 [CUDA Refresher: The CUDA Programming Model](https://developer.nvidia.com/blog/cuda-refresher-cuda-programming-model/) 및 [CUDA C++ 프로그래밍 가이드](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programming-model) 다이어그램을 바탕으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-cuda-programming-model.svg)

스레드 블록(Thread Block)은 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)의 [스레드 계층 구조](/gpu-glossary/device-software/thread-hierarchy)에서 [그리드](/gpu-glossary/device-software/thread-block-grid) 아래, 개별 [스레드](/gpu-glossary/device-software/thread) 위에 위치하는 계층입니다. 이는 [PTX](/gpu-glossary/device-software/parallel-thread-execution) 및 [SASS](/gpu-glossary/device-software/streaming-assembler)에서 구체적으로 구현되는 [협력형 스레드 배열(CTA)](/gpu-glossary/device-software/cooperative-thread-array)에 대응하는 CUDA 프로그래밍 모델의 추상적 개념입니다.

블록은 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)에서 프로그래머에게 노출되는 스레드 협력의 최소 단위입니다. 각 블록은 상호 독립적으로 실행되어야 하므로 임의의 순서로 완전히 직렬 실행되거나 임의로 인터리빙(interleaving)되어 실행되더라도 모두 올바른 실행으로 간주됩니다.

단일 CUDA [커널](/gpu-glossary/device-software/kernel) 실행은 하나 이상의 스레드 블록([스레드 블록 그리드](/gpu-glossary/device-software/thread-block-grid) 형태)을 생성하며, 각 블록은 하나 이상의 [워프(Warp)](/gpu-glossary/device-software/warp)를 포함합니다. 블록의 크기는 현재 디바이스 기준으로 최대 1024개 스레드 한도 내에서 자유롭게 지정할 수 있지만, 일반적으로 [워프](/gpu-glossary/device-software/warp) 크기(현재 디바이스 기준 32)의 배수로 설정합니다.
