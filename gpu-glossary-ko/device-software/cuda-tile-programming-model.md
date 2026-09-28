---
title: CUDA 타일 프로그래밍 모델이란 무엇인가?
---

CUDA 타일 프로그래밍 모델(CUDA Tile programming model)은 NVIDIA GPU를 대상으로 하는 타일 기반 프로그래밍 모델입니다.

전통적인 [CUDA 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)은 포인터를 전달받아 해당 포인터가 가리키는 메모리를 동시 실행을 통해 수정하는 사용자 프로그램에 [스레드 계층 구조](/gpu-glossary/device-software/thread-hierarchy)와 [메모리 계층 구조](/gpu-glossary/device-software/memory-hierarchy)를 노출합니다. 여러 [스레드](/gpu-glossary/device-software/thread)에 동일한 명령어가 병렬로 발행되므로, 이 프로그래밍 모델은 '단일 명령어 다중 스레드(SIMT, Single-Instruction, Multiple Thread)' 프로그래밍 모델에 해당합니다. 이는 [CUDA C/C++](/gpu-glossary/host-software/cuda-c)와 CUDA 타일 도입 이전 NVIDIA GPU를 대상으로 했던 [PTX](/gpu-glossary/device-software/parallel-thread-execution) 중간 표현(IR)에서 사용해 온 프로그래밍 모델입니다.

이 프로그래밍 모델은 'CUDA'의 'U'가 의미하듯 [통합된(unified) 하드웨어 기반](/gpu-glossary/device-hardware/cuda-device-architecture)을 전제로 정의되었습니다. 즉, CUDA 이전 그래픽스 프로그래밍에서 흔히 볼 수 있었던 이종(heterogeneous) 코어로 구성된 특화 디바이스 방식 대신, 균일한 [CUDA 코어](/gpu-glossary/device-hardware/cuda-core)를 갖춘 균일한 [스트리밍 다중처리기(SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)가 대부분의 연산을 수행하도록 설계되었습니다.

그러나 대다수의 [연산 대역폭(Arithmetic Bandwidth)](/gpu-glossary/perf/arithmetic-bandwidth)이 [텐서 코어(Tensor Core)](/gpu-glossary/device-hardware/tensor-core)에 집중되어 있는 최신 [SM 아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture) 기반 GPU에는 이러한 전통적인 모델이 잘 들어맞지 않습니다. 텐서 코어는 오직 행렬 곱셈 연산만을 수행할 수 있으며, 하드웨어의 나머지 부분을 프로그래밍할 때 사용하는 [워프](/gpu-glossary/device-software/warp) 수준의 비동기성이 아니라 [스레드](/gpu-glossary/device-software/thread) 수준의 명령어와 비동기성을 통해 프로그래밍해야 하기 때문입니다.

CUDA 타일 프로그래밍 모델에서는 프로그램을 _타일 커널(tile-kernel)_ 수준에서 표현합니다. 타일 커널은 단일 실행 스레드 역할을 하는 _타일 블록(tile block)_ 그리드 전반에서 동시에 실행되는 프로그램 인스턴스입니다. 타일 커널은 일반적인 경우(happy path) 포인터와 함께 배열의 전체 크기(shape) 및 접근 패턴(stride) 정보를 결합한 _구조화된 포인터(structured pointer)_를 대상으로 연산합니다. 이는 `Layout`과 `Tensor`를 다루는 [CuTe](/gpu-glossary/host-software/cute)의 타입 시스템과 매우 유사합니다.

CUDA C/C++ 및 PTX IR 기반의 전통적인 'CUDA SIMT'와 마찬가지로, 이 프로그래밍 모델 역시 고급 언어와 중간 표현(여기서는 [Tile IR](https://docs.nvidia.com/cuda/tile-ir/latest/sections/prog_model.html)) 간에 공유됩니다.

이 글을 작성하는 시점 기준으로 CUDA 타일 프로그래밍 모델은 새롭게 등장한 모델이며, 기존의 'CUDA SIMT' 프로그래밍 모델을 어느 정도까지 대체할지는 아직 명확하지 않습니다. 현재 CUDA 타일 프로그래밍 모델은 [cuTile Python](https://docs.nvidia.com/cuda/cutile-python/quickstart.html)을 통해 사용할 수 있으며, 실험적 형태이기는 하지만 [cuTile BASIC](/gpu-glossary/host-software/cutile-basic)과 [cuTile Rust](https://github.com/nvlabs/cutile-rs)를 통해서도 활용할 수 있습니다.
