---
title: NVIDIA GPU 드라이버란 무엇인가?
---

NVIDIA GPU 드라이버는 호스트 프로그램 또는 호스트 운영체제와 GPU 디바이스 간의 상호작용을 중재합니다. 애플리케이션이 GPU 드라이버와 통신할 때 사용하는 주요 인터페이스는 상위 수준부터 순서대로 [CUDA 런타임 API](/gpu-glossary/host-software/cuda-runtime-api)와 [CUDA 드라이버 API](/gpu-glossary/host-software/cuda-driver-api)가 있습니다.

![CUDA 툴킷 구조. NVIDIA GPU 드라이버는 GPU와 직접 통신하는 유일한 구성 요소입니다. Professional CUDA C Programming Guide 참고하여 수정.](https://modal-cdn.com/gpu-glossary/light-cuda-toolkit.svg)

NVIDIA는 리눅스용 오픈 GPU [커널 모듈](/gpu-glossary/host-software/nvidia-ko)의 [소스 코드](https://github.com/NVIDIA/open-gpu-kernel-modules)를 공개했습니다.
