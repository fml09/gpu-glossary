---
title: 워프 스케줄러란 무엇인가?
---

[스트리밍 다중처리기(SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)의 워프 스케줄러(Warp Scheduler)는 매 클록 사이클마다 어떤 [스레드](/gpu-glossary/device-software/thread) 그룹을 실행할지 결정합니다.

![H100 SM 내부 아키텍처. 워프 스케줄러와 디스패치 유닛(Dispatch Unit)이 주황색으로 표시되어 있습니다. NVIDIA의 [H100 백서](https://modal-cdn.com/gpu-glossary/gtc22-whitepaper-hopper.pdf)를 기반으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-gh100-sm.svg)

[워프(Warp)](/gpu-glossary/device-software/warp)라고 부르는 이 스레드 그룹은 매 클록 사이클(대략 1나노초) 단위로 즉시 전환됩니다. 이는 CPU의 동시 다중 스레딩(SMT, 흔히 "하이퍼스레딩"이라 부르는 기술)에서 볼 수 있는 정밀한 스레드 수준 병렬성과 유사하지만 훨씬 더 큰 규모로 이루어집니다. 명령어 피연산자가 준비되는 즉시 방대한 수의 동시 작업 사이를 신속하게 전환하는 워프 스케줄러 역량은 GPU가 뛰어난 [지연 시간 은닉](/gpu-glossary/perf/latency-hiding) 성능을 달성하는 핵심 원동력입니다.

CPU에서 완전한 스레드 문맥 교환(context switch)을 수행하려면 한 스레드의 문맥을 저장하고 다른 스레드의 문맥을 복원해야 하므로 수백에서 수천 클록 사이클(나노초 단위라기보다는 마이크로초 단위에 가깝습니다)이 걸립니다. 또한 CPU 문맥 교환은 캐시 지역성을 떨어뜨리고 캐시 미스율을 높여 전반적인 성능을 저하시킵니다([Mogul and Borg, 1991](https://www.researchgate.net/publication/220938995_The_Effect_of_Context_Switches_on_Cache_Performance) 참조).

반면 GPU에서는 각 [스레드](/gpu-glossary/device-software/thread)가 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)의 [레지스터 파일](/gpu-glossary/device-hardware/register-file)로부터 전용 [레지스터](/gpu-glossary/device-software/registers)를 독립적으로 할당받기 때문에, GPU 문맥 교환은 문맥을 저장하거나 복원하기 위한 별도의 데이터 이동을 전혀 수반하지 않습니다.

또한 GPU [L1 캐시](/gpu-glossary/device-hardware/l1-data-cache)는 프로그래머가 직접 제어할 수 있으며, 동일한 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)에 함께 스케줄링된 [워프](/gpu-glossary/device-software/warp) 사이에서 [공유](/gpu-glossary/device-software/shared-memory)되므로([협력형 스레드 배열](/gpu-glossary/device-software/cooperative-thread-array) 참조), GPU 문맥 교환이 캐시 적중률에 미치는 악영향이 훨씬 적습니다. GPU 내부의 프로그래머 관리형 캐시와 하드웨어 관리형 캐시 간 상호작용에 관한 세부 사항은 [CUDA C 프로그래밍 가이드의 "메모리 처리량 극대화" 섹션](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#maximize-memory-throughput)을 참고하시기 바랍니다.

워프 스케줄러는 [워프 실행 상태](/gpu-glossary/perf/warp-execution-state)도 함께 관리합니다.
