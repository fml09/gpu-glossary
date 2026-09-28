---
title: 레지스터란 무엇인가?
---

![레지스터는 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)의 [메모리 계층 구조](/gpu-glossary/device-software/memory-hierarchy)에서 개별 [스레드](/gpu-glossary/device-software/thread)와 연결된 메모리입니다(왼쪽). NVIDIA의 [CUDA Refresher: The CUDA Programming Model](https://developer.nvidia.com/blog/cuda-refresher-cuda-programming-model/) 및 [CUDA C++ 프로그래밍 가이드](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programming-model) 다이어그램을 바탕으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-cuda-programming-model.svg)

[메모리 계층 구조](/gpu-glossary/device-software/memory-hierarchy)의 최하위 계층에는 레지스터(Register)가 있으며, 개별 [스레드](/gpu-glossary/device-software/thread)가 조작하는 정보를 저장합니다.

레지스터 값은 일반적으로 [스트리밍 다중처리기(SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)의 [레지스터 파일](/gpu-glossary/device-hardware/register-file)에 저장되지만, 레지스터가 부족해지면 상당한 성능 저하를 감수하고 [GPU RAM](/gpu-glossary/device-hardware/gpu-ram)의 [전역 메모리](/gpu-glossary/device-software/global-memory)로 방출(레지스터 스필링, register spilling)되기도 합니다.

CPU를 프로그래밍할 때와 마찬가지로, 이러한 레지스터는 [CUDA C](/gpu-glossary/host-software/cuda-c)와 같은 고급 언어에서 직접 조작하지 않습니다. 레지스터는 저수준 언어(여기서는 [병렬 스레드 실행(PTX)](/gpu-glossary/device-software/parallel-thread-execution)) 수준에서 비로소 노출됩니다. 레지스터는 통상 `ptxas`와 같은 컴파일러에 의해 관리됩니다. 컴파일러의 주요 목표 중 하나는 각 [스레드](/gpu-glossary/device-software/thread)가 사용하는 레지스터 공간을 제한하여 단일 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)에 더 많은 [스레드 블록](/gpu-glossary/device-software/thread-block)이 동시에 스케줄링되도록 하고, 이를 통해 [점유율(Occupancy)](/gpu-glossary/perf/occupancy)을 끌어올리는 것입니다.

[PTX](/gpu-glossary/device-software/parallel-thread-execution) 명령어 집합 아키텍처에서 사용하는 레지스터는 [공식 문서](https://docs.nvidia.com/cuda/parallel-thread-execution/#register-state-space)에 기술되어 있습니다. 반면 [SASS](/gpu-glossary/device-software/streaming-assembler)에서 사용하는 레지스터는 공개적으로 문서화되어 있지 않은 것으로 알려져 있습니다.
