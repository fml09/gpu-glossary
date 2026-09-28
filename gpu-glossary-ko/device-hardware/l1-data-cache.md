---
title: L1 데이터 캐시란 무엇인가?
---

L1 데이터 캐시(L1 Data Cache)는 [스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor) 전용(private) 메모리입니다.

![H100 SM 내부 아키텍처. L1 데이터 캐시가 하늘색으로 표시되어 있습니다. NVIDIA의 [H100 백서](https://modal-cdn.com/gpu-glossary/gtc22-whitepaper-hopper.pdf)를 기반으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-gh100-sm.svg)

각 SM은 자신에게 스케줄링된 [스레드 그룹](/gpu-glossary/device-software/thread-block) 사이에 이 메모리를 분할하여 할당합니다.

L1 데이터 캐시는 실제 연산을 수행하는 구성 요소(예: [CUDA 코어](/gpu-glossary/device-hardware/cuda-core))와 동일한 위치에 직접 배치되어 있으며, 속도 차이도 연산 유닛보다 대략 10배 정도 느린 수준에 불과합니다.

L1 데이터 캐시는 CPU 캐시와 레지스터, 그리고 [Groq LPU 메모리 서브시스템](https://groq.com/wp-content/uploads/2023/05/GroqISCAPaper2022_ASoftwareDefinedTensorStreamingMultiprocessorForLargeScaleMachineLearning-1.pdf) 등에서 사용하는 기본 반도체 메모리 셀과 동일한 SRAM으로 구현됩니다. L1 데이터 캐시 접근은 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)의 [로드/스토어 유닛(LSU)](/gpu-glossary/device-hardware/load-store-unit)이 담당합니다.

CPU 역시 L1 캐시를 탑재하고 있습니다. 다만 CPU의 L1 캐시는 하드웨어가 완전히 자동으로 관리하는 반면, GPU에서는 [CUDA C](/gpu-glossary/host-software/cuda-c)와 같은 고급 언어를 사용할 때조차 상당 부분을 프로그래머가 직접 제어하고 관리합니다.

H100 GPU의 각 SM에 탑재된 L1 데이터 캐시는 256KiB(2,097,152비트)를 저장할 수 있습니다. 132개의 SM을 탑재한 H100 SXM 5 전체로 환산하면 총 33MiB(242,221,056비트)의 캐시 용량에 달합니다.
