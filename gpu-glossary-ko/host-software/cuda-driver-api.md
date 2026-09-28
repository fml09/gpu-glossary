---
title: CUDA 드라이버 API란 무엇인가?
---

[CUDA 드라이버 API(CUDA Driver API)](https://docs.nvidia.com/cuda/cuda-driver-api/index.html)는 NVIDIA CUDA 드라이버의 유저 공간(Userspace) 구성 요소입니다. C 표준 라이브러리 사용자에게 익숙한 형태의 기능을 제공하며, 대표적으로 GPU 디바이스에 [메모리](/gpu-glossary/device-software/global-memory)를 할당하는 `cuMalloc` 함수 등이 있습니다.

![CUDA 툴킷 구조. CUDA 드라이버 API는 애플리케이션 또는 기타 툴킷 구성 요소와 GPU 사이에 위치합니다. Professional CUDA C Programming Guide 참고하여 수정.](https://modal-cdn.com/gpu-glossary/light-cuda-toolkit.svg)

CUDA 프로그램을 작성할 때 드라이버 API를 직접 호출하는 경우는 매우 드뭅니다. 대다수 프로그램은 더 편리한 추상화를 제공하는 [CUDA 런타임 API](/gpu-glossary/host-software/cuda-runtime-api)를 사용합니다. 두 API 간의 차이는 CUDA 드라이버 API 문서의 [비교 설명 섹션](https://docs.nvidia.com/cuda/cuda-driver-api/driver-vs-runtime-api.html#driver-vs-runtime-api)에서 확인할 수 있습니다.

CUDA 드라이버 API는 일반적으로 정적으로 링크하지 않고 동적으로 링크하며, 리눅스 시스템에서는 흔히 [libcuda.so](/gpu-glossary/host-software/libcuda)라는 라이브러리 이름으로 제공됩니다.

CUDA 드라이버 API는 바이너리 호환성(Binary Compatibility)을 보장합니다. 이전 버전의 CUDA 드라이버 API를 대상으로 빌드된 애플리케이션이라 하더라도, 더 새로운 버전의 드라이버 API가 설치된 시스템에서 문제없이 실행할 수 있습니다. 즉, 운영체제의 바이너리 로더가 더 최신 버전의 드라이버 라이브러리를 로드하더라도 프로그램은 동일하게 동작합니다.

[CUDA C/C++](/gpu-glossary/host-software/cuda-c) 애플리케이션의 배포에 관한 상세 지침은 NVIDIA의 [CUDA C/C++ 모범 사례 가이드(CUDA C/C++ Best Practices Guide)](https://docs.nvidia.com/cuda/cuda-c-best-practices-guide)에 기술되어 있습니다.

CUDA 드라이버 API는 소스 코드가 공개되지 않은 독점 소프트웨어입니다. 관련 기술 문서는 [CUDA 드라이버 API 공식 문서](https://docs.nvidia.com/cuda/cuda-driver-api/index.html)에서 확인할 수 있습니다.

일반적으로 널리 쓰이지는 않지만, [LibreCuda](https://github.com/mikex86/LibreCuda)나 [tinygrad](https://github.com/tinygrad)처럼 CUDA 드라이버 API의 오픈 소스 대안을 구현하거나 활용하려는 프로젝트도 존재합니다. 구체적인 동작 방식은 [tinygrad의 소스 코드](https://github.com/tinygrad/tinygrad/blob/77f7ddf62a78218bee7b4f7b9ff925a0e581fcad/tinygrad/runtime/ops_nv.py)에서 살펴볼 수 있습니다.
