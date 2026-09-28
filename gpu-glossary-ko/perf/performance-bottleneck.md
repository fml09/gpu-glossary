---
title: 성능 병목이란 무엇인가?
---

병의 좁은 목(bottleneck)이 액체가 쏟아져 나오는 속도를 제한하듯이, 시스템의 은유적 표현인 '성능 병목(Performance Bottleneck)'은 전체 작업이 완료되는 처리 속도를 제약합니다.

![이와 같은 [루프라인 다이어그램](/gpu-glossary/perf/roofline-model)은 처리량 중심 시스템에서 성능 병목을 신속하게 식별하는 데 사용됩니다. Williams, Waterman, Patterson (2008)의 논문 다이어그램을 참고하여 작성하였습니다.](https://modal-cdn.com/gpu-glossary/light-roofline-model.svg)

병목은 성능 최적화의 주된 표적입니다. 최적화의 교과서적인 접근법은 다음과 같은 세 단계로 이루어집니다.

- 병목 지점을 파악한다.
- 해당 지점이 더 이상 병목이 되지 않을 때까지 병목을 완화한다.
- 새롭게 드러난 병목 지점을 대상으로 이 과정을 반복한다.

이러한 접근법은 엘리야후 골드랫(Eliyahu Goldratt)의 [제약 이론(Theory of Constraints)](https://en.wikipedia.org/wiki/Theory_of_constraints) 등을 통해 정형화되었으며, 토요타 생산 방식을 전 세계 제조업체로 전파하고 나아가 소프트웨어 엔지니어링 및 운영 전반에 확산되는 데 기여하였습니다.

Horace He는 [Jane Street 강연](https://youtu.be/139UPjoq7Kw?t=1229)에서 GPU 기반 프로그램의 [커널](/gpu-glossary/device-software/kernel)이 수행하는 작업을 세 가지 범주로 분류하였습니다.

- 연산(Compute): [CUDA 코어](/gpu-glossary/device-hardware/cuda-core) 또는 [텐서 코어](/gpu-glossary/device-hardware/tensor-core)에서 부동소수점 연산 실행
- 메모리(Memory): 시스템의 [메모리 계층 구조](/gpu-glossary/device-software/memory-hierarchy) 내에서 데이터 이동
- 오버헤드(Overhead): 그 외 모든 작업

따라서 GPU [커널](/gpu-glossary/device-software/kernel)의 성능 병목 현상 역시 크게 세 가지 범주로 분류할 수 있습니다.

- [연산 제약(Compute Bound)](/gpu-glossary/perf/compute-bound) 커널: 대규모 행렬 곱셈처럼 연산 유닛의 [연산 대역폭](/gpu-glossary/perf/arithmetic-bandwidth)에 의해 병목이 발생하는 경우
- [메모리 제약(Memory Bound)](/gpu-glossary/perf/memory-bound) 커널: 대규모 벡터 곱셈처럼 [메모리 서브시스템 대역폭](/gpu-glossary/perf/memory-bandwidth)에 의해 병목이 발생하는 경우
- [오버헤드 제약(Overhead Bound)](/gpu-glossary/perf/overhead) 커널: 작은 크기의 배열 연산처럼 실행 지연 시간에 의해 병목이 발생하는 경우

[루프라인 모델(Roofline Model)](/gpu-glossary/perf/roofline-model) 분석을 활용하면 프로그램의 성능이 연산/[산술 대역폭](/gpu-glossary/perf/arithmetic-bandwidth)과 [메모리 대역폭](/gpu-glossary/perf/memory-bandwidth) 중 어느 쪽에 의해 병목 현상을 겪고 있는지 신속하게 판별할 수 있습니다.

<small>물론 이론상으로는 *모든* 리소스가 병목이 될 수 있습니다. 예를 들어 전력 공급과 발열 해소 한계로 인해 일부 GPU의 성능이 이론적 최대치 밑으로 제한되기도 합니다. L2 캐시의 전력을 [스트리밍 다중처리기](/gpu-glossary/device-hardware/streaming-multiprocessor)로 재분배하여 4%의 엔드투엔드 성능 향상을 달성한 [NVIDIA의 아티클](https://developer.nvidia.com/blog/nvidia-sets-new-generative-ai-performance-and-scale-records-in-mlperf-training-v4-0/)이나, 트랜지스터 스위칭에 요구되는 전력 소모량 차이로 인해 입력 데이터 패턴에 따라 행렬 곱셈 성능이 달라질 수 있음을 보여주는 [Horace He의 글](https://www.thonking.ai/p/strangely-matrix-multiplications)을 참조하시기 바랍니다. 그럼에도 불구하고 연산과 메모리는 여전히 가장 중요하며 가장 흔하게 마주치는 핵심 병목 요인입니다.</small>
