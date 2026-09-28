---
title: 연산 대역폭이란 무엇인가?
---

연산 대역폭(Arithmetic Bandwidth)은 시스템이 산술 연산 작업을 수행할 수 있는 [최대 처리율(Peak Rate)](/gpu-glossary/perf/peak-rate)을 의미합니다.

이는 초당 달성 가능한 산술 연산 처리량의 이론적 최댓값을 나타내며, 하드웨어의 [루프라인 모델(Roofline Model)](/gpu-glossary/perf/roofline-model)에서 '연산 한계선(compute roof)'의 높이를 결정합니다.

하나의 완전한 컴퓨팅 시스템 안에는 여러 종류의 연산 대역폭이 존재합니다. 산술 연산 처리를 담당하는 각 하드웨어 유닛 그룹마다 고유한 연산 대역폭을 가집니다.

수많은 GPU에서 가장 중요한 연산 대역폭은 부동소수점 산술 연산을 수행하는 [CUDA 코어(CUDA Core)](/gpu-glossary/device-hardware/cuda-core)의 대역폭입니다. GPU는 일반적으로 정수 연산보다 부동소수점 연산에 더 큰 대역폭을 제공합니다. 또한 [컴퓨트 통합 디바이스 아키텍처(CUDA)](/gpu-glossary/device-hardware/cuda-device-architecture)의 핵심은 과거 GPU 아키텍처와 달리 CUDA 코어와 이를 뒷받침하는 시스템들이 GPU 애플리케이션을 위한 통합 컴퓨팅 인터페이스를 제공한다는 점에 있습니다.

그러나 최근 GPU에서는 [텐서 코어(Tensor Core)](/gpu-glossary/device-hardware/tensor-core)가 도입되면서 이러한 아키텍처 통합성이 다소 완화되었습니다. 텐서 코어는 오직 행렬 곱셈 연산만을 수행하지만, [CUDA 코어](/gpu-glossary/device-hardware/cuda-core)보다 훨씬 더 높은 연산 대역폭을 제공합니다. 대략적으로 텐서 코어와 CUDA 코어 간의 대역폭 비율은 약 100:1 수준에 달합니다. 따라서 성능을 극대화하려는 [커널(Kernel)](/gpu-glossary/device-software/kernel)에게는 텐서 코어의 연산 대역폭을 활용하는 것이 가장 중요합니다.

최신 GPU의 [텐서 코어](/gpu-glossary/device-hardware/tensor-core) 연산 대역폭은 초당 1000조(경 단위) 회의 부동소수점 연산을 의미하는 페타플롭스(PFLOPS) 단위로 측정됩니다. 예를 들어 [B200 GPU](https://modal.com/blog/introducing-b200-h200)는 4비트 부동소수점(FP4) 행렬 곱셈 연산을 실행할 때 9 PFLOPS에 달하는 대역폭을 제공합니다.

암페어(Ampere)부터 블랙웰(Blackwell)까지 NVIDIA 데이터센터 GPU [스트리밍 다중처리기 아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture)의 대표적인 대역폭 수치는 아래 표와 같습니다.

| **시스템 (연산 / 메모리)**                                                                                                                                  | **연산 대역폭 (TFLOPs/s)** | **[메모리 대역폭](/gpu-glossary/perf/memory-bandwidth) (TB/s)** | **[변곡점(Ridge Point)](/gpu-glossary/perf/roofline-model) (FLOPs/byte)** |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------: | -----------------------------------------------------------------: | ----------------------------------------------------------------: |
| [A100 80GB SXM BF16 TC / HBM2e](https://www.nvidia.com/content/dam/en-zz/Solutions/Data-Center/a100/pdf/nvidia-a100-datasheet-us-nvidia-1758950-r4-web.pdf) |                                 312 |                                                                  2 |                                                               156 |
| [H100 SXM BF16 TC / HBM3](https://resources.nvidia.com/en-us-gpu-resources/h100-datasheet-24306)                                                            |                                 989 |                                                               3.35 |                                                               295 |
| [B200 BF16 TC / HBM3e](https://resources.nvidia.com/en-us-dgx-systems/dgx-b200-datasheet)                                                                   |                                2250 |                                                                  8 |                                                               281 |
| [H100 SXM FP8 TC / HBM3](https://resources.nvidia.com/en-us-gpu-resources/h100-datasheet-24306)                                                             |                                1979 |                                                               3.35 |                                                               592 |
| [B200 FP8 TC / HBM3e](https://resources.nvidia.com/en-us-dgx-systems/dgx-b200-datasheet)                                                                    |                                4500 |                                                                  8 |                                                               562 |
| [B200 FP4 TC / HBM3e](https://resources.nvidia.com/en-us-dgx-systems/dgx-b200-datasheet)                                                                    |                                9000 |                                                                  8 |                                                              1125 |
