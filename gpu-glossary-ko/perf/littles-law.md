---
title: 리틀의 법칙이란 무엇인가?
---

리틀의 법칙(Little's Law)은 처리량을 바탕으로 [지연 시간을 온전히 은닉](/gpu-glossary/perf/latency-hiding)하는 데 필요한 동시성(Concurrency)의 크기를 규정하는 법칙입니다.

```
concurrency (ops) = latency (s) * throughput (ops/s)
```

리틀의 법칙은 [Lazowska 등의 고전적인 정량적 시스템 분석 교과서](https://homes.cs.washington.edu/~lazowska/qsp/Images/Chap_03.pdf)에서 성능 분석의 "가장 중요한 기본 법칙"으로 소개됩니다.

리틀의 법칙은 GPU가 [워프 스케줄러](/gpu-glossary/device-hardware/warp-scheduler)를 통한 [워프](/gpu-glossary/device-software/warp) 전환(CPU의 [동시 다중 스레딩(Simultaneous Multithreading, SMT)](https://en.wikipedia.org/wiki/Simultaneous_multithreading)과 유사한 세립도 스레드 수준 병렬성)으로 [지연 시간을 은닉](/gpu-glossary/perf/latency-hiding)할 때, 실행 대기 상태(in flight)로 유지해야 하는 명령어의 수를 결정합니다.

만약 GPU의 최대 처리율이 사이클당 1개의 명령어이고 메모리 접근 지연 시간이 400사이클이라면, 프로그램의 모든 [활성 워프](/gpu-glossary/perf/warp-execution-state)에 걸쳐 동시에 400개의 메모리 연산이 진행되어야 합니다. 만약 처리율이 사이클당 10개의 명령어로 증가한다면, 이러한 처리량 향상을 온전히 활용하기 위해 동시에 4,000개의 메모리 연산이 필요합니다. 자세한 내용은 [지연 시간 은닉](/gpu-glossary/perf/latency-hiding) 문서를 참조하시기 바랍니다.

리틀의 법칙을 적용한 흥미로운 사례로, Vasily Volkov의 [지연 시간 은닉에 관한 박사 학위 논문](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2016/EECS-2016-143.pdf) 4.3절에 제시된 관찰을 들 수 있습니다. 순수한 메모리 접근 지연 시간을 숨기는 데 필요한 워프 수는 순수한 산술 연산 지연 시간을 숨기는 데 필요한 워프 수보다 크게 많지 않았습니다(그의 실험에서는 30개 대 24개). 직관적으로는 지연 시간이 훨씬 더 긴 메모리 접근이 훨씬 더 높은 동시성을 요구할 것처럼 보입니다. 그러나 동시성은 지연 시간뿐만 아니라 처리율(Throughput)에 의해서도 결정됩니다. [메모리 대역폭](/gpu-glossary/perf/memory-bandwidth)은 [연산 대역폭](/gpu-glossary/perf/arithmetic-bandwidth)보다 훨씬 낮기 때문에, 결과적으로 필요한 동시성은 두 연산이 거의 엇비슷해집니다. 이는 산술 연산과 메모리 연산이 혼합되어 실행되는 [지연 시간 은닉](/gpu-glossary/perf/latency-hiding) 기반 시스템에서 매우 유용한 하드웨어적 균형을 이룹니다.
