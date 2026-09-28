---
title: CUDA 런타임 API란 무엇인가?
---

CUDA 런타임 API(CUDA Runtime API)는 [CUDA 드라이버 API](/gpu-glossary/host-software/cuda-driver-api)를 래핑하여 동일한 기능을 보다 높은 수준의 인터페이스로 다룰 수 있게 지원하는 상위 계층 API입니다.

![CUDA 툴킷 구조. CUDA 런타임 API는 드라이버 API를 감싸 애플리케이션 프로그래밍에 훨씬 적합한 형태를 제공합니다. Professional CUDA C Programming Guide 참고하여 수정.](https://modal-cdn.com/gpu-glossary/light-cuda-toolkit.svg)

개발 편의성이 뛰어나기 때문에 일반적으로 [드라이버 API](/gpu-glossary/host-software/cuda-driver-api)보다 런타임 API를 우선적으로 사용합니다. 다만 커널 론치 제어나 컨텍스트(Context) 관리 방식에서 약간의 세부적인 차이가 존재합니다. 이에 관한 자세한 내용은 CUDA 런타임 API 설명서의 [비교 안내 섹션](https://docs.nvidia.com/cuda/cuda-runtime-api/driver-vs-runtime-api.html#driver-vs-runtime-api)에서 확인할 수 있습니다.

[NVIDIA CUDA 툴킷 최종 사용자 라이선스 계약(EULA) 부록 A](https://docs.nvidia.com/cuda/eula/index.html#attachment-a)에 따라 런타임 API는 바이너리에 정적으로 링크할 수도 있고 동적으로 링크할 수도 있습니다. 동적 링크를 위한 공유 객체 라이브러리 파일은 리눅스 시스템에서 보통 [libcudart.so](/gpu-glossary/host-software/libcudart)라는 이름으로 제공됩니다.

CUDA 런타임 API는 독점 소프트웨어로서 소스 코드가 공개되지 않습니다. 공식 문서는 [CUDA 런타임 API 레퍼런스](https://docs.nvidia.com/cuda/cuda-runtime-api/index.html)에서 찾아볼 수 있습니다.
