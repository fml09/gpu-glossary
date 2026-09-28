---
title: 레지스터 파일이란 무엇인가?
---

[스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)의 레지스터 파일(Register File)은 [코어](/gpu-glossary/device-hardware/core)가 연산을 수행하는 과정에서 데이터를 보관하는 주 저장 공간입니다.

![H100 SM 내부 아키텍처. 레지스터 파일이 파란색으로 표시되어 있습니다. NVIDIA의 [H100 백서](https://modal-cdn.com/gpu-glossary/gtc22-whitepaper-hopper.pdf)를 기반으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-gh100-sm.svg)

CPU 레지스터와 마찬가지로, 이 레지스터들은 연산 [코어](/gpu-glossary/device-hardware/core) 처리 속도에 맞춰 동작할 수 있는 매우 빠른 메모리 소자로 구성되며, [L1 데이터 캐시](/gpu-glossary/device-hardware/l1-data-cache)보다 대략 10배 정도 빠릅니다.

레지스터 파일은 32비트 단위 레지스터들로 분할되어 있으며, 32비트 정수, 64비트 부동소수점 수, 16비트 이하 부동소수점 수(또는 이들의 묶음) 등 다양한 데이터 형식에 맞게 동적으로 재할당될 수 있습니다. 이러한 물리 레지스터는 [PTX(Parallel Thread eXecution)](/gpu-glossary/device-software/parallel-thread-execution) 중간 표현에서 정의하는 [가상 레지스터](/gpu-glossary/device-software/registers)를 물리적으로 뒷받침합니다.

[SASS(Streaming Assembler)](/gpu-glossary/device-software/streaming-assembler) 단계에서 [스레드](/gpu-glossary/device-software/thread)에 물리 레지스터를 할당하는 작업은 `ptxas` 같은 컴파일러가 제어하며, 컴파일러는 [스레드 블록](/gpu-glossary/device-software/thread-block) 단위 레지스터 파일 사용량을 최적화합니다. 각 [스레드 블록](/gpu-glossary/device-software/thread-block)이 레지스터 파일을 과도하게 점유하면(이를 흔히 "[레지스터 압박](/gpu-glossary/perf/register-pressure)"이라 부릅니다), 동시에 스케줄링 가능한 [스레드](/gpu-glossary/device-software/thread) 수가 감소하여 [점유율](/gpu-glossary/perf/occupancy)이 낮아집니다. 이는 [지연 시간 은닉](/gpu-glossary/perf/latency-hiding) 기회를 줄여 전반적인 성능에 부정적인 영향을 줄 수 있습니다.
