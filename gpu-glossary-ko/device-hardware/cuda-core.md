---
title: CUDA 코어란 무엇인가?
---

CUDA 코어(CUDA Core)는 스칼라 산술 연산 명령어를 실행하는 GPU [코어](/gpu-glossary/device-hardware/core)입니다.

![H100 SM 내부 아키텍처. CUDA 코어와 텐서 코어가 녹색으로 표시되어 있습니다. 텐서 코어의 크기가 더 크고 개수가 적은 점에 주목하십시오. NVIDIA의 [H100 백서](https://modal-cdn.com/gpu-glossary/gtc22-whitepaper-hopper.pdf)를 기반으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-gh100-sm.svg)

CUDA 코어는 행렬 연산을 전담하는 [텐서 코어(Tensor Core)](/gpu-glossary/device-hardware/tensor-core)와 대비되는 개념입니다.

CPU 코어와 달리, CUDA 코어에 전달되는 명령어는 일반적으로 독립적으로 스케줄링되지 않습니다. 대신 [워프 스케줄러(Warp Scheduler)](/gpu-glossary/device-hardware/warp-scheduler)가 여러 코어로 구성된 그룹에 동일한 명령어를 동시에 발행하며, 각 코어는 서로 다른 [레지스터](/gpu-glossary/device-software/registers)를 대상으로 해당 명령어를 수행합니다. 통상적으로 이러한 그룹은 [워프(Warp)](/gpu-glossary/device-software/warp) 단위에 해당하는 32개의 스레드로 구성되지만, 최신 GPU 아키텍처에서는 성능 저하를 감수하면서도 최소 1개의 스레드 단위로 그룹을 구성할 수 있습니다.

'CUDA 코어'라는 용어는 맥락에 따라 의미가 달라질 수 있습니다. 서로 다른 [스트리밍 다중처리기 아키텍처(Streaming Multiprocessor Architecture)](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture)에서 CUDA 코어는 32비트 정수 연산 유닛, 32비트 부동소수점 연산 유닛, 64비트 부동소수점 연산 유닛 등이 서로 다른 비율로 조합된 하드웨어 유닛을 가리킵니다. 셰이더 파이프라인에 매핑된 매우 특화된 다양한 연산 유닛을 포함하고 있었던 초기 GPU와 비교하여 이해하면 개념을 파악하기가 수월합니다([CUDA 디바이스 아키텍처](/gpu-glossary/device-hardware/cuda-device-architecture) 참조).

예를 들어, [H100 백서](https://resources.nvidia.com/en-us-hopper-architecture/nvidia-h100-tensor-c)에서는 H100 GPU의 각 [스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)마다 128개의 'FP32 CUDA 코어'가 탑재되어 있다고 기술하고 있습니다. 이는 32비트 부동소수점 연산 유닛의 개수를 정확히 나타내지만, 위의 도표에서 확인할 수 있듯이 32비트 정수 연산 유닛이나 64비트 부동소수점 연산 유닛의 개수보다는 두 배 많은 수치입니다. 따라서 실제 연산 성능을 추정할 때는 단순한 코어 명칭보다는 수행하려는 연산에 직접 할당된 하드웨어 유닛의 실제 개수를 확인하는 것이 가장 바람직합니다.
