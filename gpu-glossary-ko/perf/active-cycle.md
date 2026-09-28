---
title: 활성 사이클이란 무엇인가?
---

활성 사이클(Active Cycle)은 [스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)에 상주하는 [활성 워프(Active Warp)](/gpu-glossary/perf/warp-execution-state)가 적어도 하나 이상 존재하는 클록 사이클을 의미합니다. 이때 상주하는 [워프(Warp)](/gpu-glossary/device-software/warp)는 [발행 가능(Eligible)](/gpu-glossary/perf/warp-execution-state) 상태이거나 [스톨(Stalled)](/gpu-glossary/perf/warp-execution-state) 상태일 수 있습니다.

![이 다이어그램에 표시된 모든 사이클은 활성 사이클에 해당합니다. GTC 2025의 [*연산 및 명령어 처리율 극대화를 위한 CUDA 기법(CUDA Techniques to Maximize Compute and Instruction Throughput)*](https://www.nvidia.com/en-us/on-demand/session/gtc25-s72685/) 발표 내용을 참고하여 작성하였습니다.](https://modal-cdn.com/gpu-glossary/light-cycles.svg)
