---
title: 워프란 무엇인가?
---

워프(Warp)는 함께 스케줄링되어 병렬로 실행되는 [스레드](/gpu-glossary/device-software/thread) 그룹입니다. 워프 내의 모든 [스레드](/gpu-glossary/device-software/thread)는 단일 [스트리밍 다중처리기(SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)에 스케줄링됩니다. 단일 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)은 통상 여러 워프를 실행하며, 최소한 동일한 [협력형 스레드 배열(CTA)](/gpu-glossary/device-software/cooperative-thread-array)(즉 [스레드 블록](/gpu-glossary/device-software/thread-block))에 속한 모든 워프를 함께 실행합니다.

워프는 GPU에서 실행되는 전형적인 실행 단위입니다. 정상적인 실행 상태에서는 단일 워프에 속한 모든 [스레드](/gpu-glossary/device-software/thread)가 동일한 명령어를 병렬로 실행하며, 이를 이른바 '단일 명령어 다중 스레드(SIMT, Single-Instruction, Multiple Thread)' 모델이라고 합니다. 워프 내의 [스레드](/gpu-glossary/device-software/thread)들이 서로 갈라져 서로 다른 명령어를 실행하는 상황을 [워프 분기(Warp Divergence)](/gpu-glossary/perf/warp-divergence)라고 부르며, 이 경우 일반적으로 성능이 급격히 저하됩니다.

워프 크기는 엄밀히 말하면 기기 의존적인 상수이지만, 실제로는(그리고 이 용어집의 다른 모든 부분에서도) 32개 스레드를 의미합니다.

워프에 명령어가 발행되면 일반적으로 단일 클럭 사이클 내에 결과가 바로 나오지 않으므로, 이전 연산 결과에 의존하는 후속 명령어를 곧바로 발행할 수 없습니다. 이는 칩 외부로 접근해야 하는 [전역 메모리](/gpu-glossary/device-software/global-memory)에서의 데이터 인출([GPU RAM](/gpu-glossary/device-hardware/gpu-ram) 접근)에서 가장 두드러지게 나타나지만, 특정 산술 연산 명령어에서도 마찬가지로 지연이 발생합니다(각 명령어별 클럭 사이클 소요 시간 표는 [CUDA C++ 모범 사례 가이드](https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/index.html#arithmetic-instructions) 참조).

필요한 피연산자가 준비되지 않아 다음 명령어가 지연되고 있는 워프의 상태를 [스톨(stalled)](/gpu-glossary/perf/warp-execution-state) 상태라고 합니다.

단일 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)에 여러 워프가 스케줄링되어 있는 경우, [워프 스케줄러](/gpu-glossary/device-hardware/warp-scheduler)는 특정 명령어의 실행 결과가 반환되기를 기다리는 대신 곧바로 실행 가능한 다른 워프를 선택하여 실행합니다. 이러한 [지연 시간 숨기기(Latency Hiding)](/gpu-glossary/perf/latency-hiding) 메커니즘을 통해 GPU는 높은 처리량을 달성하고 프로그램 실행 중 모든 코어가 쉬지 않고 일하도록 보장합니다. 따라서 각 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)에 스케줄링되는 워프의 수를 극대화하여 SM이 실행할 수 있는 [적격(eligible)](/gpu-glossary/perf/warp-execution-state) 워프를 항상 확보해 두는 것이 성능 최적화에 유리합니다. 워프에 명령어가 발행된 사이클의 비율을 [발행 효율(Issue Efficiency)](/gpu-glossary/perf/issue-efficiency)이라고 하며, 워프 스케줄링의 동시성 수준을 [점유율(Occupancy)](/gpu-glossary/perf/occupancy)이라고 합니다.

워프는 사실 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)의 [스레드 계층 구조](/gpu-glossary/device-software/thread-hierarchy)에 공식적으로 포함된 추상화 요소가 아닙니다. 그 대신 해당 모델을 NVIDIA GPU 상에 구현하는 과정에서 비롯된 구현 세부 사항(implementation detail)에 해당합니다. 그런 점에서 워프는 CPU의 [캐시 라인(cache line)](https://www.nic.uoregon.edu/~khuck/ts/acumem-report/manual_html/ch03s02.html)과 다소 비슷합니다. 프로그래머가 직접 제어하지 않으며 프로그램의 논리적 정확성을 보장하기 위해 반드시 고려할 필요는 없지만, [최고의 성능](/gpu-glossary/perf)을 달성하기 위해서는 반드시 이해해야 하는 하드웨어 특성이기 때문입니다.

[Lindholm et al., 2008](https://www.cs.cmu.edu/afs/cs/academic/class/15869-f11/www/readings/lindholm08_tesla.pdf)에 따르면, 워프(Warp)라는 이름은 "인류 최초의 병렬 실(thread) 기술"인 방직(weaving)의 날실(warp)에서 유래하였습니다. 다른 GPU 프로그래밍 모델에서 워프에 해당하는 개념으로는 WebGPU의 [서브그룹(subgroup)](https://github.com/gpuweb/gpuweb/pull/4368), DirectX의 [웨이브(wave)](https://microsoft.github.io/DirectX-Specs/d3d/HLSL_SM_6_6_WaveSize.html), Apple Metal의 [SIMD 그룹(simdgroup)](https://developer.apple.com/documentation/metal/compute_passes/creating_threads_and_threadgroups#2928931) 등이 있습니다.
