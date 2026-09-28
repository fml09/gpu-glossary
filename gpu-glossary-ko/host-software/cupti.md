---
title: NVIDIA CUDA 프로파일링 도구 인터페이스(CUPTI)란 무엇인가?
abbreviation: CUPTI
---

NVIDIA CUDA 프로파일링 도구 인터페이스(CUPTI, NVIDIA CUDA Profiling Tools Interface)는 GPU에서 실행되는 [CUDA C++](/gpu-glossary/host-software/cuda-c), [PTX](/gpu-glossary/device-software/parallel-thread-execution), [SASS](/gpu-glossary/device-software/streaming-assembler) 코드의 프로파일링을 수행하기 위한 일련의 API를 제공합니다. 무엇보다 CUPTI는 CPU 호스트와 GPU 디바이스 사이의 타임스탬프를 정밀하게 동기화하는 핵심 기능을 담당합니다.

CUPTI 인터페이스는 [Nsight Systems 프로파일러](/gpu-glossary/host-software/nsight-systems)나 [PyTorch 프로파일러](https://modal.com/docs/examples/torch_profiling)와 같은 다양한 성능 분석 도구에서 폭넓게 활용됩니다.

자세한 기술 문서는 [CUPTI 공식 문서](https://docs.nvidia.com/cupti/)에서 확인할 수 있습니다.

Modal에서 구동되는 GPU 애플리케이션에 프로파일링 도구를 적용하는 방법은 [Modal 가이드 예제](/docs/examples/torch_profiling)를 참고하시기 바랍니다.
