---
title: 스트리밍 다중처리기(SM)란 무엇인가?
abbreviation: SM
---

우리가 [GPU를 프로그래밍할 때](/gpu-glossary/host-software/cuda-software-platform), GPU의 스트리밍 다중처리기가 실행할 [명령어 시퀀스](/gpu-glossary/device-software/streaming-assembler)를 생성하게 됩니다.

![H100 GPU의 스트리밍 다중처리기 내부 아키텍처 다이어그램. GPU 코어는 녹색, 기타 연산 유닛은 분홍색, 스케줄링 유닛은 주황색, 메모리는 파란색으로 표시되어 있습니다. NVIDIA의 [H100 백서](https://modal-cdn.com/gpu-glossary/gtc22-whitepaper-hopper.pdf)를 기반으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-gh100-sm.svg)

NVIDIA GPU의 스트리밍 다중처리기(Streaming Multiprocessor, SM)는 개념상 CPU 코어와 유사합니다. 즉, SM은 연산을 직접 수행할 뿐만 아니라 연산에 필요한 상태 데이터를 관련 캐시 및 레지스터에 저장합니다. 그러나 CPU 코어와 비교하면 GPU SM은 단순하며 개별 연산 능력이 약한 프로세서입니다. SM 내부 실행은 단일 명령어 내에서 파이프라인화되어 있지만(1990년대 이후 거의 모든 CPU와 유사), 투기적 실행이나 명령어 포인터 분기 예측 기능은 탑재하지 않았습니다(오늘날 모든 고성능 CPU와 구별되는 점입니다).

하지만 GPU SM은 훨씬 더 많은 [스레드](/gpu-glossary/device-software/thread)를 병렬로 실행할 수 있습니다.

비교를 위해 살펴보면, [AMD EPYC 9965](https://www.techpowerup.com/cpu-specs/epyc-9965.c3904) CPU는 최대 500W 전력을 소모하며 192개 코어를 탑재하고 있습니다. 각 코어는 한 번에 최대 2개 스레드의 명령어를 실행할 수 있어 총 384개 스레드를 병렬 실행하며, 스레드당 약 1.25W를 소모합니다.

반면 H100 SXM GPU는 최대 700W를 소모하며 132개 SM을 갖추고 있습니다. 각 SM에는 4개의 [워프 스케줄러](/gpu-glossary/device-hardware/warp-scheduler)가 탑재되어 클록 사이클마다 각각 32개 스레드(즉, [워프](/gpu-glossary/device-software/warp))에 명령어를 동시 발행할 수 있습니다. 이를 합산하면 클록 사이클마다 총 128 × 132 > 16,000개 이상의 스레드가 병렬 실행되며, 스레드당 소모 전력은 약 0.05W(5cW)에 불과합니다. 주목할 점은 이것이 진정한 의미의 병렬 실행이라는 사실입니다. 즉, 16,000개 스레드 각각이 매 클록 사이클마다 실제로 진행됩니다.

또한 GPU SM은 명령어들이 교차로 실행되는 방대한 수의 *동시성(concurrent)* 스레드를 지원합니다.

H100의 단일 SM은 32개 스레드로 구성된 스레드 그룹 64개에 걸쳐 최대 2,048개의 스레드를 동시에 유지하고 교차 실행할 수 있습니다. 132개 SM 전체로 환산하면 250,000개가 넘는 동시 스레드를 처리할 수 있는 셈입니다.

CPU 역시 많은 스레드를 동시에 실행할 수 있습니다. 하지만 GPU SM에서는 [워프 스케줄러](/gpu-glossary/device-hardware/warp-scheduler)를 통해 [워프](/gpu-glossary/device-software/warp) 간 문맥 전환이 단 1클록 사이클 만에 이루어집니다(CPU의 문맥 교환보다 1,000배 이상 빠릅니다). 이처럼 즉각 전환 가능한 대규모 [워프](/gpu-glossary/device-software/warp) 풀과 빠른 [워프 전환](/gpu-glossary/device-hardware/warp-scheduler) 덕분에 메모리 읽기, 스레드 동기화, 기타 지연 시간이 긴 명령어로 발생하는 대기 시간을 [은닉](/gpu-glossary/perf/latency-hiding)할 수 있습니다. 그 결과 [CUDA 코어](/gpu-glossary/device-hardware/cuda-core)와 [텐서 코어](/gpu-glossary/device-hardware/tensor-core)가 제공하는 [연산 대역폭](/gpu-glossary/perf/arithmetic-bandwidth)을 빈틈없이 활용할 수 있습니다.

이러한 [지연 시간 은닉](/gpu-glossary/perf/latency-hiding)은 GPU 성능을 지탱하는 핵심 비결입니다. CPU는 대규모 하드웨어 관리 캐시와 정교한 명령어 분기 예측 기법을 동원하여 사용자와 프로그래머로부터 지연 시간을 감추고자 합니다. 하지만 이러한 부가 하드웨어 때문에 실리콘 면적, 전력, 발열 예산 중 실제 순수 연산 유닛에 할당할 수 있는 비중이 크게 줄어듭니다.

![GPU는 CPU에 비해 면적 중 연산(녹색)에 더 많은 비중을 할당하고, 제어 및 캐싱(주황색과 파란색)에는 적은 비중을 할당합니다. [Fabien Sanglard의 블로그](https://fabiensanglard.net/cuda) 다이어그램을 참고하여 수정하였으며, 해당 원본은 [CUDA C 프로그래밍 가이드](https://docs.nvidia.com/cuda/cuda-c-programming-guide/)의 다이어그램을 바탕으로 작성된 것으로 보입니다.](https://modal-cdn.com/gpu-glossary/light-cpu-vs-gpu.svg)

반면 신경망 추론이나 순차적 데이터베이스 스캔처럼 프로그래머가 [캐시](/gpu-glossary/device-hardware/l1-data-cache) 동작을 비교적 명확하게 [표현](/gpu-glossary/device-software/cuda-programming-model)할 수 있는 워크로드의 경우(예: 각 입력 행렬의 일부분을 로드하여 관련 출력을 계산할 때까지 캐시에 유지하는 구조), 제어 로직을 덜어내고 연산 유닛에 집중한 GPU 아키텍처가 훨씬 더 높은 처리량을 발휘합니다.
