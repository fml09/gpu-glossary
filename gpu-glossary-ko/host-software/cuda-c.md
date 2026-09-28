---
title: CUDA C++ 프로그래밍 언어란 무엇인가?
---

CUDA C++는 C++ 프로그래밍 언어를 확장하여 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)을 구현한 언어입니다.

CUDA C++는 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)을 완성하기 위해 C++에 여러 핵심 기능을 추가합니다.

- **`__global__` 키워드를 통한 [커널](/gpu-glossary/device-software/kernel) 정의**: CUDA [커널](/gpu-glossary/device-software/kernel)은 포인터를 인자로 받고 반환형이 `void`인 C++ 함수 형태로 구현되며, 함수 선언 앞에 이 키워드를 지정합니다.
- **`<<<>>>` 삼중 꺾쇠괄호를 통한 [커널](/gpu-glossary/device-software/kernel) 실행(Launch)**: CPU 호스트는 [스레드 블록 그리드](/gpu-glossary/device-software/thread-block-grid)의 크기와 구성을 설정하는 삼중 꺾쇠괄호 문법을 사용하여 커널을 실행합니다.
- **디바이스 하드웨어 제어 기능**: `__shared__` 키워드를 사용한 [공유 메모리](/gpu-glossary/device-software/shared-memory) 할당, `__syncthreads()` 내장 함수를 통한 배리어 동기화(Barrier Synchronization), 내장 변수인 `blockDim` 및 `threadIdx`를 활용한 [스레드 블록](/gpu-glossary/device-software/thread-block)과 [스레드](/gpu-glossary/device-software/thread) 인덱싱 기능 등을 제공합니다.

CUDA C++ 프로그램은 `gcc` 같은 호스트 C/C++ 컴파일러 드라이버와 [NVIDIA CUDA 컴파일러 드라이버](/gpu-glossary/host-software/nvcc)인 `nvcc`를 결합하여 컴파일합니다.

[Modal](https://modal.com) 환경에서 CUDA C++를 사용하는 구체적인 지침은 [CUDA 가이드 문서](https://modal.com/docs/guide/cuda)에서 확인할 수 있습니다.
