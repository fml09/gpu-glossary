---
title: cuBLAS란 무엇인가?
---

cuBLAS(CUDA Basic Linear Algebra Subroutines)는 [BLAS(Basic Linear Algebra Subprograms)](https://en.wikipedia.org/wiki/Basic_Linear_Algebra_Subprograms) 표준 규격을 NVIDIA가 고성능으로 구현한 수치 연산 라이브러리입니다. 자주 사용되는 주요 선형대수 연산을 극도로 최적화된 [커널](/gpu-glossary/device-software/kernel) 형태로 제공하는 독점 소프트웨어입니다.

개발자는 행렬 곱셈과 같은 기본적인 연산을 직접 작성하고 최적화할 필요 없이 호스트 코드에서 cuBLAS 함수를 간편하게 호출할 수 있습니다. 이 라이브러리는 FP32, FP16 등 다양한 데이터 타입, 행렬 크기, [스트리밍 다중처리기(SM) 아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture)에 맞추어 미세 조정된 방대한 커널 풀을 갖추고 있습니다. 런타임에 cuBLAS는 내부 휴리스틱을 사용하여 가장 처리 성능이 우수한 커널과 최적의 실행 파라미터를 자동으로 선택합니다. 그 결과 cuBLAS는 NVIDIA GPU 기반 [고성능 수치 연산](/gpu-glossary/perf)의 표준 토대로 자리 잡았으며, [cuDNN](/gpu-glossary/host-software/cudnn) 같은 특화 [커널](/gpu-glossary/device-software/kernel) 라이브러리와 더불어 PyTorch를 비롯한 주요 딥러닝 프레임워크의 핵심 연산을 가속하는 데 폭넓게 활용됩니다.

cuBLAS를 사용할 때 가장 흔히 겪는 혼선은 행렬 데이터의 배치 순서(Layout)에서 발생합니다. 역사적인 배경과 Fortran으로 작성되었던 기존 BLAS 표준과의 호환성을 지키기 위해 cuBLAS는 행렬이 [열 우선 순서(Column-major order)](https://en.wikipedia.org/wiki/Row-_and_column-major_order)로 메모리에 저장되어 있다고 가정합니다. 이는 C, C++, 파이썬 등에서 보편적으로 쓰이는 행 우선 순서(Row-major order)와 정반대 방식입니다. 아울러 BLAS 함수를 호출할 때는 연산 크기(`M`, `N`, `K`)뿐만 아니라 메모리에서 다음 열이 시작하는 간격인 리딩 디멘션(Leading Dimension, 예: `lda`)을 명시해야 합니다. 리딩 디멘션은 인접한 두 열 사이의 메모리 보폭(Stride)을 나타냅니다. 할당된 행렬 전체를 사용할 경우 리딩 디멘션은 행 개수와 동일합니다. 하지만 더 큰 상위 행렬에서 잘라낸 부분 행렬(Submatrix)을 다루는 경우라면 원본 상위 행렬의 행 개수가 리딩 디멘션이 됩니다.

다행히 GEMM처럼 연산 집약적인 커널에서는 행렬을 행 우선에서 열 우선으로 재정렬할 필요가 없습니다. 행렬 곱에서 `C = A @ B`가 성립할 때 `C^T = B^T @ A^T`라는 수학적 성질을 활용할 수 있기 때문입니다. 핵심 원리는 행 우선 순서로 저장된 행렬의 실제 메모리 배치가 해당 행렬의 전치 행렬을 열 우선 순서로 저장했을 때의 배치와 물리적으로 완벽히 동일하다는 점입니다. 따라서 행 우선 행렬 `A`와 `B`를 전달할 때 호출 인자의 순서와 행렬 크기를 맞바꾸어 cuBLAS에 전달하면, cuBLAS는 `C^T`를 계산하여 열 우선 순서로 메모리에 기록합니다. 이렇게 생성된 결과 메모리 블록을 행 우선 관점에서 읽으면 우리가 원하는 행렬 `C`를 정확히 얻을 수 있습니다. 이 기법을 구현한 예시는 다음과 같습니다.

```cpp
#include <cublas_v2.h>

// performs single-precision C = alpha * A @ B + beta * C
// on row-major matrices using cublasSgemm
void sgemm_row_major(cublasHandle_t handle, int M, int N, int K,
                     const float *alpha,
                     const float *A, const float *B,
                     const float *beta,
                     float *C) {

  // A is M x K (row-major), cuBLAS sees it as A^T (K x M, column-major),
  //   the leading dimension of A^T is K
  // B is K x N (row-major), cuBLAS sees it as B^T (N x K, column-major),
  //   the leading dimension of B^T is N
  // C is M x N (row-major), cuBLAS sees it as C^T (N x M, column-major),
  //   the leading dimension of C^T is N

  // note the swapped A and B, and the swapped M and N
  cublasSgemm(handle, CUBLAS_OP_N, CUBLAS_OP_N,
              N, M, K,
              alpha,
              B, N,  // leading dimension of B^T
              A, K,  // leading dimension of A^T
              beta,
              C, N); // leading dimension of C^T
}
```

이 예제의 전체 실행 코드는 [Godbolt](https://godbolt.org/z/axzYb75ro)에서 직접 실행해 볼 수 있습니다.

`CUBLAS_OP_N` 플래그는 커널이 전달받은 행렬을 추가적인 전치 변환 없이 그대로 연산에 사용하도록 지정합니다.

cuBLAS 라이브러리를 사용하려면 [nvcc](/gpu-glossary/host-software/nvcc)로 컴파일할 때 `-lcublas` 옵션을 주어 라이브러리를 링크해야 합니다. 지원되는 함수들은 `cublas_v2.h` 헤더 파일에 선언되어 있습니다.

cuBLAS에 관한 더 상세한 정보는 [공식 cuBLAS 문서](https://docs.nvidia.com/cuda/cublas/)에서 확인할 수 있습니다.
