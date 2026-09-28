---
title: CUDA 그래프란 무엇인가?
---

CUDA 그래프(CUDA Graph)는 호스트가 디바이스로 한꺼번에 전달하여 실행할 수 있도록 여러 [커널](/gpu-glossary/device-software/kernel) 실행 및 관련 작업들을 노드와 엣지 형태의 그래프로 정의한 연산 실행 모델입니다.

CUDA 그래프를 사용하는 주된 목적은 짧은 시간 동안 수많은 [커널](/gpu-glossary/device-software/kernel)을 호스트에서 식별하고 설정하여 디바이스에 제출할 때 발생하는 호스트 측 [오버헤드](/gpu-glossary/perf/overhead)를 대폭 줄이는 것입니다. 개별 커널을 디바이스에 발행하는 데에는 매번 수 마이크로초 단위의 시간이 소요되므로, 수 밀리초 이내에 수백 개의 커널을 연속해서 호출해야 하는 환경에서는 이러한 론치 오버헤드가 심각한 병목이 될 수 있습니다. 이는 [저지연 LLM 추론](https://modal.com/docs/guide/high-performance-llm-inference) 환경에서 특히 두드러지게 나타납니다.

CUDA 그래프는 일반적으로 [CUDA 런타임](/gpu-glossary/host-software/cuda-runtime-api)의 스트림 캡처(Stream Capture) API를 활용해 생성합니다. 단일 CUDA 스트림에서 일어나는 모든 연산 작업을 캡처하여 그래프 구조로 저장한 뒤, 필요할 때마다 다음과 같이 손쉽게 재실행(Replay)할 수 있습니다.

```cpp
// 캡처 (Capture)
cudaStreamBeginCapture(stream);
kernelGemm<<<{32, 20},64,19200,stream>>>(a, b, c);
kernelEpilogue<<<{256,2},{8,32},0,stream>>>(c, c);
cudaStreamEndCapture(stream, &graph);

// 실행 (Launch)
cudaGraphInstantiate(&graphExec, graph, flags);
cudaGraphLaunch(graphExec, stream);
```

CUDA 그래프를 다루는 [CUDA 런타임](/gpu-glossary/host-software/cuda-runtime-api) 인터페이스에 관한 자세한 내용은 [NVIDIA 공식 프로그래밍 가이드](https://docs.nvidia.com/cuda/cuda-programming-guide/04-special-topics/cuda-graphs.html)에서 확인할 수 있습니다.

이 기능은 PyTorch에서도 `torch.cuda.graph` 컨텍스트 매니저 등의 형태로 제공되므로, 딥러닝 신경망 모델의 훈련 및 추론 워크플로에서 CUDA 그래프를 손쉽게 캡처하고 활용할 수 있습니다.

아래는 B200 GPU에서 `torch.Linear` 계층 연산을 수행하는 동안 캡처한 CUDA 그래프의 실제 구조 예시입니다.

```
┌─────────────────────────────────────────────────────────────────────────┐
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                          NODE 0: KERNEL                           │  │
│  ├───────────────────────────────────────────────────────────────────┤  │
│  │  ID:         0 (topoId: 1)                                        │  │
│  │  Kernel:     cutlass3x_sm100_simt_sgemm_f32_f32_f32_f32_f32_      │  │
│  │              64x32x16_1x1x1_3_tnn_align1_bias_f32_relu            │  │
│  │              <<<{32,20},64,19200>>>                               │  │
│  │  Node handle: 0x0000564604539520                                  │  │
│  │  Func handle: 0x0000564603AFCC00                                  │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                              │                                          │
│                              │                                          │
│                              ▼                                          │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                          NODE 1: KERNEL                           │  │
│  ├───────────────────────────────────────────────────────────────────┤  │
│  │  ID:         1 (topoId: 0)                                        │  │
│  │  Kernel:     _ZN8cublasLt8epilogue4impl12globalKernelILi8E...     │  │
│  │              <<<{256,2},{8,32},0>>>                               │  │
│  │  Node handle: 0x0000564604539C88                                  │  │
│  │  Func handle: 0x00005646044770F0                                  │  │
│  └───────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

위 구조에서 각 [커널](/gpu-glossary/device-software/kernel)은 `0x564603AFCC00`과 같은 실제 메모리 주소 포인터로 식별됩니다. 입력 및 출력 버퍼 역시 특정 디바이스 메모리 포인터에 직접 연결됩니다. 이러한 물리적 하드웨어 자원 의존성 때문에 일반적인 파일 저장 방식으로는 CUDA 그래프를 직렬화(Serialization)하거나 다른 인스턴스로 이식하기 어렵습니다. 예외적으로 [호스트 및 디바이스 메모리 전체를 스냅샷으로 체크포인트하고 복원하는 특수 환경](https://modal.com/docs/guide/memory-snapshots)에서만 제한적으로 이식이 가능합니다.
