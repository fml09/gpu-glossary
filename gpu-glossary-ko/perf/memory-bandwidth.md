---
title: 메모리 대역폭이란 무엇인가?
---

메모리 대역폭(Memory Bandwidth)은 [메모리 계층 구조](/gpu-glossary/device-software/memory-hierarchy)의 서로 다른 계층 간에 데이터를 전송할 수 있는 최대 속도를 의미합니다.

이는 초당 이동 가능한 데이터의 양(바이트/초)에 대한 이론적 최대 처리량을 나타내며, 하드웨어의 [루프라인 모델(Roofline Model)](/gpu-glossary/perf/roofline-model)에서 '메모리 한계선(memory roof)'의 기울기를 결정합니다.

하나의 완전한 시스템 안에는 [메모리 계층 구조](/gpu-glossary/device-software/memory-hierarchy)의 각 계층 사이에 수많은 종류의 메모리 대역폭이 존재합니다.

그중에서도 가장 핵심적인 대역폭은 [GPU RAM](/gpu-glossary/device-hardware/gpu-ram)과 [스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)의 [레지스터 파일](/gpu-glossary/device-hardware/register-file) 사이에 형성되는 대역폭입니다. 대부분의 [커널](/gpu-glossary/device-software/kernel)이 다루는 [작업 집합(Working Set)](https://en.wikipedia.org/wiki/Working_set_size)은 메모리 계층 구조의 상위 캐시에 다 들어가지 못하고 오직 [GPU RAM](/gpu-glossary/device-software/memory-hierarchy)에만 담길 수 있기 때문입니다. 이러한 이유로 GPU [커널](/gpu-glossary/device-software/kernel)의 성능을 [루프라인 모델](/gpu-glossary/perf/roofline-model)로 분석할 때 이 대역폭을 주요 지표로 활용합니다.

최신 GPU의 메모리 대역폭은 초당 테라바이트(TB/s) 단위로 측정됩니다. 예를 들어 [B200 GPU](https://modal.com/blog/introducing-b200-h200)는 HBM3e 메모리와 초당 8 TB에 달하는 양방향 메모리 대역폭을 갖추고 있습니다. 이는 해당 GPU에 탑재된 [텐서 코어](/gpu-glossary/device-hardware/tensor-core)의 [연산 대역폭](/gpu-glossary/perf/arithmetic-bandwidth)에 비해 훨씬 낮은 수치이며, 그 결과 [변곡점](/gpu-glossary/perf/roofline-model)에서의 요구 [연산 강도](/gpu-glossary/perf/arithmetic-intensity)가 더욱 높아지게 됩니다.

암페어(Ampere)부터 블랙웰(Blackwell)까지 NVIDIA 데이터센터 GPU [스트리밍 다중처리기 아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture)의 대표적인 대역폭 수치는 아래 표와 같습니다.

| **시스템 (연산 / 메모리)**                                                                                                                                   | **[연산 대역폭](/gpu-glossary/perf/arithmetic-bandwidth) (TFLOPs/s)** | **메모리 대역폭 (TB/s)** | **[변곡점(Ridge Point)](/gpu-glossary/perf/roofline-model) (FLOPs/byte)** |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------- | -----------------------------------------------------------------------------: | --------------------------: | ----------------------------------------------------------------: |
| [A100 80GB SXM BF16 TC / HBM2e](https://www.nvidia.com/content/dam/en-zz/Solutions/Data-Center/a100/pdf/nvidia-a100-datasheet-us-nvidia-1758950-r4-web.pdf) |                                                                            312 |                           2 |                                                               156 |
| [H100 SXM BF16 TC / HBM3](https://resources.nvidia.com/en-us-gpu-resources/h100-datasheet-24306)                                                            |                                                                            989 |                        3.35 |                                                               295 |
| [B200 BF16 TC / HBM3e](https://resources.nvidia.com/en-us-dgx-systems/dgx-b200-datasheet)                                                                   |                                                                           2250 |                           8 |                                                               281 |
| [H100 SXM FP8 TC / HBM3](https://resources.nvidia.com/en-us-gpu-resources/h100-datasheet-24306)                                                             |                                                                           1979 |                        3.35 |                                                               592 |
| [B200 FP8 TC / HBM3e](https://resources.nvidia.com/en-us-dgx-systems/dgx-b200-datasheet)                                                                    |                                                                           4500 |                           8 |                                                               562 |
| [B200 FP4 TC / HBM3e](https://resources.nvidia.com/en-us-dgx-systems/dgx-b200-datasheet)                                                                    |                                                                           9000 |                           8 |                                                              1125 |
