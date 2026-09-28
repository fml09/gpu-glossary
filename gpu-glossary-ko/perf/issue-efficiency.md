---
title: 발행 효율이란 무엇인가?
---

발행 효율(Issue Efficiency)은 [워프 스케줄러(Warp Scheduler)](/gpu-glossary/device-hardware/warp-scheduler)가 [발행 가능 워프](/gpu-glossary/perf/warp-execution-state)로부터 명령어를 발행하여 실행 파이프라인을 얼마나 효과적으로 쉬지 않고 가동하는지 측정하는 지표입니다.

![이 다이어그램에 표시된 4개의 클록 사이클 중 3개 사이클에서 명령어가 발행되었으므로 발행 효율은 75%입니다. GTC 2025의 [*연산 및 명령어 처리율 극대화를 위한 CUDA 기법*](https://www.nvidia.com/en-us/on-demand/session/gtc25-s72685/) 발표 내용을 참고하여 작성하였습니다.](https://modal-cdn.com/gpu-glossary/light-cycles.svg)

발행 효율이 100%라는 것은 모든 [스케줄러](/gpu-glossary/device-hardware/warp-scheduler)가 매 사이클마다 명령어를 성공적으로 발행했음을 의미하며, 이는 매 사이클마다 적어도 하나 이상의 [발행 가능 워프](/gpu-glossary/perf/warp-execution-state)가 존재했음을 나타냅니다. 100% 미만의 수치는 일부 사이클 동안 모든 [활성 워프](/gpu-glossary/perf/warp-execution-state)가 데이터, 하드웨어 리소스 또는 명령어 간 종속성 문제로 인해 [스톨(Stalled)](/gpu-glossary/perf/warp-execution-state) 상태에 놓였음을 나타냅니다. 이 경우 스케줄러는 유휴(idle) 상태로 대기하게 되며 전반적인 명령어 처리율이 저하됩니다.
