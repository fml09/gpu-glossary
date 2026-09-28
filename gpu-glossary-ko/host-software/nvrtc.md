---
title: NVIDIA 런타임 컴파일러(NVRTC)란 무엇인가?
abbreviation: nvrtc
---

NVIDIA 런타임 컴파일러(NVIDIA Runtime Compiler, `nvrtc`)는 CUDA C/C++ 코드를 런타임에 컴파일해 주는 라이브러리입니다. 별도의 외부 프로세스로 [NVIDIA CUDA 컴파일러 드라이버(nvcc)](/gpu-glossary/host-software/nvcc)를 구동하지 않고도 라이브러리 함수 호출만으로 [CUDA C++](/gpu-glossary/host-software/cuda-c) 코드를 [PTX](/gpu-glossary/device-software/parallel-thread-execution)로 즉각 컴파일할 수 있습니다. 런타임에 동적으로 C/C++ 커널 코드를 생성하고 이를 GPU에서 즉시 실행하고자 하는 라이브러리 및 딥러닝 프레임워크에서 널리 활용합니다.

이렇게 생성된 [PTX](/gpu-glossary/device-software/parallel-thread-execution) 코드는 실행 시점에 중간 표현(IR)에서 실제 GPU 기계어인 [SASS 어셈블리](/gpu-glossary/device-software/streaming-assembler)로 한 단계 더 JIT(Just-In-Time) 컴파일됩니다. 이 과정은 [NVIDIA GPU 드라이버](/gpu-glossary/host-software/nvidia-gpu-drivers)가 처리하며, NVRTC 자체의 컴파일 동작과는 명확히 구분됩니다. 상위 아키텍처 호환성을 위해 PTX 코드를 포함하여 배포된 일반 CUDA 바이너리 역시 실행 시점에 동일한 드라이버 JIT 컴파일 과정을 거칩니다.

NVRTC는 소스 코드가 공개되지 않은 독점 라이브러리입니다. 관련 문서는 [NVRTC 공식 문서](https://docs.nvidia.com/cuda/nvrtc/index.html)에서 확인할 수 있습니다.
