---
title: 텐서 메모리란 무엇인가?
---

텐서 메모리(Tensor Memory)는 [B200](https://modal.com/blog/introducing-b200-h200)을 비롯한 일부 최신 GPU의 [스트리밍 다중처리기(SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)에 탑재되어, [텐서 코어](/gpu-glossary/device-hardware/tensor-core)의 입력과 출력을 저장하도록 설계된 특화 메모리입니다.

텐서 메모리 접근은 매우 엄격히 제한됩니다. 데이터는 [워프그룹](/gpu-glossary/device-software/warpgroup) 내 4개 [워프](/gpu-glossary/device-software/warp)가 집합적으로만 다룰 수 있으며, 텐서 메모리와 [레지스터](/gpu-glossary/device-software/registers) 간 특정 패턴에 맞춰 데이터를 전송하거나, [공유 메모리](/gpu-glossary/device-software/shared-memory) 데이터를 텐서 메모리에 기록하거나, 특정 피연산자에 텐서 메모리를 사용하는 행렬 곱셈-누적(MMA) 명령어를 [텐서 코어](/gpu-glossary/device-hardware/tensor-core)에 발행하는 것만 허용됩니다. ['컴퓨트 통합' 디바이스 아키텍처](/gpu-glossary/device-hardware/cuda-device-architecture)라는 표현이 무색할 정도로 고도로 특화된 구조입니다.

구체적으로 `D += A @ B`를 계산하는 `tcgen05.mma` [PTX(Parallel Thread eXecution)](/gpu-glossary/device-software/parallel-thread-execution) 명령어가 텐서 메모리를 활용하려면 다음과 같은 조건을 만족해야 합니다. "누적기(accumulator)" 행렬 `D`는 *반드시* 텐서 메모리에 위치해야 하고, 좌측 행렬 `A`는 텐서 메모리 또는 [공유 메모리](/gpu-glossary/device-software/shared-memory)에 위치할 수 있으며, 우측 행렬 `B`는 텐서 메모리가 아닌 [공유 메모리](/gpu-glossary/device-software/shared-memory)에 *반드시* 있어야 합니다. 구조가 다소 복잡하지만 이는 임의로 결정된 설계가 아닙니다. 행렬 곱셈 과정에서 누적기는 개별 타일(tile)보다 훨씬 자주 접근되므로, [텐서 코어](/gpu-glossary/device-hardware/tensor-core)와 텐서 메모리 사이 배선을 짧고 단순하게 유지하는 특화 하드웨어 구성이 훨씬 더 큰 성능 이점을 제공하기 때문입니다. 이 과정에서 연산 대상 행렬 중 어느 것도 [레지스터](/gpu-glossary/device-software/registers)에 머무르지 않는다는 점도 중요합니다.

주의할 점은, 텐서 메모리가 [텐서 메모리 가속기(TMA)](/gpu-glossary/device-hardware/tensor-memory-accelerator)와 직접적으로 연관되지 않는다는 사실입니다. TMA는 텐서 메모리가 아닌 [L1 데이터 캐시](/gpu-glossary/device-hardware/l1-data-cache)로 데이터를 로드합니다. 대략적으로 말하자면, 데이터는 [텐서 코어](/gpu-glossary/device-hardware/tensor-core) 연산 결과로서만 해당 캐시에서 텐서 메모리로 옮겨지며, 이후 신경망 행렬 곱셈 뒤에 이어지는 비선형 활성화 함수 계산 같은 후처리 작업을 위해 명시적으로 다시 외부로 반출됩니다.

텐서 메모리의 세부 사양과 행렬 곱셈에서의 활용 패턴은 GTC 2025의 [_CUTLASS를 활용한 블랙웰 텐서 코어 프로그래밍(Programming Blackwell Tensor Cores with CUTLASS)_ 강연](https://www.nvidia.com/en-us/on-demand/session/gtc25-s72720/)을 참고하시기 바랍니다.
