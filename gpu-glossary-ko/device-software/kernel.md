---
title: CUDA 커널이란 무엇인가?
---

![단일 커널 실행(launch)은 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)의 [스레드 블록 그리드](/gpu-glossary/device-software/thread-block-grid)에 해당합니다. NVIDIA의 [CUDA Refresher: The CUDA Programming Model](https://developer.nvidia.com/blog/cuda-refresher-cuda-programming-model/) 및 [CUDA C++ 프로그래밍 가이드](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#programming-model) 다이어그램을 바탕으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-cuda-programming-model.svg)

커널(Kernel)은 프로그래머가 주로 작성하고 조합하는 [CUDA](/gpu-glossary/device-software/cuda-programming-model) 코드의 기본 단위로, CPU를 대상으로 하는 프로그래밍 언어의 프로시저(procedure)나 함수(function)와 유사합니다.

일반적인 프로시저와 달리 커널은 한 번 호출(실행 또는 론칭, launch)되고 한 번 반환되지만, 다수의 [스레드](/gpu-glossary/device-software/thread)에 의해 각 스레드마다 한 번씩 수없이 많이 실행됩니다. 이러한 실행은 일반적으로 동시적(실행 순서가 비결정적임)이며 병렬적(서로 다른 실행 유닛에서 동시에 일어남)으로 이루어집니다.

커널을 실행하는 모든 스레드의 집합은 커널 그리드, 즉 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)의 [스레드 계층 구조](/gpu-glossary/device-software/thread-hierarchy)에서 최상위 수준인 [스레드 블록 그리드](/gpu-glossary/device-software/thread-block-grid)로 구성됩니다. 커널 그리드는 여러 [스트리밍 다중처리기(SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)에 걸쳐 실행되므로 GPU 전체 규모에서 작동합니다. 이에 대응하는 [메모리 계층 구조](/gpu-glossary/device-software/memory-hierarchy)의 수준은 [전역 메모리](/gpu-glossary/device-software/global-memory)입니다.

[CUDA C++](/gpu-glossary/host-software/cuda-c)에서 커널은 호스트에 의해 호출될 때 디바이스의 [전역 메모리](/gpu-glossary/device-software/global-memory)를 가리키는 포인터를 전달받으며, 반환값은 없습니다(void). 즉, 메모리의 값을 직접 수정하는 방식으로 결과를 도출합니다.

CUDA 커널 프로그래밍의 감을 잡기 위해 CUDA 커널의 "Hello World"라 할 수 있는 두 정방행렬 `A`와 `B`의 행렬 곱셈 구현 두 가지를 살펴보겠습니다. 이 두 가지 구현은 교과서적인 행렬 곱셈 알고리즘을 [스레드 계층 구조](/gpu-glossary/device-software/thread-hierarchy)와 [메모리 계층 구조](/gpu-glossary/device-software/memory-hierarchy)에 매핑하는 방식에서 차이가 납니다.

가장 단순한 구현은 [Programming Massively Parallel Processors](https://www.amazon.com/dp/0323912311)(제4판, 그림 3.11)의 첫 번째 행렬 곱 커널에서 착안한 방식입니다. 여기서는 각 [스레드](/gpu-glossary/device-software/thread)가 출력 행렬의 원소 하나를 계산하는 모든 작업을 전담합니다. 즉, `A`의 특정 행(`row`)과 `B`의 특정 열(`col`) 원소를 차례대로 [레지스터](/gpu-glossary/device-software/registers)에 로드하고, 두 원소를 곱한 뒤 그 결과를 누적하여 최종 합을 다시 [전역 메모리](/gpu-glossary/device-software/global-memory)에 저장합니다.

```cpp
__global__ void mm(float* A, float* B, float* C, int N) {
    int row = blockIdx.y * blockDim.y + threadIdx.y;
    int col = blockIdx.x * blockDim.x + threadIdx.x;

    if (row < N && col < N) {
        float sum = 0.0f;
        for (int k = 0; k < N; k++) {
            sum += A[row * N + k] * B[k * N + col];
        }
        C[row * N + col] = sum;
    }
}
```

이 커널에서 각 [스레드](/gpu-glossary/device-software/thread)는 [전역 메모리](/gpu-glossary/device-software/global-memory)에서 데이터를 1회 읽을 때마다 단 1회의 부동소수점 연산(FLOP)을 수행합니다(`A`에서 1회 로드, `B`에서 1회 로드하여 곱셈 1회와 덧셈 1회를 수행). 그러나 이러한 방식으로는 [GPU 성능을 온전히 활용](https://modal.com/blog/gpu-utilization-guide)할 수 없습니다. FLOPs/s로 측정되는 [CUDA 코어](/gpu-glossary/device-hardware/cuda-core)의 [연산 대역폭](/gpu-glossary/perf/arithmetic-bandwidth)이 [GPU RAM](/gpu-glossary/device-hardware/gpu-ram)과 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor) 사이의 [메모리 대역폭](/gpu-glossary/perf/memory-bandwidth)보다 훨씬 높기 때문입니다.

알고리즘의 작업을 [스레드 계층 구조](/gpu-glossary/device-software/thread-hierarchy)와 [메모리 계층 구조](/gpu-glossary/device-software/memory-hierarchy)에 더 정교하게 매핑하면 [메모리 연산 대비 부동소수점 연산의 비율(연산 집약도)](/gpu-glossary/perf/arithmetic-intensity)을 높일 수 있습니다. 아래의 "타일링(tiled)" 행렬 곱 커널은 [Programming Massively Parallel Processors](https://www.amazon.com/dp/0323912311)(제4판, 그림 5.9)에서 영감을 얻은 코드로, `A`와 `B`의 부분 행렬 로드는 [공유 메모리](/gpu-glossary/device-software/shared-memory)에, `C`의 부분 행렬 연산은 [스레드 블록](/gpu-glossary/device-software/thread-block)에 각각 매핑합니다.

```cpp
#define TILE_WIDTH 16

__global__ void mm(float* A, float* B, float* C, int N) {

    // declare variables in shared memory ("smem")
    __shared__ float As[TILE_WIDTH][TILE_WIDTH];
    __shared__ float Bs[TILE_WIDTH][TILE_WIDTH];

    int row = blockIdx.y * TILE_WIDTH + threadIdx.y;
    int col = blockIdx.x * TILE_WIDTH + threadIdx.x;

    float c_output = 0;
    for (int m = 0; m < N/TILE_WIDTH; ++m) {

        // each thread loads one element of A and one of B from global memory into smem
        As[threadIdx.y][threadIdx.x] = A[row * N + (m * TILE_WIDTH + threadIdx.x)];
        Bs[threadIdx.y][threadIdx.x] = B[(m * TILE_WIDTH + threadIdx.y) * N + col];

        // we wait until all threads in the 16x16 block are done loading into smem
        // so that it contains two 16x16 tiles
        __syncthreads();

        // then we loop over the inner dimension,
        // performing 16 multiplies and 16 adds per pair of loads from global memory
        for (int k = 0; k < TILE_WIDTH; ++k) {
            c_output += As[threadIdx.y][k] * Bs[k][threadIdx.x];
        }
        // wait for all threads to finish computing
        // before any start loading the next tile into smem
        __syncthreads();
    }
    C[row * N + col] = c_output;
}
```

원소 두 개를 로드하는 외부 루프가 한 번 반복될 때마다 스레드는 내부 루프를 16회 반복하며 곱셈과 덧셈을 수행하므로, 전역 메모리 읽기 1회당 16 FLOP의 연산을 수행하게 됩니다.

이 역시 행렬 곱셈을 완벽하게 최적화한 커널과는 거리가 멉니다. [Anthropic의 Si Boehm이 작성한 작업 일지](https://siboehm.com/articles/22/CUDA-MMM)에서는 메모리 읽기 대비 FLOP 비율을 더욱 끌어올리고 알고리즘을 하드웨어에 한층 긴밀하게 매핑하는 최적화 기법들을 단계별로 소개합니다. 위에서 살펴본 커널들은 해당 글의 커널 1 및 커널 3과 유사하며, 해당 작업 일지에서는 총 10개의 커널을 다룹니다.

해당 작업 일지와 이 글에서는 [CUDA 코어](/gpu-glossary/device-hardware/cuda-core)에서 실행되는 커널 작성만을 다루었습니다. 진정으로 가장 빠른 행렬 곱셈 커널은 훨씬 더 높은 [연산 대역폭](/gpu-glossary/perf/arithmetic-bandwidth)을 제공하는 [텐서 코어](/gpu-glossary/device-hardware/tensor-core)에서 실행됩니다.
