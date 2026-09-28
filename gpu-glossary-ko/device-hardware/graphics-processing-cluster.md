---
title: 그래픽/GPU 처리 클러스터(GPC)란 무엇인가?
abbreviation: GPC
---

GPC(Graphics/GPU Processing Cluster)는 여러 [텍스처 처리 클러스터(Texture Processing Cluster, TPC)](/gpu-glossary/device-hardware/texture-processing-cluster)와 래스터 엔진(raster engine)이 결합된 하드웨어 유닛입니다(TPC 자체는 여러 [스트리밍 다중처리기(SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)로 이루어진 그룹입니다). 래스터 엔진은 그래픽 렌더링 작업에 핵심적인 역할을 수행합니다. 과거 GPC는 '그래픽 처리 클러스터(Graphics Processing Cluster)'의 약어로 사용되었으나, 최근에는 [NVIDIA CUDA C++ 프로그래밍 가이드](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html) 등에서 'GPU 처리 클러스터(GPU Processing Cluster)'로 표기하기도 합니다.

H100을 비롯한 [컴퓨트 성능](/gpu-glossary/device-software/compute-capability) 9.0 GPU가 도입되면서, [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)의 [스레드 계층 구조](/gpu-glossary/device-software/thread-hierarchy)에 '클러스터(Cluster)'라는 새로운 계층이 추가되었습니다. 하나의 [스레드 블록](/gpu-glossary/device-software/thread-block)에 속한 스레드들이 단일 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)에 스케줄링되는 것처럼, 스레드 블록들로 묶인 클러스터는 동일한 GPC에 스케줄링됩니다. 또한 이러한 클러스터는 [메모리 계층 구조](/gpu-glossary/device-software/memory-hierarchy)에서 분산 공유 메모리(Distributed Shared Memory)라는 전용 계층을 활용합니다. 본 문서의 다른 항목에서는 이 기능에 대한 세부 설명을 다루지 않습니다.
