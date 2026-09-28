---
title: 협력형 스레드 배열(CTA)이란 무엇인가?
---

![협력형 스레드 배열(CTA)은 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)의 스레드 계층 구조에서 [스레드 블록](/gpu-glossary/device-software/thread-block) 수준에 해당합니다. NVIDIA의 [CUDA Refresher: The CUDA Programming Model](https://developer.nvidia.com/blog/cuda-refresher-cuda-programming-model/) 및 [CUDA C++ 프로그래밍 가이드](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programming-model)의 다이어그램을 바탕으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-cuda-programming-model.svg)

협력형 스레드 배열(Cooperative Thread Array, CTA)은 동일한 [스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)에 스케줄링되는 스레드 집합입니다. CTA는 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)의 [스레드 블록](/gpu-glossary/device-software/thread-block)을 [PTX](/gpu-glossary/device-software/parallel-thread-execution) 및 [SASS](/gpu-glossary/device-software/streaming-assembler) 수준에서 구현한 실체입니다. CTA는 하나 이상의 [워프(Warp)](/gpu-glossary/device-software/warp)로 구성됩니다.

프로그래머는 CTA 내부의 [스레드](/gpu-glossary/device-software/thread)들이 서로 협력하여 동작하도록 제어할 수 있습니다. [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)의 [L1 데이터 캐시](/gpu-glossary/device-hardware/l1-data-cache)에 위치하며 프로그래머가 직접 관리하는 [공유 메모리](/gpu-glossary/device-software/shared-memory) 덕분에 이러한 협력 작업을 빠르게 수행할 수 있습니다. 단일 CTA 내부의 스레드와 달리, 서로 다른 CTA에 속한 스레드는 배리어(barrier) 동기화 방식으로 협력할 수 없으며 원자적 연산(atomic update) 명령어 등을 통해 [전역 메모리](/gpu-glossary/device-software/global-memory)를 거쳐 협력해야 합니다. 런타임 시 CTA 스케줄링은 드라이버가 제어하므로 CTA 실행 순서는 정해져 있지 않으며, 특정 CTA가 다른 CTA의 완료를 대기하도록 만들면 쉽게 데드락(교착 상태)에 빠질 수 있습니다.

단일 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)에 동시에 스케줄링할 수 있는 CTA 개수는 [달성 가능한 점유율(Occupancy)](/gpu-glossary/perf/occupancy)을 결정하며 여러 요인의 영향을 받습니다. 기본적으로 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)이 보유한 하드웨어 자원은 [레지스터 파일](/gpu-glossary/device-hardware/register-file)의 라인 수, [워프](/gpu-glossary/device-software/warp) 슬롯, [L1 데이터 캐시](/gpu-glossary/device-hardware/l1-data-cache) 내 [공유 메모리](/gpu-glossary/device-software/shared-memory) 용량 등으로 제한되어 있습니다. 각 CTA는 SM에 스케줄링될 때 [컴파일](/gpu-glossary/host-software/nvcc) 시점에 계산된 일정량의 자원을 사용합니다.
