---
title: GPU RAM이란 무엇인가?
---

![H100과 같은 고성능 데이터센터 GPU에서는 프로세서 다이 바로 옆 다이에 RAM이 배치됩니다. 위키백과의 [고대역폭 메모리(High-Bandwidth Memory)](https://en.wikipedia.org/wiki/High_Bandwidth_Memory) 문서를 참고하여 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-hbm-schematic.svg)

GPU 최하위 계층 메모리는 모든 [스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)에서 주소를 지정할 수 있는 대용량(수 메가바이트에서 수십 기가바이트) 저장소입니다.

이 메모리는 흔히 GPU RAM(Random Access Memory) 또는 비디오 RAM(VRAM)이라고 부릅니다. [레지스터](/gpu-glossary/device-hardware/register-file)나 [캐시 메모리](/gpu-glossary/device-hardware/l1-data-cache)에 쓰이는 정적 RAM(SRAM)보다 속도는 느리지만 면적을 덜 차지하는 동적 RAM(DRAM) 셀을 사용합니다. DRAM과 SRAM에 관한 자세한 내용은 Ulrich Drepper의 2007년 아티클인 ["What Every Programmer Should Know About Memory"](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf)를 참고하시기 바랍니다.

GPU RAM은 일반적으로 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)과 동일한 다이에 올라가지 않습니다. 다만 H100 같은 최신 데이터센터급 GPU에서는 지연 시간을 줄이고 [대역폭](/gpu-glossary/perf/memory-bandwidth)을 높이기 위해 공용 [인터포저(interposer)](https://en.wikipedia.org/wiki/Interposer) 위에 함께 배치됩니다. 또한 소비자용 GPU나 CPU에서 흔히 쓰이는 DDR(Double Data Rate) 메모리 대신 [고대역폭 메모리(High-Bandwidth Memory, HBM)](https://en.wikipedia.org/wiki/High_Bandwidth_Memory) 기술을 적용합니다.

RAM은 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)의 [전역 메모리](/gpu-glossary/device-software/global-memory)를 구현하는 데 사용되며, [레지스터 파일](/gpu-glossary/device-hardware/register-file) 용량을 초과하여 넘친(spill) [레지스터](/gpu-glossary/device-software/registers) 데이터를 보관하는 용도로도 쓰입니다.

H100 GPU는 RAM에 최대 80GiB(687,194,767,360비트)를 저장할 수 있습니다.
