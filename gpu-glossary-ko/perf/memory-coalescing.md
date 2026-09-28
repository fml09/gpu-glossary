---
title: 메모리 병합이란 무엇인가?
---

메모리 병합(Memory Coalescing)은 여러 논리적(logical) 메모리 읽기 요청을 단 한 번의 물리적(physical) 메모리 접근으로 묶어서 처리함으로써 [메모리 대역폭](/gpu-glossary/perf/memory-bandwidth) 활용률을 극대화하는 하드웨어 최적화 기법입니다.

메모리 병합은 [전역 메모리(Global Memory)](/gpu-glossary/device-software/global-memory)에 접근할 때 일어납니다. [공유 메모리(Shared Memory)](/gpu-glossary/device-software/shared-memory)의 효율적인 접근 방식에 관해서는 [뱅크 충돌(Bank Conflict)](/gpu-glossary/perf/bank-conflict) 문서를 참조하시기 바랍니다.

[CUDA](/gpu-glossary/device-hardware/cuda-device-architecture) GPU에서 [전역 메모리](/gpu-glossary/device-software/global-memory)는 GDDR이나 HBM 같은 동적 랜덤 액세스 메모리(DRAM) 기술로 제작된 [GPU RAM](/gpu-glossary/device-hardware/gpu-ram)을 기반으로 동작합니다. 이러한 기술은 높은 [메모리 대역폭](/gpu-glossary/perf/memory-bandwidth)을 제공하지만, CPU RAM에 쓰이는 동급 기술인 DDR5와 비교해도 접근 지연 시간이 상당히 깁니다. DRAM의 접근 지연 시간은 미세한 커패시터가 액세스 라인을 충전하는 속도에 의해 제한되며, 이는 열, 전력 및 크기 제약으로 인해 근본적인 물리적 한계를 갖습니다. 이처럼 긴 지연 시간 때문에 모든 논리적 메모리 접근을 개별적인 물리적 접근으로 처리한다면 GPU의 [메모리 대역폭](/gpu-glossary/perf/memory-bandwidth)을 온전히 활용할 수 없게 됩니다.

