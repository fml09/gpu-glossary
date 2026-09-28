---
title: 점유율이란 무엇인가?
---

점유율(Occupancy)은 디바이스에서 수용 가능한 최대 [활성 워프(Active Warp)](/gpu-glossary/perf/warp-execution-state) 수 대비 실제 상주하는 [활성 워프](/gpu-glossary/perf/warp-execution-state) 수의 비율을 의미합니다.

![4개의 클록 사이클마다 사이클당 4개의 워프 슬롯이 있어 총 16(4×4)개의 워프 슬롯이 존재하며, 그중 15개 슬롯에 활성 워프가 있으므로 점유율은 약 94%입니다. GTC 2025의 [*연산 및 명령어 처리율 극대화를 위한 CUDA 기법*](https://www.nvidia.com/en-us/on-demand/session/gtc25-s72685/) 발표 내용을 참고하여 작성하였습니다.](https://modal-cdn.com/gpu-glossary/light-cycles.svg)

점유율 측정 방식에는 두 가지 유형이 있습니다.

- *이론적 점유율(Theoretical Occupancy)*: 커널 실행 설정 및 디바이스 사양에 의해 결정되는 점유율의 이론적 상한선을 나타냅니다.
- *달성 점유율(Achieved Occupancy)*: [커널](/gpu-glossary/device-software/kernel) 실행 도중, 즉 [활성 사이클](/gpu-glossary/perf/active-cycle) 동안 실제로 달성된 워프 점유율을 측정합니다.

[CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)에서 하나의 [스레드 블록(Thread Block)](/gpu-glossary/device-software/thread-block)에 속한 모든 [스레드(Thread)](/gpu-glossary/device-software/thread)는 동일한 [스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)에 스케줄링됩니다. 각 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)은 [공유 메모리(Shared Memory)](/gpu-glossary/device-software/shared-memory) 공간이나 레지스터처럼 스레드 블록 간에 분할 할당되어야 하는 한정된 하드웨어 리소스를 가지므로, 단일 SM에 동시에 상주할 수 있는 스레드 블록의 개수가 제한됩니다.

구체적인 예시로 다음과 같은 하드웨어 사양을 갖는 NVIDIA H100 GPU를 살펴보겠습니다.

```
Maximum warps/SM: 64
Maximum blocks/SM: 32
(32 bit) Registers: 65536
Shared memory (smem): 228 KB
```

스레드 블록당 32개의 [스레드](/gpu-glossary/device-software/thread), 스레드당 8개의 [레지스터](/gpu-glossary/device-software/registers), 블록당 12 KB의 [공유 메모리](/gpu-glossary/device-software/shared-memory)를 사용하는 [커널](/gpu-glossary/device-software/kernel)의 경우, 공유 메모리에 의해 상주 가능한 블록 수가 제한됩니다.

```
64 > 1   = warps/block = 32 threads/block ÷ 32 threads/warp
32 < 256 = blocks/register-file = 65,536 registers/register-file ÷ (32 threads/block × 8 registers/thread)
32       = blocks/SM
19       = blocks/smem = 228 KB/smem ÷ 12 KB/block
```

[레지스터 파일](/gpu-glossary/device-hardware/register-file)의 크기는 동시에 256개의 [스레드 블록](/gpu-glossary/device-software/thread-block)을 수용할 수 있을 만큼 넉넉하지만, [공유 메모리](/gpu-glossary/device-software/shared-memory) 용량 제한으로 인해 각 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)당 19개의 스레드 블록만 실행할 수 있습니다. 이는 19개의 [워프](/gpu-glossary/device-software/warp)에 해당합니다. 이는 [레지스터](/gpu-glossary/device-software/registers)에 저장되는 프로그램의 중간 계산값 크기가 [공유 메모리](/gpu-glossary/device-software/shared-memory)에 유지되어야 하는 [작업 집합(Working Set)](https://en.wikipedia.org/wiki/Working_set)의 크기보다 훨씬 작은 일반적인 경우에 흔히 발생합니다.

명령어 지연 시간을 [은닉](/gpu-glossary/perf/latency-hiding)할 만큼 충분한 [발행 가능 워프](/gpu-glossary/perf/warp-execution-state)가 존재하지 않을 때 낮은 점유율은 심각한 성능 저하를 야기하며, 이는 낮은 명령어 [발행 효율](/gpu-glossary/perf/issue-efficiency)과 [파이프라인 활용률 저하](/gpu-glossary/perf/pipe-utilization)로 나타납니다. 하지만 [지연 시간 은닉](/gpu-glossary/perf/latency-hiding)을 달성할 수 있는 충분한 수준의 점유율을 확보한 뒤에는 점유율을 무리하게 더 높이더라도 오히려 성능이 저하될 수 있습니다. 높은 점유율을 유지하려면 [스레드](/gpu-glossary/device-software/thread)당 할당되는 리소스가 줄어들어 [레지스터 압박(Register Pressure)](/gpu-glossary/perf/register-pressure) 병목이 발생하거나, 현대 GPU 아키텍처가 성능을 극대화하도록 설계된 기반인 [연산 강도](/gpu-glossary/perf/arithmetic-intensity)가 저하될 수 있기 때문입니다.

더 넓은 관점에서 점유율은 GPU가 동시에 처리할 수 있는 최대 병렬 작업량 중 어느 정도의 비율을 다루고 있는지를 측정하는 지표일 뿐이며, 대다수 커널에서 점유율 자체가 최종적인 최적화 목표는 아닙니다. 오히려 핵심 목표는 작업이 [연산 제약](/gpu-glossary/perf/compute-bound) 상태일 때 연산 리소스의 [활용률](/gpu-glossary/perf/pipe-utilization)을 극대화하거나, [메모리 제약](/gpu-glossary/perf/memory-bound) 상태일 때 메모리 대역폭의 활용률을 극대화하는 것입니다.

실제로 호퍼(Hopper) 및 블랙웰(Blackwell) [아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture) GPU에서 최고 성능을 내는 고성능 GEMM 커널들은 불과 한 자릿수의 낮은 점유율로 동작하는 경우가 흔합니다. [텐서 코어](/gpu-glossary/device-hardware/tensor-core)를 완전히 포화시키는 데에는 많은 수의 [워프](/gpu-glossary/device-software/warp)가 필요하지 않기 때문입니다.
