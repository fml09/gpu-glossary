---
title: 오버헤드란 무엇인가?
---

오버헤드 지연 시간(Overhead Latency)은 실질적으로 유용한 작업이 전혀 수행되지 않은 채 낭비되는 시간을 의미합니다.

GPU가 최대 속도로 동작하는 [연산 제약](/gpu-glossary/perf/compute-bound)이나 [메모리 제약](/gpu-glossary/perf/memory-bound)으로 인한 [병목](/gpu-glossary/perf/performance-bottleneck) 시간과 달리, 오버헤드로 인한 지연 시간은 GPU가 새로운 작업을 전달받기를 기다리며 유휴 상태로 대기하는 시간을 나타냅니다.

오버헤드는 대개 GPU가 작업을 충분히 빠르게 전달받지 못하게 만드는 CPU 측 병목 현상에서 비롯됩니다. 예를 들어 CUDA API 호출 오버헤드는 커널을 실행할 때마다 약 10 마이크로초(μs) 수준의 시간을 소모합니다. 게다가 PyTorch나 TensorFlow 같은 프레임워크는 어떤 [커널](/gpu-glossary/device-software/kernel)을 실행할지 결정하는 데에도 수십 마이크로초를 소비합니다. 완전히 표준화된 표현은 아니지만, 업계에서는 이를 일반적으로 [호스트 오버헤드(Host Overhead)](https://modal.com/blog/host-overhead-inference-efficiency)라고 부릅니다. 디바이스 측의 여러 [커널](/gpu-glossary/device-software/kernel)을 하나로 묶어 호스트 측에서 단 한 번에 실행할 수 있도록 지원하는 [CUDA 그래프(CUDA Graph)](/gpu-glossary/host-software/cuda-graph)는 이러한 오버헤드를 해소하는 대표적인 해결책입니다. 자세한 내용은 [GTC 2025 발표: 동시성 및 시스템 활용률 극대화를 위한 CUDA 기법(CUDA Techniques to Maximize Concurrency and System Utilization)](https://www.nvidia.com/en-us/on-demand/session/gtc25-s72686/)을 참고하시기 바랍니다.

'메모리 오버헤드(Memory Overhead)' 또는 '통신 오버헤드(Communications Overhead)'는 CPU와 GPU 사이, 혹은 한 GPU와 다른 GPU 사이에서 데이터를 주고받는 과정에서 발생하는 지연 시간을 의미합니다. 하지만 통신 대역폭 자체가 주된 한계 요인인 경우에는 이를 메모리가 여러 머신에 분산되어 있는 일종의 [메모리 제약(Memory Bound)](/gpu-glossary/perf/memory-bound) 상태로 바라보는 것이 분석에 더 유용할 때가 많습니다.
