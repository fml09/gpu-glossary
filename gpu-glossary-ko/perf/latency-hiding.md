---
title: 지연 시간 은닉이란 무엇인가?
---

지연 시간 은닉(Latency Hiding)은 [수많은 작업을 동시에 병렬로 실행](/gpu-glossary/perf/littles-law)함으로써 지연 시간이 긴 연산의 대기 시간을 가리는 최적화 전략입니다.

성능이 뛰어난 GPU 프로그램은 수많은 [스레드(Thread)](/gpu-glossary/device-software/thread)의 실행을 교차 배치(인터리빙)하여 긴 지연 시간을 감춥니다. 이를 통해 긴 명령어 지연 시간에도 불구하고 프로그램의 전체 처리량을 높은 수준으로 유지할 수 있습니다. 한 [워프가 스톨](/gpu-glossary/perf/warp-execution-state)되어 느린 메모리 연산 결과를 기다리는 동안, GPU는 즉시 실행 가능한 다른 [발행 가능 워프](/gpu-glossary/perf/warp-execution-state)로 전환하여 명령어를 실행합니다.

이러한 메커니즘을 통해 모든 실행 유닛을 동시에 쉬지 않고 가동할 수 있습니다. 예를 들어 한 [워프](/gpu-glossary/device-software/warp)가 [텐서 코어(Tensor Core)](/gpu-glossary/device-hardware/tensor-core)를 활용해 행렬 곱셈을 수행하는 동안, 다른 워프는 [CUDA 코어](/gpu-glossary/device-hardware/cuda-core)에서 산술 연산(예: [행렬 곱셈 피연산자의 양자화 또는 역양자화](https://arxiv.org/abs/2408.11743))을 실행할 수 있으며, 또 다른 세 번째 워프는 [로드/스토어 유닛(LSU)](/gpu-glossary/device-hardware/load-store-unit)을 통해 데이터를 가져올 수 있습니다.

구체적인 이해를 돕기 위해 다음과 같은 단순한 [스트리밍 어셈블러(SASS)](/gpu-glossary/device-software/streaming-assembler) 명령어 시퀀스를 살펴보겠습니다.

```nasm
LDG.E.SYS R1, [R0]        // memory load, 400 cycles
IMUL R2, R1, 0xBEEF       // integer multiply, 6 cycles
IADD R4, R2, 0xAFFE       // integer add, 4 cycles
IMUL R6, R4, 0x1337       // integer multiply, 6 cycles
```

단일 스레드에서 순차적으로 실행할 경우 이 시퀀스를 완료하는 데 416사이클이 소요됩니다. 하지만 동시 실행을 통해 이러한 지연 시간을 은닉할 수 있습니다. 매 사이클마다 한 개의 명령어를 발행할 수 있다고 가정하면, [리틀의 법칙(Little's Law)](/gpu-glossary/perf/littles-law)에 따라 416개의 [스레드](/gpu-glossary/device-software/thread)를 동시에 실행함으로써 평균적으로 매 사이클마다 이 시퀀스를 하나씩 완료할 수 있습니다. 즉, `R6` 데이터를 사용하는 후속 작업자 관점에서 메모리 지연 시간이 완전히 은닉됩니다.

명령어 발행 단위는 개별 [스레드](/gpu-glossary/device-software/thread)가 아니라 [워프](/gpu-glossary/device-software/warp)라는 점에 유의해야 합니다. 각 [워프](/gpu-glossary/device-software/warp)는 32개의 [스레드](/gpu-glossary/device-software/thread)로 이루어지므로, 이 코드 조각의 지연 시간을 숨기려면 416 ÷ 32 = 13개의 [워프](/gpu-glossary/device-software/warp)가 필요합니다. 지연 시간 은닉이 성공적으로 이루어지면 GPU 스케줄링 시스템은 이만큼의 워프를 실행 대기 상태(in-flight)로 유지하며, 한 워프가 스톨될 때마다 워프 간 전환을 수행하여 느린 연산이 완료되기를 기다리는 동안에도 실행 유닛이 결코 유휴 상태에 머무르지 않도록 합니다.

[텐서 코어](/gpu-glossary/device-hardware/tensor-core) 도입 이전 GPU에서의 지연 시간 은닉에 관한 심도 있는 분석은 [Vasily Volkov의 박사 학위 논문](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2016/EECS-2016-143.pdf)을 참고하시기 바랍니다.
