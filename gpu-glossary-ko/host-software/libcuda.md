---
title: libcuda.so란 무엇인가?
---

`libcuda.so`는 리눅스 시스템에서 [CUDA 드라이버 API(CUDA Driver API)](/gpu-glossary/host-software/cuda-driver-api)를 구현한 바이너리 공유 객체(Shared Object) 파일의 대표적인 이름입니다.

CUDA 프로그램은 이 라이브러리를 동적으로 링크하여 사용합니다. 시스템에 이 파일이 존재하지 않는다면 일반적으로 NVIDIA 드라이버가 올바르게 설치되지 않았음을 의미합니다.
