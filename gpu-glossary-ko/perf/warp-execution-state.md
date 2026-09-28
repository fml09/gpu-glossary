---
title: 워프 실행 상태란 무엇인가?
---

[커널(Kernel)](/gpu-glossary/device-software/kernel)을 실행하는 [워프(Warp)](/gpu-glossary/device-software/warp)의 상태는 상호 배타적이지 않은 몇 가지 상태 형용사로 설명됩니다. 바로 활성(Active), 스톨(Stalled), 발행 가능(Eligible), 선택됨(Selected) 상태입니다.

![워프 실행 상태가 색상별로 구분되어 표시되어 있습니다. GTC 2025의 [*연산 및 명령어 처리율 극대화를 위한 CUDA 기법*](https://www.nvidia.com/en-us/on-demand/session/gtc25-s72685/) 발표 내용을 참고하여 작성하였습니다.](https://modal-cdn.com/gpu-glossary/light-cycles.svg)

[워프](/gpu-glossary/device-software/warp)는 소속 [스레드(Thread)](/gpu-glossary/device-software/thread)들이 실행을 시작한 시점부터 워프 내의 모든 스레드가 [커널](/gpu-glossary/device-software/kernel) 실행을 완전히 종료할 때까지 *활성(Active)* 상태로 간주됩니다. 활성 워프들은 매 사이클마다 [워프 스케줄러(Warp Scheduler)](/gpu-glossary/device-hardware/warp-scheduler)가 명령어를 발행할 후보(즉, 명령어 발행 슬롯에 배치할 대상)를 선택하는 워프 풀(pool)을 형성합니다.

[스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)당 허용되는 최대 활성 [워프](/gpu-glossary/device-software/warp) 수는 [아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture)마다 다르며, [컴퓨트 성능(Compute Capability)](/gpu-glossary/device-software/compute-capability)에 관한 [NVIDIA 공식 문서](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html?highlight=compute%2520capability#compute-capabilities)에 명시되어 있습니다. 예를 들어 컴퓨트 성능 9.0인 H100 SXM GPU의 경우 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)당 최대 64개의 활성 워프(2,048개 스레드)가 상주할 수 있습니다. 활성 워프라고 해서 매 사이클마다 반드시 명령어를 실행하고 있는 것은 아닙니다. 위 다이어그램에서는 단 하나의 슬롯을 제외한 모든 슬롯과 사이클에 활성 워프가 존재하며, 이는 높은 [점유율(Occupancy)](/gpu-glossary/perf/occupancy)을 나타냅니다.

*발행 가능(Eligible)* [워프](/gpu-glossary/device-software/warp)는 다음 명령어를 발행할 준비가 완전히 끝난 활성 워프를 의미합니다. 워프가 발행 가능 상태가 되려면 다음 조건들이 모두 충족되어야 합니다.

- 다음 실행할 명령어가 이미 인출(fetch)되어 있어야 합니다.
- 해당 명령어에 필요한 실행 유닛(하드웨어 파이프라인)이 사용 가능한 상태여야 합니다.
- 이전 명령어와의 모든 데이터 종속성이 해결되어 있어야 합니다.
- 실행을 가로막는 동기화 배리어(barrier)가 없어야 합니다.

발행 가능 [워프](/gpu-glossary/device-software/warp)는 [워프 스케줄러](/gpu-glossary/device-hardware/warp-scheduler)가 즉시 명령어를 발행할 수 있는 직접적인 후보군입니다. 위 다이어그램에서 사이클 n + 2를 제외한 모든 사이클에 발행 가능 워프가 존재합니다. 여러 사이클에 걸쳐 발행 가능 워프가 전혀 존재하지 않는 상황이 지속되면, 특히 [CUDA 코어](/gpu-glossary/device-hardware/cuda-core)처럼 지연 시간이 짧은 산술 연산 유닛을 주로 사용할 때 성능이 심각하게 저하될 수 있습니다.

*스톨(Stalled)* [워프](/gpu-glossary/device-software/warp)는 해결되지 않은 종속성이나 하드웨어 리소스 충돌로 인해 다음 명령어를 발행하지 못하고 대기 중인 활성 워프를 가리킵니다. 워프가 스톨 상태에 빠지는 주된 이유는 다음과 같습니다.

- 실행 종속성: 이전 산술 연산 명령어의 실행 결과를 기다려야 하는 경우
- 메모리 종속성: 이전 메모리 연산 결과가 도착하기를 기다려야 하는 경우
- 파이프라인 충돌: 필요한 실행 유닛이 현재 다른 연산에 의해 점유되어 있는 경우

[SM](/gpu-glossary/device-hardware/streaming-multiprocessor) 외부로 벗어나지 않는 가변 지연 시간 명령어 대기로 인해 워프가 스톨되는 경우를 "단기 스코어보드(short scoreboard)" 스톨이라 부르며, SM 외부로 나가는 가변 지연 시간 메모리 연산 대기로 인해 스톨되는 경우를 "장기 스코어보드(long scoreboard)" 스톨이라 부릅니다. 이 두 유형을 통칭하여 [스코어보드 스톨(Scoreboard Stall)](/gpu-glossary/perf/scoreboard-stall)이라고 합니다.

위 다이어그램에서는 매 사이클마다 여러 슬롯에서 스톨된 [워프](/gpu-glossary/device-software/warp)를 확인할 수 있습니다. 워프가 스톨되었다고 해서 본질적으로 나쁜 것만은 아닙니다. 메모리 로드나 [수십 사이클 동안 실행될 수 있는](https://arxiv.org/abs/2206.02874) `HMMA`와 같은 [텐서 코어](/gpu-glossary/device-hardware/tensor-core) 명령어처럼 실행 시간이 긴 명령어의 [지연 시간을 은닉](/gpu-glossary/perf/latency-hiding)하려면, 동시에 스톨 상태에 머무르는 다수의 워프 집합이 반드시 필요하기 때문입니다.

*선택됨(Selected)* [워프](/gpu-glossary/device-software/warp)는 현재 사이클에 명령어를 실행하도록 [워프 스케줄러](/gpu-glossary/device-hardware/warp-scheduler)에 의해 최종 선택된 발행 가능 워프를 의미합니다. 매 사이클마다 [워프 스케줄러](/gpu-glossary/device-hardware/warp-scheduler)는 발행 가능 워프 풀을 살펴보고, 후보가 존재할 경우 그중 하나를 선택하여 명령어를 발행합니다. 발행 가능 워프가 존재하는 모든 사이클에는 항상 선택된 워프가 있습니다. [활성 사이클](/gpu-glossary/perf/active-cycle) 중에서 실제로 워프가 선택되어 명령어가 성공적으로 발행된 사이클의 비율을 [발행 효율(Issue Efficiency)](/gpu-glossary/perf/issue-efficiency)이라고 합니다.
