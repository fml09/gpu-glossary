---
title: 파이프라인 활용률이란 무엇인가?
---

파이프라인 활용률(Pipe Utilization)은 [커널(Kernel)](/gpu-glossary/device-software/kernel)이 각 [스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor) 내부의 실행 리소스를 얼마나 효과적으로 사용하는지 측정하는 지표입니다.

각 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)은 다양한 명령어 유형에 최적화된 독립적인 실행 파이프라인을 다수 포함하고 있습니다. 일반 부동소수점 산술 연산을 위한 [CUDA 코어](/gpu-glossary/device-hardware/cuda-core), 텐서 축약(행렬 곱셈)을 위한 [텐서 코어](/gpu-glossary/device-hardware/tensor-core), 메모리 접근을 위한 [로드/스토어 유닛(LSU)](/gpu-glossary/device-hardware/load-store-unit), 분기 처리를 위한 제어 흐름 유닛 등이 이에 해당합니다. 파이프라인 활용률은 특정 파이프가 적어도 하나 이상의 [워프](/gpu-glossary/device-software/warp)를 활발히 실행하고 있을 때, 해당 파이프라인의 [최대 처리율(Peak Rate)](/gpu-glossary/perf/peak-rate) 대비 몇 퍼센트를 달성하고 있는지를 모든 활성 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)에 걸쳐 평균을 내어 보여줍니다.

파이프라인 활용률 수준에서 세부적인 성능 디버깅을 시작하기 전에, GPU 프로그래머는 먼저 전체 [GPU 커널 활용률](https://modal.com/blog/gpu-utilization-guide)과 [SM 활용률](/gpu-glossary/perf/streaming-multiprocessor-utilization)을 우선적으로 확인해야 합니다.

파이프라인 활용률은 [Nsight Compute](https://developer.nvidia.com/nsight-compute)(`ncu`)의 `sm__inst_executed_pipe_*.avg.pct_of_peak_sustained_active` 메트릭을 통해 확인할 수 있습니다. 여기서 와일드카드(`*`) 위치에는 [`fma`](/gpu-glossary/device-hardware/cuda-core), [`tensor`](/gpu-glossary/device-hardware/tensor-core), [`lsu`](/gpu-glossary/device-hardware/load-store-unit), `adu`(주소) 등 특정 파이프라인 명칭이 들어갑니다.
