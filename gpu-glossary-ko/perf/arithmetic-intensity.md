---
title: 연산 강도란 무엇인가?
---

연산 강도(Arithmetic Intensity)는 [커널(Kernel)](/gpu-glossary/device-software/kernel)에서 메모리 연산 대비 산술 연산의 비율을 나타냅니다.

![루프라인 모델(Roofline Model)에서는 가로축에 연산 강도(Operational/Arithmetic Intensity)를 표시합니다. Williams, Waterman, Patterson (2008)의 논문 다이어그램을 참고하여 작성하였습니다.](https://modal-cdn.com/gpu-glossary/light-roofline-model.svg)

높은 연산 강도는 [커널](/gpu-glossary/device-software/kernel)이 메모리에서 1바이트를 읽어올 때마다 많은 수의 산술 연산을 수행함을 의미합니다. 현대 GPU는 [메모리 대역폭](/gpu-glossary/perf/memory-bandwidth) 대비 [연산 대역폭](/gpu-glossary/perf/arithmetic-bandwidth)의 비율이 매우 높기 때문에, 가장 효율적인 커널은 높은 연산 강도를 지닙니다. 이는 메모리 [병목](/gpu-glossary/perf/performance-bottleneck)을 완화하고자 할 때, 작업 부하를 메모리 서브시스템에서 연산 서브시스템으로 이전함으로써 메모리 대역폭을 절약하고 연산 유닛의 부하를 높이는 방식으로 최적화할 수 있음을 의미합니다.

예를 들어 [전역 메모리(Global Memory)](/gpu-glossary/device-software/global-memory) 상의 데이터를 압축하면 전송해야 하는 바이트 수가 줄어들어 메모리 트래픽이 감소하지만, 연산 유닛은 추가적인 압축 해제 연산을 수행해야 합니다. 이전에 메모리 대역폭에 의해 [병목 현상](/gpu-glossary/perf/performance-bottleneck)을 겪고 있었다면, 이러한 방식은 전체 성능을 크게 개선할 수 있습니다. 또한 이동한 바이트 수 대비 FLOPs의 비율이 높아지므로 연산 강도 역시 증가합니다.

또 다른 예로 [역전파 알고리즘(Backpropagation Algorithm)](https://www.nature.com/articles/323533a0)은 순전파 과정에서 장기간 유지되어야 하는 중간값(활성화 값)을 생성하며, 이 값들은 일반적으로 전역 메모리에 저장되었다가 역전파 과정에서 다시 조회됩니다. 특정 상황에서는 이러한 중간값 중 일부만 저장해 두고 나머지는 필요할 때 재계산하는 방식([그래디언트 체크포인팅(Gradient Checkpointing)](https://arxiv.org/abs/1604.06174))이 더 빠른데, 이 기법 또한 연산 강도를 높이는 방법입니다.

알고리즘마다 본질적인 연산 복잡도와 메모리 복잡도가 서로 다르기 때문에, 데이터 크기에 따른 연산 강도의 증가 추세(스케일링)도 각기 다릅니다. 연산 복잡도가 O(1)이고 메모리 복잡도가 O(N)인 알고리즘은 O(1/N)의 연산 강도 스케일링을 가지는 반면, 연산 복잡도가 O(N)이고 메모리 복잡도가 O(1)인 알고리즘은 O(N)의 연산 강도 스케일링을 보입니다.

| **커널(Kernel)**          |    **FLOPs** | **이동한 바이트 수(Bytes Moved)** | **연산 강도(Arithmetic Intensity)** | **연산 강도 스케일링(Arithmetic Intensity Scaling)** |
| :------------------------ | -----------: | --------------------------------: | -----------------------------------: | ----------------------------------------------------: |
| SAXPY y = ax + y          |           2N |                               12N |                                  1/6 |                                                  O(1) |
| 단정밀도 실수 FFT         | 5/2 N log(N) |                               16N |                          5/32 log(N) |                                             O(log(N)) |
| SGEMM C = A @ B + C       |         2N^3 |                             16N^2 |                                  N/8 |                                                  O(N) |

특히 행렬 곱셈은 연산 강도가 선형적으로 증가합니다. 즉, O(N)의 연산 강도 스케일링을 가집니다. 연산 복잡도는 O(N^3)인 반면 메모리 복잡도는 O(N^2)에 불과하기 때문입니다. 이러한 유리한 스케일링 특성 덕분에 행렬 곱셈 애플리케이션을 높은 연산 강도에 최적화된 최신 하드웨어에 효율적으로 매핑할 수 있습니다([루프라인 모델 문서](/gpu-glossary/perf/roofline-model) 참조). 이는 지난 수십 년 동안 신경망과 같은 행렬 곱셈 기반 머신러닝 알고리즘이 비약적인 성공을 거둘 수 있었던 핵심 요인 중 하나입니다.

트랜스포머(Transformer) 신경망에 사용되는 바흐다나우 어텐션(Bahdanau Attention)의 연산 강도에 관한 자세한 분석은 Zadouri, Strauss, Dao의 [논문](https://arxiv.org/abs/2505.21487)을 참조하시기 바랍니다.

연산 작업이 [연산 제약(Compute Bound)](/gpu-glossary/perf/compute-bound) 상태(즉, [루프라인 모델](/gpu-glossary/perf/roofline-model)의 변곡점을 넘어서는 상태)에 도달하기 위해 요구되는 최소 연산 강도는 시스템의 고유한 고정 파라미터이므로 한 번만 계산해 두면 됩니다. 최근 NVIDIA 데이터센터 GPU의 변곡점 연산 강도는 아래 표에 정리되어 있습니다. 암페어(Ampere), 호퍼(Hopper), 블랙웰(Blackwell) [스트리밍 다중처리기 아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture)로 발전함에 따라 최대 변곡점의 수치가 점차 증가하고 있음을 확인할 수 있습니다.

| **시스템 (연산 / 메모리)**                                                                                                                                   | **[연산 대역폭](/gpu-glossary/perf/arithmetic-bandwidth) (TFLOPs/s)** | **[메모리 대역폭](/gpu-glossary/perf/memory-bandwidth) (TB/s)** | **[변곡점(Ridge Point)](/gpu-glossary/perf/roofline-model) (FLOPs/byte)** |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------- | -----------------------------------------------------------------------------: | -----------------------------------------------------------------: | ----------------------------------------------------------------: |
| [A100 80GB SXM BF16 TC / HBM2e](https://www.nvidia.com/content/dam/en-zz/Solutions/Data-Center/a100/pdf/nvidia-a100-datasheet-us-nvidia-1758950-r4-web.pdf) |                                                                            312 |                                                                  2 |                                                               156 |
| [H100 SXM BF16 TC / HBM3](https://resources.nvidia.com/en-us-gpu-resources/h100-datasheet-24306)                                                            |                                                                            989 |                                                               3.35 |                                                               295 |
| [B200 BF16 TC / HBM3e](https://resources.nvidia.com/en-us-dgx-systems/dgx-b200-datasheet)                                                                   |                                                                           2250 |                                                                  8 |                                                               281 |
| [H100 SXM FP8 TC / HBM3](https://resources.nvidia.com/en-us-gpu-resources/h100-datasheet-24306)                                                             |                                                                           1979 |                                                               3.35 |                                                               592 |
| [B200 FP8 TC / HBM3e](https://resources.nvidia.com/en-us-dgx-systems/dgx-b200-datasheet)                                                                    |                                                                           4500 |                                                                  8 |                                                               562 |
| [B200 FP4 TC / HBM3e](https://resources.nvidia.com/en-us-dgx-systems/dgx-b200-datasheet)                                                                    |                                                                           9000 |                                                                  8 |                                                              1125 |
