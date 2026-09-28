---
title: 뱅크 충돌이란 무엇인가?
---

한 [워프(Warp)](/gpu-glossary/device-software/warp)에 속한 여러 [스레드(Thread)](/gpu-glossary/device-software/thread)가 [공유 메모리(Shared Memory)](/gpu-glossary/device-software/shared-memory) 내의 동일한 뱅크에 속하면서 서로 다른 주소를 가진 데이터에 동시에 접근을 요청할 때, 이를 뱅크 충돌(Bank Conflict)이라고 부릅니다.

![스레드가 서로 다른 공유 메모리 뱅크에 접근할 때는 접근이 병렬로 처리됩니다(왼쪽). 반면 모든 스레드가 동일한 뱅크의 서로 다른 주소에 접근하면 접근이 직렬화됩니다(오른쪽).](https://modal-cdn.com/gpu-glossary/light-bank-conflict.svg)

뱅크 충돌이 발생하면 서로 다른 [스레드](/gpu-glossary/device-software/thread)의 메모리 요청이 순차적으로 직렬화(serialized)되어 처리됩니다. 이로 인해 메모리 처리량이 정수 배수만큼 대폭 감소하여 [메모리 대역폭](/gpu-glossary/perf/memory-bandwidth)을 온전히 활용하지 못하게 됩니다.

여타 SRAM 기반 캐시 메모리와 마찬가지로, [스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)의 [공유 메모리](/gpu-glossary/device-software/shared-memory)는 '뱅크(Bank)'라는 단위 그룹으로 나뉘어 구성됩니다. 이러한 뱅크들은 동시에 독립적으로 접근할 수 있으므로 전체 대역폭을 향상시키는 역할을 합니다.

GPU에는 통상 32개의 뱅크가 존재하며, 각 뱅크는 4바이트(32비트) 폭을 가집니다. 연속된 32비트 워드(GPU는 32비트 부동소수점 및 정수 연산을 기본으로 설계되었습니다)는 연속된 뱅크에 순차적으로 매핑됩니다.

```
Address:  0x00  0x04  0x08  0x0C  0x10  0x14  0x18  0x1C  ...  0x7C
Bank:       0     1     2     3     4     5     6     7   ...    31

Address:  0x80  0x84  0x88  0x8C  0x90  0x94  0x98  0x9C  ...  0xFC
Bank:       0     1     2     3     4     5     6     7   ...    31
```

32 × 4 = 128바이트 간격으로 떨어져 있는 주소들은 모두 동일한 뱅크에 매핑됩니다. [공유 메모리](/gpu-glossary/device-software/shared-memory) 용량은 킬로바이트 단위이므로, 여러 서로 다른 주소가 필연적으로 동일한 뱅크에 매핑됩니다.

공유 메모리 배열의 연속된 요소를 순서대로 접근하면, [워프](/gpu-glossary/device-software/warp) 내의 각 [스레드](/gpu-glossary/device-software/thread)는 모두 서로 다른 뱅크에 접근하게 됩니다.

```cpp
__shared__ float data[1024];  // array in shared memory

// all 32 threads access consecutive elements of data
int tid = threadIdx.x;
float value = data[tid];  // address LSBs: 0x00, 0x04, 0x08, ...
```

각 [스레드](/gpu-glossary/device-software/thread)가 서로 다른 뱅크에 접근하므로, 32개 스레드의 모든 메모리 접근이 단 한 번의 메모리 트랜잭션으로 완료됩니다. 이는 위 다이어그램의 왼쪽에 해당합니다.

하지만 한 행에 32개 요소가 포함된 행 우선(Row-Major) 방식의 [공유 메모리](/gpu-glossary/device-software/shared-memory) 배열에서 [스레드](/gpu-glossary/device-software/thread)들이 동일한 열(Column)에 접근하도록 다음과 같이 코드를 작성했다고 가정해 보겠습니다.

```cpp
float value = data[tid * 32];  // address LSBs: 0x000, 0x080, 0x100 ...
// recall: floats are 4 bytes wide
```

위 다이어그램의 오른쪽에서 볼 수 있듯이, 이 경우 모든 스레드의 접근이 동일한 뱅크(Bank 0)로 몰리게 됩니다. 따라서 모든 접근이 직렬화되어 처리되어야 하므로 지연 시간이 수십 사이클 수준에서 수백 사이클 수준으로 약 32배나 급증합니다. 이러한 뱅크 충돌은 [공유 메모리](/gpu-glossary/device-software/shared-memory) 배열을 전치(Transpose)하거나 패딩(Padding)을 추가하는 방법으로 해결할 수 있습니다. 뱅크 충돌을 해결하는 다양한 기법에 대한 자세한 내용은 [GTC 2024 발표: CUDA 프로그래밍 및 성능 최적화 입문(Introduction to CUDA Programming and Performance Optimization)](https://www.nvidia.com/en-us/on-demand/session/gtc24-s62191/)을 참고하시기 바랍니다.

여러 [스레드](/gpu-glossary/device-software/thread)가 동일한 뱅크 내의 동일한 주소(즉, 완전히 동일한 데이터)에 동시에 접근하는 경우에는 데이터가 멀티캐스트 또는 브로드캐스트되므로 충돌이 발생하지 않습니다.
