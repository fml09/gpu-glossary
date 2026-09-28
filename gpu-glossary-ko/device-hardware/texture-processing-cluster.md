---
title: 텍스처 처리 클러스터(TPC)란 무엇인가?
abbreviation: TPC
---

텍스처 처리 클러스터(Texture Processing Cluster, TPC)는 서로 인접한 두 [스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)로 묶인 하드웨어 유닛입니다.

블랙웰(Blackwell) [SM 아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture)가 등장하기 전까지, TPC는 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)의 [메모리 계층 구조](/gpu-glossary/device-software/memory-hierarchy)나 [스레드 계층 구조](/gpu-glossary/device-software/thread-hierarchy)의 특정 계층에 직접 매핑되지 않았습니다.

하지만 블랙웰 [SM 아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture)의 5세대 [텐서 코어](/gpu-glossary/device-hardware/tensor-core)가 도입되면서 [PTX(Parallel Thread eXecution)](/gpu-glossary/device-software/parallel-thread-execution) [스레드 계층 구조](/gpu-glossary/device-software/thread-hierarchy)에 TPC와 대응하는 "CTA 쌍(CTA pair)" 계층이 새롭게 추가되었습니다. 다수의 `tcgen05` [PTX](/gpu-glossary/device-software/parallel-thread-execution) 명령어에는 단일 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor)을 사용하는 `.cta_group::1`이나 TPC 내 SM 한 쌍을 묶어 사용하는 `::2`를 지정할 수 있는 `.cta_group` 필드가 포함됩니다. 이는 `MMA` 같은 [SASS(Streaming Assembler)](/gpu-glossary/device-software/streaming-assembler) 명령어의 `1SM` 및 `2SM` 변형에 매핑됩니다.