메모리 병합은 DRAM 기술의 내부 구조를 활용하여 특정 접근 패턴에서 대역폭을 최대로 활용할 수 있도록 지원합니다. DRAM 주소에 접근할 때마다 연속된 여러 주소의 데이터가 단일 클록 내에서 병렬로 한꺼번에 인출됩니다. 이에 대한 조금 더 자세한 내용은 [Programming Massively Parallel Processors 제4판](https://www.amazon.com/dp/0323912311) 6.1절을, 포괄적인 세부 사항은 Ulrich Drepper의 훌륭한 아티클인 [모든 프로그래머가 메모리에 대해 알아야 할 것(What Every Programmer Should Know About Memory)](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf)을 참고하시기 바랍니다. 이렇게 연속된 메모리 위치를 한 번에 접근하고 전송하는 단위를 *DRAM 버스트(DRAM burst)*라고 부릅니다. 동시에 발생하는 여러 논리적 접근 요청이 단 하나의 물리적 버스트로 처리될 때, 해당 메모리 접근을 '병합(coalesced)되었다'고 표현합니다. 여기서 물리적 접근은 메모리 트랜잭션(memory transaction)의 일부이며, 다른 메모리 병합 관련 문서에서도 흔히 사용되는 용어입니다.

CPU에서도 버스트를 캐시 라인에 매핑하여 접근 효율을 높이는 유사한 방식이 사용됩니다. 다만 CPU에서는 하드웨어 캐시가 이를 자동으로 처리하는 반면, GPU 프로그래밍에서는 프로그래머가 직접 이러한 접근을 설계하고 관리해야 한다는 점이 다릅니다.

하지만 이는 생각보다 어렵지 않습니다. DRAM 버스트 구조가 [CUDA PTX](/gpu-glossary/device-software/parallel-thread-execution)의 단일 명령어 다중 스레드(SIMT) 실행 모델과 완벽하게 맞아떨어지기 때문입니다. 즉, 일반적인 실행 환경에서는 한 [워프](/gpu-glossary/device-software/warp) 내의 모든 [스레드](/gpu-glossary/device-software/thread)가 동일한 시점에 동일한 명령어를 실행합니다. 덕분에 [CUDA](/gpu-glossary/device-software/cuda-programming-model) 프로그래머는 병합 접근이 가능하도록 코드를 쉽게 작성할 수 있고, 메모리 관리 하드웨어 또한 병합 가능한 접근을 간단히 감지할 수 있습니다. 일반적으로 단일 버스트는 128바이트를 처리할 수 있는데, 이는 우연이 아니며 한 [워프](/gpu-glossary/device-software/warp)에 속한 32개 [스레드](/gpu-glossary/device-software/thread)가 각각 32비트(4바이트) 부동소수점 데이터를 하나씩 로드하기에 정확히 일치하는 크기입니다.

메모리 병합이 성능에 미치는 영향을 확인하기 위해, 접근하는 요소 간의 간격인 보폭(`stride`)을 변경해가며 배열에서 값을 읽어오는 다음 [커널](/gpu-glossary/device-software/kernel)을 살펴보겠습니다. 보폭이 커질수록 각 [워프](/gpu-glossary/device-software/warp)가 발행한 읽기 명령을 처리하는 데 필요한 DRAM 버스트 수가 늘어나 논리적 접근당 물리적 접근 횟수가 증가하고, 그 결과 메모리 처리량이 급격히 감소합니다.

```cpp
__global__ void strided_read_kernel(const float* __restrict__ in,
                                    float* __restrict__ out,
                                    size_t N, int stride)
{
    const size_t t  = blockIdx.x * blockDim.x + threadIdx.x;
    const size_t T  = gridDim.x * (size_t)blockDim.x;

    float acc = 0.f;

    for (size_t j = (size_t)t * (size_t)stride; j < N; j += (size_t)T * (size_t)stride) {
        // across a warp, addresses differ by (stride * sizeof(float))
        float v = in[j]; // perfectly coalesced for stride == 1
        acc = acc * 1.000000119f + v;  // force compiler to keep the load
    }

    // do one write per thread (negligible vs reads)
    if (t < N) out[t] = acc;
}
```

이 커널을 Godbolt의 마이크로 벤치마크([여기](https://godbolt.org/z/KbWhEWjcb)에서 직접 재현 가능)로 실행해 보면 보폭과 처리량 사이의 상관관계를 명확히 확인할 수 있습니다.

```
# Device: Tesla T4 (SM 75)
# N = 67108864 floats (256.0 MB), iters = 10
stride        GB/s
    1       206.0
    2       130.5
    4        68.8
    8        33.8
   16        16.8
   32        15.2
   64        13.6
  128        11.2
```

보폭을 2로 늘리면 각 [워프](/gpu-glossary/device-software/warp)의 요청을 처리하는 데 필요한 DRAM 버스트 수가 2배로 증가하여 처리량이 절반으로 줄어듭니다. 보폭을 4로 다시 2배 늘리면 처리량은 또다시 절반으로 감소합니다. 보폭이 16이 되어 처리량이 16분의 1로 떨어진 이후부터는 패턴의 양상이 달라집니다. 이때부터는 데이터의 지역성 저하로 인해 온디바이스 TLB 미스 등 메모리 서브시스템의 다른 구성 요소들의 영향이 두드러지면서 성능 저하 추세가 완만해집니다.

전역 메모리 접근 모범 사례에 대한 추가 정보는 NVIDIA 개발자 블로그의 [CUDA C/C++ 커널에서 전역 메모리에 효율적으로 접근하는 방법(How to Access Global Memory Efficiently in CUDA C/C++ Kernels)](https://developer.nvidia.com/blog/how-access-global-memory-efficiently-cuda-c-kernels/) 글을 참고하시기 바랍니다.
