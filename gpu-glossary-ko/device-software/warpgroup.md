---
title: 워프그룹이란 무엇인가?
---

워프그룹(Warpgroup)은 첫 번째 워프의 워프 순위(warp-rank)가 4의 배수인 연속된 4개의 [워프](/gpu-glossary/device-software/warp) 묶음입니다.

워프그룹 수준의 명령어가 디스패치되면 워프그룹당 4개 워프 × 워프당 32개 스레드로 총 128개의 [스레드](/gpu-glossary/device-software/thread)가 함께 협력하여 동작합니다. 이처럼 더 큰 단위로 작업을 처리하면 워프 간 명시적인 동기화가 필요하지 않으며, 명령어당 더 큰 크기의 문제(특히 더 큰 규모의 행렬 곱셈)를 처리할 수 있습니다. 행렬 곱셈의 규모가 커질수록 최신 데이터센터 GPU에 탑재된 [텐서 코어](/gpu-glossary/device-hardware/tensor-core)의 막대한 [연산 대역폭](/gpu-glossary/perf/arithmetic-bandwidth)을 훨씬 더 효과적으로 포화시킬(saturate) 수 있습니다.

워프그룹은 NVIDIA의 Hopper [스트리밍 다중처리기(SM) 아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture)에서 처음 도입되었으며, `wgmma.mma_async`와 같은 워프그룹 수준의 행렬 곱셈 명령어를 지원하는 데 사용됩니다. 자세한 내용은 [Colfax 연구소 블로그 글](https://research.colfax-intl.com/cutlass-tutorial-wgmma-hopper/)을 참조하십시오. 또한 워프그룹은 [Flash Attention 4](https://modal.com/blog/reverse-engineer-flash-attention-4)와 같은 고성능 Hopper 및 Blackwell [커널](/gpu-glossary/device-software/kernel)에서 파이프라인 구성 요소를 조직화하는 데 핵심적인 역할을 담당합니다.

[병렬 스레드 실행(PTX)](/gpu-glossary/device-software/parallel-thread-execution) IR에서 특정 워프의 워프 순위(warp-rank)는 다음과 같이 계산됩니다.

```cpp
int linearIdx = (%tid.x + %tid.y * %ntid.x  + %tid.z * %ntid.x * %ntid.y);
int warpRank = linearIdx / 32;
```

여기서 `tid`는 특수 PTX [레지스터](/gpu-glossary/device-software/registers)를 통해 접근하는 스레드 인덱스입니다.

따라서 8개 워프로 구성된 디스패치에서 유효한 워프그룹은 다음과 같습니다.

- **워프그룹 0**: 워프 순위 0, 1, 2, 3
- **워프그룹 1**: 워프 순위 4, 5, 6, 7

워프 순위 정렬 제한을 두는 구체적인 목적은 공식 문서에 명시되어 있지 않은 것으로 알려져 있습니다. 다만 최신 데이터센터 GPU의 [스트리밍 다중처리기(SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)는 각각 독립적인 [워프 스케줄러](/gpu-glossary/device-hardware/warp-scheduler)와 텐서 코어를 갖춘 4개의 하위 유닛(sub-unit)으로 나뉘어 구성되어 있는 구조와 관련이 깊은 것으로 추정됩니다.
