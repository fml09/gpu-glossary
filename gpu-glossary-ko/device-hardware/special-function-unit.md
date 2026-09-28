---
title: 특수 기능 유닛(SFU)이란 무엇인가?
abbreviation: SFU
---

[스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor) 내 특수 기능 유닛(Special Function Unit, SFU)은 특정 산술 연산을 가속하는 하드웨어 유닛입니다.

![H100 SM 내부 아키텍처. 특수 기능 유닛은 [로드/스토어 유닛](/gpu-glossary/device-hardware/load-store-unit)과 함께 분홍색으로 표시되어 있습니다. NVIDIA의 [H100 백서](https://modal-cdn.com/gpu-glossary/gtc22-whitepaper-hopper.pdf)를 기반으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-gh100-sm.svg)

신경망 워크로드에서 자주 쓰이는 대표적인 예로 `exp`, `sin`, `cos` 같은 초월함수(transcendental function) 연산이 있습니다.

SFU와 관련된 [SASS(Streaming Assembler)](/gpu-glossary/device-software/streaming-assembler) 명령어는 대개 `MUFU`로 시작합니다(예: `MUFU.SQRT`, `MUFU.EX2`). [CUDA C++](/gpu-glossary/host-software/cuda-c) 내장 함수(intrinsic) `expf` 구현에 `MUFU.EX2` 명령어가 쓰인 어셈블리 예시는 [Godbolt 링크](https://godbolt.org/z/WGh3rPe83)에서 확인할 수 있습니다.
