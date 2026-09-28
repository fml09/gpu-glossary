---
title: CUDA 바이너리 유틸리티란 무엇인가?
---

CUDA 바이너리 유틸리티(CUDA Binary Utilities)는 [NVIDIA CUDA 컴파일러 드라이버(nvcc)](/gpu-glossary/host-software/nvcc)가 생성한 바이너리의 내부 내용을 검사하고 조작하기 위한 도구 모음입니다.

대표적인 도구인 `cuobjdump`는 전체 호스트 바이너리 또는 그 안에 포함된 CUDA 전용 `cubin` 파일의 세부 내용을 분석하고 추출하는 데 활용됩니다.

또 다른 도구인 `nvidisasm`은 `cubin` 파일을 역어셈블하는 용도로 사용됩니다. 바이너리에서 [SASS 어셈블리 코드](/gpu-glossary/device-software/streaming-assembler)를 추출할 수 있으며, 제어 흐름 그래프(Control Flow Graph)를 그리거나 어셈블리 명령어를 원본 CUDA 소스 코드 라인과 연결해 분석할 수 있습니다.

이 도구들에 대한 자세한 설명은 [CUDA 바이너리 유틸리티 공식 문서](https://docs.nvidia.com/cuda/cuda-binary-utilities/index.html)에서 찾아볼 수 있습니다.
