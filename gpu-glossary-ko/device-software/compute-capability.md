---
title: 컴퓨트 성능(Compute Capability)이란 무엇인가?
---

[병렬 스레드 실행(Parallel Thread Execution, PTX)](/gpu-glossary/device-software/parallel-thread-execution) 명령어 집합의 명령어는 특정 물리적 GPU와만 호환됩니다. 명령어 집합과 [컴파일러](/gpu-glossary/host-software/nvcc) 관점에서 물리적 GPU의 세부 사양을 추상화하기 위해 사용하는 버전 관리 체계를 '컴퓨트 성능(Compute Capability)'이라고 합니다.

대부분의 컴퓨트 성능 버전 번호는 주 버전(major version)과 부 버전(minor version)이라는 두 가지 요소로 구성됩니다. NVIDIA는 [양파 껍질(onion layer)](https://docs.nvidia.com/cuda/parallel-thread-execution/#ptx-module-directives-target) 모델에 따라 주 버전과 부 버전 전반에 걸쳐 하위 호환성(이전 [PTX](/gpu-glossary/device-software/parallel-thread-execution) 코드가 최신 GPU에서도 정상 작동함)을 보장합니다.

Hopper 아키텍처에서 NVIDIA는 `9.0a`의 `a`와 같은 추가적인 버전 접미사를 도입했습니다. 이 접미사가 붙은 기능은 양파 껍질 모델에서 벗어나며, 동일한 주 버전 안에서도 향후 호환성을 보장하지 않습니다.

Blackwell 아키텍처에서는 또 다른 버전 접미사인 `10.0f`의 `f`를 도입했습니다. 이 역시 양파 껍질 모델에서 벗어나며 [시맨틱 버저닝(SemVer)](https://semver.org/) 체계에 더 가깝습니다. 즉, 부 버전 간 호환성은 보장되지만 주 버전 간 호환성은 보장되지 않습니다.

[PTX](/gpu-glossary/device-software/parallel-thread-execution) 컴파일을 위한 대상 컴퓨트 성능은 [NVIDIA CUDA 컴파일러 드라이버](/gpu-glossary/host-software/nvcc)인 `nvcc`를 호출할 때 지정할 수 있습니다. 기본적으로 컴파일러는 일치하는 [스트리밍 다중처리기(SM) 아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture)에 최적화된 [SASS](/gpu-glossary/device-software/streaming-assembler)도 함께 생성합니다. [`nvcc`](/gpu-glossary/host-software/nvcc) [문서](https://docs.nvidia.com/cuda/cuda-compiler-driver-nvcc/index.html#virtual-architectures)에서는 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor) 버전으로 나타내는 '물리적 GPU 아키텍처'와 대비하여 컴퓨트 성능을 '가상 GPU 아키텍처'라고 부릅니다.

각 컴퓨트 성능 버전별 기술 사양은 [NVIDIA CUDA C 프로그래밍 가이드의 Compute Capability 섹션](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html)에서 확인할 수 있습니다.
