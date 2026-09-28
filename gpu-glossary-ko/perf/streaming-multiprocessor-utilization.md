---
title: SM 활용률이란 무엇인가?
---

SM 활용률(Streaming Multiprocessor Utilization)은 [스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)가 명령어를 실행하고 있는 시간의 백분율을 측정하는 지표입니다.

SM 활용률은 [`nvidia-smi`](/gpu-glossary/host-software/nvidia-smi)에서 흔히 확인하는 [커널 활용률(Kernel Utilization)](https://modal.com/blog/gpu-utilization-guide)과 유사하지만, 훨씬 세밀한 단위로 측정됩니다. GPU의 어느 한 곳에서라도 [커널](/gpu-glossary/device-software/kernel)이 실행 중인 시간의 비율을 보고하는 대신, GPU 내의 모든 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)이 실제로 [커널](/gpu-glossary/device-software/kernel)을 실행하는 데 소비한 시간의 비율을 집계합니다. 예를 들어 단 하나의 [스레드 블록](/gpu-glossary/device-software/thread-block)만 생성되어 [커널](/gpu-glossary/device-software/kernel)이 단 1개의 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)만 사용하는 경우, 해당 커널이 동작하는 동안 GPU 커널 활용률은 100%로 표시되지만, 실제 SM 활용률은 최대 전체 SM 개수분의 1(H100 GPU 기준 1% 미만)에 불과합니다.

[CPU 활용률과 달리 GPU 활용률은 높을수록 바람직하듯이](https://modal.com/blog/gpu-utilization-guide), SM 활용률 역시 100%에 근접할 정도로 높은 것이 이상적입니다.

그러나 SM 활용률이 일반 GPU 활용률보다 훨씬 세밀하다고 하더라도, GPU의 내부 연산 리소스가 얼마나 알차게 사용되고 있는지를 파악하기에는 여전히 충분하지 않습니다. SM 활용률이 매우 높은데도 애플리케이션의 처리 성능이 여전히 부족하다면, 각 SM이 내부 기능 유닛을 얼마나 효과적으로 활용하는지 측정하는 [파이프라인 활용률(Pipe Utilization)](/gpu-glossary/perf/pipe-utilization)을 점검해야 합니다. SM 활용률은 높은데 [파이프라인 활용률](/gpu-glossary/perf/pipe-utilization)이 낮다면, [커널](/gpu-glossary/device-software/kernel)이 수많은 SM에 분산되어 실행되기는 하지만 각 SM 내부의 하드웨어 연산 리소스를 온전히 포화시키지 못하고 있음을 의미합니다.
