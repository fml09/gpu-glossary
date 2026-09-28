---
title: 분기 효율이란 무엇인가?
---

분기 효율(Branch Efficiency)은 조건문을 만났을 때 [워프(Warp)](/gpu-glossary/device-software/warp) 내의 모든 [스레드(Thread)](/gpu-glossary/device-software/thread)가 동일한 실행 경로를 따르는 빈도를 측정하는 지표입니다.

분기 효율은 실행된 전체 분기 명령어 중에서 균일한(uniform) 제어 흐름 결정을 내린 비율로 계산됩니다. 제어 흐름의 균일성은 [워프](/gpu-glossary/device-software/warp) 단위로 측정되므로, 분기 효율이 높다는 것은 [워프 발산(Warp Divergence)](/gpu-glossary/perf/warp-divergence)이 발생하지 않았음을 나타냅니다.

모든 조건문이 분기 효율을 떨어뜨리는 것은 아닙니다. 대부분의 [커널](https://godbolt.org/z/d1PsYYPnW)에 자주 등장하는 다음과 같은 전형적인 "경계 검사(bounds-check)" 코드 조각을 살펴보겠습니다.

```cpp
int idx = blockIdx.x * blockDim.x + threadIdx.x;
    if (idx < n)
```

이러한 코드는 일반적으로 매우 높은 분기 효율을 보입니다. 스레드 인덱스가 `n`의 경계에 걸쳐 있는 단 하나의 [워프](/gpu-glossary/device-software/warp)를 제외하고는, 대다수 워프에 속한 모든 [스레드](/gpu-glossary/device-software/thread)가 해당 조건식에 대해 동일한 참 또는 거짓 값을 갖기 때문입니다.

CPU 역시 분기 동작의 균일성을 중요하게 다루지만, CPU의 주된 관심사는 하드웨어 분기 예측 및 투기적 실행(speculative execution)을 지원하기 위한 시간적 균일성입니다. 즉, 프로그램이 실행되는 동안 특정 분기문이 여러 번 반복해서 실행되면서 CPU 내부 회로에 분기 이력 데이터가 축적될 때 성능이 향상됩니다.

반면 GPU는 공간적 균일성을 중요하게 여깁니다. 즉, 시간상 동일한 시점에 병렬로 실행되면서 서로 다른 데이터에 매핑되는 [워프](/gpu-glossary/device-software/warp) 내 [스레드](/gpu-glossary/device-software/thread)들 사이에서 균일성을 측정하며, 이 스레드들이 모두 균일하게 분기할 때 비로소 성능이 향상됩니다.
