---
title: CUDA 소프트웨어 플랫폼이란 무엇인가?
---

CUDA 소프트웨어 플랫폼(CUDA Software Platform)은 CUDA 프로그램을 개발하기 위한 소프트웨어 도구 및 라이브러리의 모음입니다.

CUDA는 *Compute Unified Device Architecture*(통합 컴퓨팅 디바이스 아키텍처)의 약어입니다. 문맥에 따라 "CUDA"라는 명칭은 서로 다른 여러 개념을 나타냅니다. [상위 수준의 디바이스 아키텍처](/gpu-glossary/device-hardware/cuda-device-architecture)를 의미하기도 하고, [해당 설계를 따르는 아키텍처를 위한 병렬 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)을 가리키기도 하며, C/C++ 같은 고급 언어를 확장하여 이 프로그래밍 모델을 지원하는 소프트웨어 플랫폼을 뜻하기도 합니다.

CUDA의 근본적인 비전은 [Lindholm 외 연구진의 2008년 백서](https://www.cs.cmu.edu/afs/cs/academic/class/15869-f11/www/readings/lindholm08_tesla.pdf)에 상세히 기술되어 있습니다. 이 논문은 NVIDIA 공식 기술 문서에 등장하는 여러 핵심 논리, 다이어그램, 주요 표현의 원천이 되는 자료이므로 읽어보기를 강력히 권장합니다.

여기서는 CUDA *소프트웨어 플랫폼*에 집중하여 설명합니다. FORTRAN, 파이썬, [BASIC](/gpu-glossary/host-software/cutile-basic) 등 다른 언어를 위한 CUDA 소프트웨어 플랫폼도 존재하지만, 가장 널리 쓰이는 표준인 [CUDA C++](/gpu-glossary/host-software/cuda-c) 버전을 중심으로 다룹니다.

CUDA 플랫폼은 크게 두 부류의 구성 요소로 나눌 수 있습니다. 아래 다이어그램처럼 애플리케이션을 *빌드*할 때 사용하는 [NVIDIA CUDA 컴파일러 드라이버(nvcc)](/gpu-glossary/host-software/nvcc) 툴체인과 애플리케이션 *내부*나 *외부*에서 호출되는 [CUDA 드라이버 API](/gpu-glossary/host-software/cuda-driver-api) 및 [CUDA 런타임 API](/gpu-glossary/host-software/cuda-runtime-api)가 대표적입니다.

![CUDA 툴킷 구조. Professional CUDA C Programming Guide 참고하여 수정.](https://modal-cdn.com/gpu-glossary/light-cuda-toolkit.svg)

이러한 기본 API 위에는 선형대수 연산을 다루는 [cuBLAS](/gpu-glossary/host-software/cublas), 심층 신경망을 지원하는 [cuDNN](/gpu-glossary/host-software/cudnn)처럼 범용 및 도메인 특화 고성능 [커널](/gpu-glossary/device-software/kernel) 라이브러리가 배치되어 있습니다. 나아가 [CUTLASS](/gpu-glossary/host-software/cutlass)와 [CuTe DSL](/gpu-glossary/host-software/cute-dsl)처럼 개발자가 고성능 커널을 직접 설계하고 합성할 수 있도록 돕는 고급 도구들도 제공됩니다.
