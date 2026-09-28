---
title: libcudart.so란 무엇인가?
---

`libcudart.so`는 리눅스 시스템에서 [CUDA 런타임 API(CUDA Runtime API)](/gpu-glossary/host-software/cuda-runtime-api)를 구현한 바이너리 공유 객체(Shared Object) 파일의 대표적인 이름입니다.

배포되는 CUDA 바이너리는 이 파일을 정적으로 링크하는 경우가 많지만, PyTorch처럼 CUDA 툴킷을 기반으로 구축된 라이브러리와 프레임워크는 대개 이를 동적으로 로드합니다.
