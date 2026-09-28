---
title: 로드/스토어 유닛(LSU)이란 무엇인가?
abbreviation: LSU
---

로드/스토어 유닛(Load/Store Unit, LSU)은 GPU 메모리 서브시스템에 데이터를 읽어오거나(load) 저장하는(store) 요청을 발행하는 유닛입니다.

![H100 SM 내부 아키텍처. 로드/스토어 유닛은 [특수 기능 유닛](/gpu-glossary/device-hardware/special-function-unit)과 함께 분홍색으로 표시되어 있습니다. NVIDIA의 [H100 백서](https://modal-cdn.com/gpu-glossary/gtc22-whitepaper-hopper.pdf)를 기반으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-gh100-sm.svg)

[CUDA 프로그래머](/gpu-glossary/host-software/cuda-software-platform) 입장에서 가장 중요한 점은, LSU가 [스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor) 내부 온칩 SRAM [L1 데이터 캐시](/gpu-glossary/device-hardware/l1-data-cache)와 직접 통신하고, 오프칩 디바이스 메모리인 [전역 RAM](/gpu-glossary/device-hardware/gpu-ram)과 간접적으로 통신한다는 사실입니다. 이 두 메모리는 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)의 [메모리 계층 구조](/gpu-glossary/device-software/memory-hierarchy)에서 각각 가장 가까운 계층과 가장 바깥쪽 계층을 구성합니다.
