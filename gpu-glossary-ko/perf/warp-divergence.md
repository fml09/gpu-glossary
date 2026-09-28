---
title: 워프 발산이란 무엇인가?
---

워프 발산(Warp Divergence)은 제어 흐름문(조건문)으로 인해 한 [워프(Warp)](/gpu-glossary/device-software/warp) 내의 스레드들이 서로 다른 실행 경로를 따르게 될 때 발생합니다.

예를 들어 다음과 같은 [커널(Kernel)](/gpu-glossary/device-software/kernel)을 살펴보겠습니다.

```cpp
__global__ void divergent_kernel(float* data, int n) {
    int idx = blockIdx.x * blockDim.x + threadIdx.x;
    if (idx < n) {
        if (data[idx] > 0.5f) {
		    // A
            data[idx] = data[idx] * 4.0f;
        } else {
		    // B
            data[idx] = data[idx] + 2.0f;
        }
        data[idx] = data[idx] * data[idx];
    }
}
```

한 [워프](/gpu-glossary/device-software/warp)에 속한 [스레드(Thread)](/gpu-glossary/device-software/thread)들이 데이터 값에 따라 분기하는 조건문을 만났을 때, 각 스레드는 `data[idx]`의 값에 따라 A 블록을 실행하거나 B 블록을 실행해야 합니다. 이러한 데이터 종속성과 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model) 및 [PTX 머신 모델](/gpu-glossary/device-software/parallel-thread-execution)의 구조적 제약으로 인해, 프로그래머나 컴파일러가 워프 내부에서 발생하는 제어 흐름의 분기를 원천적으로 회피할 수는 없습니다.

대신 [워프 스케줄러(Warp Scheduler)](/gpu-glossary/device-hardware/warp-scheduler)는 특정 [스레드](/gpu-glossary/device-software/thread)들이 명령어를 실행하지 않도록 '마스킹(masking)'하는 방식으로 이러한 분기 경로들을 조율합니다. 이는 조건자 [레지스터(Predicate Register)](/gpu-glossary/device-software/registers)를 활용하여 수행됩니다.

실행 흐름을 명확히 이해하기 위해 컴파일된 [SASS](/gpu-glossary/device-software/streaming-assembler) 코드([Godbolt 링크](https://godbolt.org/z/EGWKb5oWr))를 살펴보겠습니다.

```nasm
LDG.E.SYS R4, [R2]                       // L1 load data[idx]
FSETP.GT.AND P0, PT, R4.reuse, 0.5, PT   // L2 set P0 to data[idx] > 0.5
FADD R0, R4, 2                           // L3 store 2 + data[idx] in R0
@P0 FMUL R0, R4, 4                       // L4 in some threads, store 4 * data[idx] in R0
FMUL R5, R0, R0                          // L5 store R0 * R0 in R5
STG.E.SYS [R2], R5                       // L6 store R5 in data[idx]
```

데이터를 `R4`로 로드(`L1`)한 후, [워프](/gpu-glossary/device-software/warp) 내 32개의 모든 [스레드](/gpu-glossary/device-software/thread)가 동시에 `FSETP.GT.AND`를 실행(`L2`)하며, 각 스레드는 `R4`에 담긴 값에 따라 고유한 `P0` 플래그 값을 부여받습니다. 여기서 [컴파일러(nvcc)](/gpu-glossary/host-software/nvcc)의 기발한 최적화가 드러납니다. `L3`에서 *모든* [스레드](/gpu-glossary/device-software/thread)가 조건 검사 없이 블록 B(2를 더하는 코드)를 먼저 실행하여 결과를 `R0`에 기록합니다. 그런 다음 `P0`가 참인 스레드만 블록 A의 코드(`L4`, 4를 곱하는 코드)를 실행하여 `L3`에서 `R0`에 기록되었던 값을 덮어씁니다. 이 `L4` 명령어 실행 시점에 해당 워프는 '발산(divergent)' 상태가 됩니다. 이후 `L5`에 도달하면 모든 스레드가 다시 동일한 코드를 실행합니다. [워프 스케줄러](/gpu-glossary/device-hardware/warp-scheduler)가 동일한 클록 사이클에 모든 스레드에 같은 명령어를 발행하여 정렬 상태를 회복하면, 워프가 '수렴(converged)'했다고 표현합니다.

이 방식은 `L3`과 `L4` 두 라인 모두에 조건자를 적용하는 단순한 분기 방식보다 효율적입니다. 경험적으로 비용이 저렴하고 풍부한 [CUDA 코어](/gpu-glossary/device-hardware/cuda-core)의 연산 능력을 다소 소모하더라도, 상대적으로 비용이 많이 드는 복잡한 제어 흐름을 피하는 쪽을 선택한 것입니다. GPU 프로그래밍에서는 단순한 조건자 분기일지라도 제어 복잡성을 추가하는 것보다(`L4`를 실행하는 스레드 입장에서 불필요한 `FADD`를 수행하더라도) 약간의 연산량을 낭비하는 편이 전반적인 성능에 더 유리한 경우가 흔합니다.

컴파일러가 워프 발산을 적극적으로 회피하려는 이유 중 하나는 볼타(Volta) 이전의 초기 GPU 아키텍처에서 발산된 워프가 항상 완전히 직렬화되어 실행되었기 때문입니다. 최신 GPU는 독립 스레드 스케줄링(Independent Thread Scheduling)을 지원하므로 직렬화에 따른 페널티를 과거만큼 전적으로 겪지는 않지만, 워프 발산은 여전히 하드웨어 효율성을 떨어뜨리는 주요 요인입니다.
