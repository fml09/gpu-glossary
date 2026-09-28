---
title: 루프라인 모델이란 무엇인가?
---

루프라인 모델(Roofline Model)은 프로그램의 성능이 [메모리 대역폭](/gpu-glossary/perf/memory-bandwidth)과 [연산 대역폭](/gpu-glossary/perf/arithmetic-bandwidth) 중 어느 쪽에 의해 제약받는지를 신속하게 판별하기 위해 고안된 직관적인 시각적 성능 분석 모델입니다.

![변곡점(Ridge point) 왼쪽에 위치한 [커널](/gpu-glossary/device-software/kernel)은 [메모리 서브시스템 대역폭에 의해 제한](/gpu-glossary/perf/memory-bound)되며, 변곡점 오른쪽에 위치한 커널은 [산술 연산 서브시스템 대역폭에 의해 제한](/gpu-glossary/perf/compute-bound)됩니다. 루프라인 모델을 최초로 제안한 Williams, Waterman, Patterson (2008)의 논문 다이어그램을 참고하여 작성하였습니다.](https://modal-cdn.com/gpu-glossary/light-roofline-model.svg)

루프라인 모델에서는 하드웨어 사양으로부터 도출된 두 개의 '지붕(Roof)'이 달성 가능한 성능의 상한선(Ceiling)을 형성합니다.

- '연산 한계선(Compute Roof)': 대상 하드웨어([CUDA 코어](/gpu-glossary/device-hardware/cuda-core) 또는 [텐서 코어](/gpu-glossary/device-hardware/tensor-core))의 [최대 처리율(Peak Rate)](/gpu-glossary/perf/peak-rate), 즉 [연산 대역폭](/gpu-glossary/perf/arithmetic-bandwidth)을 나타냅니다.
- '메모리 한계선(Memory Roof)': 대상 하드웨어의 피크 메모리 처리량, 즉 [메모리 대역폭](/gpu-glossary/perf/memory-bandwidth)을 나타냅니다.

이 모델은 가로축(X축)에 [연산 강도(Arithmetic Intensity)](/gpu-glossary/perf/arithmetic-intensity)(바이트당 연산 수, ops/byte), 세로축(Y축)에 달성 성능(초당 연산 수, ops/s)을 배치한 2차원 평면으로 시각화됩니다. '연산 한계선'은 높이가 [연산 대역폭](/gpu-glossary/perf/arithmetic-bandwidth)과 같은 수평선 형태를 띱니다. '메모리 한계선'은 기울기가 [메모리 대역폭](/gpu-glossary/perf/memory-bandwidth)과 같은 경사선 형태를 가집니다. 기울기는 밑변 변화량 대비 높이 변화량이므로, 이 선의 단위는 바이트/초(초당 연산 수 ÷ 바이트당 연산 수)가 됩니다.

특정 [커널](/gpu-glossary/device-software/kernel)의 X축 좌표를 확인하면, 해당 커널이 근본적으로 [연산 제약(Compute Bound)](/gpu-glossary/perf/compute-bound) 상태인지(수평선 아래에 위치) 아니면 [메모리 제약(Memory Bound)](/gpu-glossary/perf/memory-bound) 상태인지(경사선 아래에 위치) 즉시 파악할 수 있습니다. [오버헤드(Overhead)](/gpu-glossary/perf/overhead)의 영향으로 인해 실제 커널이 이 두 한계선에 완전히 맞닿는 경우는 드뭅니다.

경사선과 수평선이 만나는 경계 지점을 '변곡점(Ridge Point)'이라고 부릅니다. 변곡점의 X축 좌표는 메모리 [병목](/gpu-glossary/perf/performance-bottleneck)에서 벗어나는 데 필요한 최소 [연산 강도](/gpu-glossary/perf/arithmetic-intensity)를 의미합니다. 변곡점이 왼쪽으로 치우쳐 있을수록 하드웨어의 최대 성능을 끌어내기가 더 수월하지만, 연산 성능에 비해 메모리 대역폭의 발전 속도가 상대적으로 더디기 때문에 역사적으로 시스템의 변곡점은 점차 오른쪽으로 이동해 왔습니다.

연산 한계선과 메모리 한계선은 서브시스템당 한 번만 계산하면 됩니다. 다만 시스템 전체뿐만 아니라 서브시스템에 따라서도 달라집니다. 가령 [텐서 코어](/gpu-glossary/device-hardware/tensor-core)는 [CUDA 코어](/gpu-glossary/device-hardware/cuda-core)보다 훨씬 더 높은 FLOPS를 제공합니다.

NVIDIA의 커널 성능 프로파일링 도구인 Nsight Compute는 프로파일링 대상 [커널](/gpu-glossary/device-software/kernel)에 대해 루프라인 분석을 자동으로 수행해 줍니다.

루프라인 모델은 얼핏 단순해 보이지만 깊이 있는 통찰을 담고 있습니다. 예를 들어 다이어그램 어디에도 시스템 지연 시간(latency)은 직접 표시되지 않으며 오직 대역폭과 처리량만 나타납니다. 모델이 명확한 관점과 철학을 담고 있기 때문에 단순하게 표현될 수 있는 것이며, 이러한 철학과 배경 논리를 이해하는 것이 루프라인을 올바르게 활용하고 그 진가를 발휘하는 핵심 열쇠입니다.

루프라인 모델은 Samuel Williams, Andrew Waterman, David Patterson이 [2008년 논문](https://people.eecs.berkeley.edu/~kubitron/cs252/handouts/papers/RooflineVyNoYellow.pdf)에서 처음 발표하였습니다. 저자들은 이전부터 이어져 온 여러 하드웨어 스케일링 트렌드가 향후 시스템 아키텍처를 근본적으로 바꿀 것임을 내다보고 이 모델을 고안하였습니다.

첫째, Patterson이 2004년 유명 논문에서 별도로 지적했듯이 [지연 시간 개선은 대역폭 개선에 뒤처집니다(Latency lags bandwidth)](https://dl.acm.org/doi/pdf/10.1145/1022594.1022596). 구체적으로 연산, 메모리, 스토리지 등 다양한 서브시스템 전반에서 역사적으로 지연 시간이 선형적으로 단축되는 동안 대역폭은 2차 함수적으로 비약적인 향상을 이뤄냈습니다. 이는 미래의 컴퓨터 시스템이 GPU처럼 철저히 처리량 중심(throughput-oriented)으로 진화할 것임을 시사했습니다.

둘째, 오랫동안 관찰되었듯이 프로세서 코어와 같은 연산 서브시스템의 성능 발전 속도가 [캐시](/gpu-glossary/device-hardware/l1-data-cache)나 [DRAM](/gpu-glossary/device-hardware/gpu-ram) 같은 메모리 서브시스템의 발전 속도를 훨씬 앞질렀습니다. 이는 1994년 Wulf와 McKee에 의해 [메모리 장벽(Memory Wall)](https://www.eecs.ucf.edu/~lboloni/Teaching/EEL5708_2006/slides/wulf94.pdf)이라는 용어로 대중화되었습니다.

마지막으로 2000년대 초반에는 트랜지스터의 고정 누설 전류로 인한 전력 소모 및 발열 문제로 인해 동일 전력에서 클록 속도를 높이는 [데너드 스케일링(Dennard scaling)](https://en.wikipedia.org/wiki/Dennard_scaling)이 한계에 직면했습니다. 과거에는 클록 속도의 지속적인 향상 덕분에 CPU와 같은 범용 지연 시간 중심 시스템이 특수 목적 하드웨어보다 우위를 점할 수 있었습니다. 그러나 클록 속도 향상의 정체에도 불구하고 칩당 트랜지스터 수가 증가하는 [무어의 법칙(Moore's Law)](https://en.wikipedia.org/wiki/Moore%27s_law)은 계속 이어졌습니다. 트랜지스터는 풍부하지만 가용 전력이 제한적인 상황에서 아키텍처가 선택한 해결책은 하드웨어 특화(specialization), 즉 컴퓨터를 서로 다른 특화된 작업을 수행하는 전용 컴포넌트들로 분할하는 것이었습니다. 대표적인 사례로 구글의 [Pixel Visual Core](https://blog.google/products/pixel/pixel-visual-core-image-processing-and-machine-learning-pixel-2/) 이미지 코프로세서를 들 수 있으며, 이는 Hennessy와 Patterson의 명저 [컴퓨터 구조(Computer Architecture: A Quantitative Approach)](https://archive.org/details/computerarchitectureaquantitativeapproach6thedition/page/n13/mode/2up) 제6판 7장에 상세히 설명되어 있습니다.

이러한 흐름들을 종합해 볼 때 미래의 시스템은 처리량 중심으로 발전할 것이며, 여러 대역폭 중에서도 [메모리 서브시스템 대역폭](/gpu-glossary/perf/memory-bandwidth)이 가장 결정적인 [성능 병목](/gpu-glossary/perf/performance-bottleneck)으로 자리 잡을 것이라는 저자들의 예측은 정확히 들어맞았습니다. 따라서 이러한 최신 시스템에서 최대 성능을 발휘하려면 해당 하드웨어의 특화 연산에 대해 높은 연산 집약도를 확보해야 합니다. GPU의 경우에는 [텐서 코어](/gpu-glossary/device-hardware/tensor-core)를 활용한 매우 거대한 행렬 곱셈을 통해 충분히 높은 [연산 강도](/gpu-glossary/perf/arithmetic-intensity)를 달성하는 것이 핵심입니다.
