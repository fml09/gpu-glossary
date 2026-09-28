---
title: 스코어보드 스톨이란 무엇인가?
---

스코어보드 스톨(Scoreboard Stall)은 이전 명령어의 실행 결과에 대한 데이터 종속성 때문에 다음 명령어를 즉시 발행하지 못할 때 발생합니다.

스코어보드(Scoreboard)는 현재 실행 중(in-flight)인 명령어에 의해 쓰기 작업이 예정되어 대기 중인 [레지스터](/gpu-glossary/device-software/registers)들을 추적하는 하드웨어 구조입니다. [워프(Warp)](/gpu-glossary/device-software/warp)는 [스톨 상태](/gpu-glossary/perf/warp-execution-state)에 머무르는 동안 실행을 계속 진행할 수 없습니다.

스코어보드 스톨은 크게 단기 스코어보드 스톨(Short Scoreboard Stall)과 장기 스코어보드 스톨(Long Scoreboard Stall)의 두 가지 유형으로 분류됩니다.

단기 스코어보드 스톨은 [스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor) 외부로 벗어나지 않는 가변 지연 시간 명령어의 결과를 기다릴 때 발생합니다. 대표적으로 `LDS`나 `STS` 같은 [공유 메모리(Shared Memory)](/gpu-glossary/device-software/shared-memory) 연산이 여기에 해당합니다. 또한 다소 생소할 수 있지만, [공유 메모리](/gpu-glossary/device-software/shared-memory) 접근과 동일한 메모리 입출력(MIO) 하드웨어 경로를 공유한다는 [구조적 특성](https://stackoverflow.com/questions/66123750/what-are-the-long-and-short-scoreboards-w-r-t-mio-l1tex)으로 인해 특정 [특수 기능 유닛(SFU)](/gpu-glossary/device-hardware/special-function-unit) 연산 대기도 단기 스코어보드 스톨에 포함됩니다.

장기 스코어보드 스톨은 [전역 메모리(Global Memory)](/gpu-glossary/device-software/global-memory) 로드(`LDG`)나 스토어(`STG`)처럼 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor) 외부로 나갈 수 있는 메모리 연산 결과를 기다릴 때 발생합니다. 장기 스코어보드 스톨은 [메모리 제약(Memory Bound)](/gpu-glossary/perf/memory-bound) 코드에서 발생하는 스톨의 절대다수를 차지합니다.

일부 스코어보드 정보는 [스트리밍 어셈블러(SASS)](/gpu-glossary/device-software/streaming-assembler) 코드에서 직접 확인할 수 있습니다. 예를 들어 컴파일러는 각 [워프](/gpu-glossary/device-software/warp)마다 6개의 고유한 배리어 식별자를 할당하여 명령어 간 데이터 종속성을 추적합니다.

구체적으로 `--dump-sass` 플래그를 붙여 `cuobjdump`를 실행하면 다음과 같은 형태의 코드를 확인할 수 있습니다.

```nasm
[barrier:  :  :  :  ]  /*line*/  INSTRUCTION Ri, [Rj] ; # Format: scoreboard info, line number, instruction, operands
[B------:R-:W2:-:S04]  /*00f0*/  LDG.E.SYS R0, [R2] ;   # Sets scoreboard 2
[B------:R-:W2:-:S01]  /*0100*/  LDG.E.SYS R5, [R4] ;   # `ptxas` intelligently reuses scoreboard 2
...
[B--2---:R-:W-:Y:S08]  /*0150*/  IMAD R0, R0, c[0x0][0x160], R5 ;  # Waits on scoreboard 2
```

위 코드에서 `IMAD` 명령어는 스코어보드 2번에 배리어(`B--2---`)가 걸려 있음을 확인할 수 있으며, 이는 명령어가 발행되기 전에 해당 비트 플래그가 클리어되어야 함을 나타냅니다. 앞선 두 개의 `LDG` 명령어는 발행될 때 스코어보드 2번을 설정(`W2` 쓰기)하여, `IMAD` 명령어가 실행되기 전에 `R0` 및 `R5` 레지스터에 올바른 값이 준비되도록 보장합니다.

여러 스코어보드에 동시에 배리어를 설정할 수도 있습니다. 예를 들어 `B01--4-`는 0번, 1번, 4번 스코어보드가 모두 클리어될 때까지 대기함을 의미합니다. 데이터 종속성이 충족되면 해당하는 스코어보드의 카운터가 감소합니다.

스코어보드가 재사용되는 경우, 장기 스코어보드 스톨과 단기 스코어보드 스톨이 동일한 스코어보드를 공유하면서 서로 혼동되어 Nsight Compute가 보고하는 스톨 분류가 부정확해질 수 있습니다.

동적 명령어 스케줄링에서 종속성 추적을 위한 [스코어보딩(Scoreboarding)](https://www.cs.umd.edu/~meesh/411/website/projects/dynamic/scoreboard.html) 기법은 1966년 오일러의 거듭제곱의 합 추측을 [반증](https://www.ams.org/journals/bull/1966-72-06/S0002-9904-1966-11654-3/S0002-9904-1966-11654-3.pdf)하는 데 기여했던 "최초의 슈퍼컴퓨터" [CDC 6600(Control Data Corporation 6600)](https://en.wikipedia.org/wiki/CDC_6600) 시절까지 거슬러 올라갑니다. CPU와 달리 GPU에서의 스코어보딩은 단일 [스레드](/gpu-glossary/device-software/thread) 내부의 비순차적 실행(명령어 수준 병렬성)을 위해 사용되지 않고, 여러 스레드 및 워프 간의 동시 실행(스레드 수준 병렬성)을 조율하는 데 사용됩니다([해당 NVIDIA 특허](https://patents.google.com/patent/US7676657) 참조).

GPU 스코어보드 구현에 관한 자세한 내용은 [Matthew D. Sinclair 교수의 강의 자료](https://pages.cs.wisc.edu/~sinclair/courses/cs758/fall2019/handouts/lecture/cs758-fall19-gpu_uarch2.pdf)를 참고하시기 바랍니다.
