---
title: 레지스터 압박이란 무엇인가?
---

레지스터 압박(Register Pressure)은 [레지스터 파일(Register File)](/gpu-glossary/device-hardware/register-file)의 용량 한계로 인해 [병목 현상(Bottleneck)](/gpu-glossary/perf/performance-bottleneck)이 발생할 때 사용하는 표현입니다.

[병렬 스레드 실행(PTX)](/gpu-glossary/device-software/parallel-thread-execution) 언어에서 [레지스터](/gpu-glossary/device-software/registers)는 가상화되어 있어 무제한으로 쓸 수 있지만, [스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)의 물리적 [레지스터 파일](/gpu-glossary/device-hardware/register-file) 용량은 엄격히 제한되어 있습니다.

한 [스레드(Thread)](/gpu-glossary/device-software/thread)가 차지하는 [레지스터 파일](/gpu-glossary/device-hardware/register-file)의 공간은 해당 [커널(Kernel)](/gpu-glossary/device-software/kernel)을 컴파일한 [스트리밍 어셈블러(SASS)](/gpu-glossary/device-software/streaming-assembler) 코드에 의해 결정됩니다. 또한 한 [스레드 블록(Thread Block)](/gpu-glossary/device-software/thread-block)에 속한 모든 [스레드](/gpu-glossary/device-software/thread)는 동일한 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)에 스케줄링되므로, 하나의 스레드 블록이 요구하는 총 레지스터 공간은 커널 실행 설정(블록당 스레드 수)에 의해서도 결정됩니다. [스레드 블록](/gpu-glossary/device-software/thread-block) 하나에 할당되는 레지스터 공간이 늘어날수록 단일 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)에 동시에 상주할 수 있는 스레드 블록의 수는 줄어들게 되며, 이는 [점유율(Occupancy)](/gpu-glossary/perf/occupancy)을 떨어뜨려 명령어 [지연 시간을 은닉](/gpu-glossary/perf/latency-hiding)하기 어렵게 만듭니다.

레지스터 압박과 최신 [스트리밍 다중처리기 아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture)에 추가된 핵심 기능들(암페어 아키텍처의 비동기 복사, 호퍼 아키텍처의 [텐서 메모리 가속기(TMA)](/gpu-glossary/device-hardware/tensor-memory-accelerator), 블랙웰 아키텍처의 [텐서 메모리](/gpu-glossary/device-hardware/tensor-memory)) 사이의 상관관계에 대해서는 [SemiAnalysis의 분석 글](https://semianalysis.com/2025/06/23/nvidia-tensor-core-evolution-from-volta-to-blackwell/)을 참조하시기 바랍니다.

레지스터 압박은 CPU에서도 발생합니다. CPU에서도 유사한 레지스터 [병목](/gpu-glossary/perf/performance-bottleneck)으로 인해 [자동 벡터화 과정에서 루프 스트립 마이닝(Strip-Mining)](https://hogback.atmos.colostate.edu/rr/old/tidbits/intel/macintel/doc_files/source/extfile/optaps_for/common/optaps_vec_mine.htm)을 적용할 수 있는 범위가 제한되기도 합니다.
