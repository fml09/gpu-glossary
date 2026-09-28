---
title: 최대 처리율이란 무엇인가?
---

최대 처리율(Peak Rate)은 하드웨어 시스템이 작업을 처리할 수 있는 이론상의 최대 속도를 의미합니다.

최대 처리율은 모든 실행 유닛이 완벽한 효율로 최대 용량에서 가동될 때 도달할 수 있는 GPU 성능의 절대적인 상한선을 나타냅니다. 이는 [레지스터](/gpu-glossary/device-software/registers), [메모리 대역폭](/gpu-glossary/perf/memory-bandwidth), 동기화 배리어 등 그 어떠한 리소스 제약도 [병목 현상](/gpu-glossary/perf/performance-bottleneck)을 일으키지 않는 이상적인 동작 상태를 가정합니다.

최대 처리율은 실제로 달성된 성능을 평가하는 절대적인 척도가 됩니다. [루프라인 분석](/gpu-glossary/perf/roofline-model)에서는 [연산 제약](/gpu-glossary/perf/compute-bound) 한계선(roof)을 결정하며, [파이프라인 활용률](/gpu-glossary/perf/pipe-utilization) 지표에서는 활용률 계산의 분모 역할을 담당할 뿐만 아니라 [GPU 활용률을 판단하는 최종적인 기준](https://modal.com/blog/gpu-utilization-guide)이 됩니다.

NVIDIA 엔지니어들은 이를 물리학 법칙이 프로그램 속도에 부과하는 한계라는 의미에서 '빛의 속도(Speed of Light)'라는 은유적인 표현으로 자주 부릅니다.

최대 처리율은 각 GPU 아키텍처의 고정된 하드웨어 사양으로부터 직접 계산됩니다.

예를 들어 132개의 SM을 탑재하고 각 SM마다 128개의 FP32 코어를 갖춘 [NVIDIA H100 GPU](https://resources.nvidia.com/en-us-hopper-architecture/nvidia-h100-tensor-c)는 코어당 2회의 부동소수점 연산으로 구성된 단정밀도 융합 곱셈 덧셈(FMA) 명령어를 사이클당 1회 발행할 수 있습니다. 이는 클록당 33,792개의 [명령어(Instructions per clock, IPC)](https://en.wikipedia.org/wiki/Instructions_per_cycle)에 해당합니다. H100의 연산 서브시스템 클록은 FP32 코어를 가동할 때 최대 1980 MHz(초당 19억 8천만 클록)로 동작할 수 있으므로, 최대 처리율은 66조 9,080억 FLOPS, 즉 66.9 TFLOPS가 됩니다.

이 수치는 [NVIDIA H100 백서](https://resources.nvidia.com/en-us-hopper-architecture/nvidia-h100-tensor-c)에 명시된 텐서 코어를 사용하지 않는 피크 FP32 TFLOPS 사양과 정확히 일치합니다.
