---
title: GPU 코어란 무엇인가?
---

코어(Core)는 [스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)를 구성하는 주요 연산 유닛입니다.

![H100 GPU의 스트리밍 다중처리기 내부 아키텍처. CUDA 코어와 텐서 코어가 녹색으로 표시되어 있습니다. NVIDIA의 [H100 백서](https://modal-cdn.com/gpu-glossary/gtc22-whitepaper-hopper.pdf)를 기반으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-gh100-sm.svg)

GPU 코어 유형의 대표적인 예로는 [CUDA 코어(CUDA Core)](/gpu-glossary/device-hardware/cuda-core)와 [텐서 코어(Tensor Core)](/gpu-glossary/device-hardware/tensor-core)가 있습니다.

GPU 코어는 실제 연산을 수행하는 구성 요소라는 점에서 CPU 코어와 유사해 보이지만, 이러한 비유는 상당한 오해를 불러일으킬 수 있습니다. 오히려 [정량적 컴퓨터 구조학](https://archive.org/details/computerarchitectureaquantitativeapproach6thedition) 관점에서, 데이터가 들어가서 변환된 데이터가 나오는 일종의 '파이프(pipe)'로 생각하는 편이 이해하기에 더 적절합니다. 이러한 파이프는 하드웨어 관점에서는 특정 [명령어](/gpu-glossary/device-software/streaming-assembler)와 연결되며, 프로그래머 관점에서는 근본적으로 서로 다른 처리량 특성(예: [텐서 코어](/gpu-glossary/device-hardware/tensor-core)의 부동소수점 행렬 곱셈 연산 처리량)을 제공합니다.

실제 CPU 코어에 더 가까운 구성 요소는 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)입니다. SM은 정보를 저장하는 [레지스터 메모리](/gpu-glossary/device-hardware/register-file), 데이터를 변환하는 코어, 그리고 데이터 변환을 지정하고 지시하는 [명령어 스케줄러](/gpu-glossary/device-hardware/warp-scheduler)를 모두 갖추고 있습니다.
