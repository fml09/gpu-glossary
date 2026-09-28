---
title: 전역 메모리란 무엇인가?
---

![전역 메모리는 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)의 [메모리 계층 구조](/gpu-glossary/device-software/memory-hierarchy)에서 가장 높은 수준에 위치하며, [GPU RAM](/gpu-glossary/device-hardware/gpu-ram)에 저장됩니다. NVIDIA의 [CUDA Refresher: The CUDA Programming Model](https://developer.nvidia.com/blog/cuda-refresher-cuda-programming-model/) 및 [CUDA C++ 프로그래밍 가이드](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programming-model) 다이어그램을 바탕으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-cuda-programming-model.svg)

[CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)에서는 [스레드 계층 구조](/gpu-glossary/device-software/thread-hierarchy)의 각 수준마다 [메모리 계층 구조](/gpu-glossary/device-software/memory-hierarchy)에서 이에 대응하는 메모리에 접근할 수 있습니다. 이 메모리는 스레드 간 협력 및 통신에 사용되며, 하드웨어나 런타임이 아닌 프로그래머가 직접 관리합니다.

이 메모리 계층 구조에서 가장 상위에 있는 계층이 바로 전역 메모리(Global Memory)입니다. 전역 메모리는 유효 범위(scope)와 생명 주기(lifetime) 측면에서 모두 전역적입니다. 즉, [스레드 블록 그리드](/gpu-glossary/device-software/thread-block-grid) 내의 모든 [스레드](/gpu-glossary/device-software/thread)가 전역 메모리에 접근할 수 있으며, 그 생명 주기는 프로그램 실행 전체 기간 동안 유지됩니다.

전역 메모리에 위치한 자료구조에 접근할 때는 CPU 메모리와 마찬가지로 원자적 연산 명령어(atomic instruction)를 사용하여 모든 접근 주체 간 동기화를 수행할 수 있습니다. 동일한 [협력형 스레드 배열(CTA)](/gpu-glossary/device-software/cooperative-thread-array) 내부에서는 배리어 등을 활용하여 더욱 긴밀하게 동기화할 수 있습니다.

[메모리 계층 구조](/gpu-glossary/device-software/memory-hierarchy)의 이 계층은 일반적으로 [GPU RAM(VRAM)](/gpu-glossary/device-hardware/gpu-ram)에 구현되며, [CUDA 드라이버 API](/gpu-glossary/host-software/cuda-driver-api) 또는 [CUDA 런타임 API](/gpu-glossary/host-software/cuda-runtime-api)가 제공하는 메모리 할당자를 통해 호스트에서 할당합니다.

'전역(global)'이라는 용어는 공교롭게도 [CUDA C/C++](/gpu-glossary/host-software/cuda-c)의 `__global__` 키워드와 충돌합니다. `__global__` 키워드는 호스트에서 호출되어 디바이스에서 실행되는 함수([커널](/gpu-glossary/device-software/kernel))를 수식하는 데 쓰이는 반면, 전역 메모리는 디바이스상에만 존재하는 메모리 공간이기 때문입니다. 초기 CUDA 아키텍트였던 니콜라스 윌트(Nicholas Wilt)는 저서인 [_CUDA Handbook_](https://www.cudahandbook.com/)에서 이러한 명명 방식이 "개발자에게 극도의 혼란을 안겨주기 위해" 선택되었다며 짓궂게 지적하기도 했습니다.
