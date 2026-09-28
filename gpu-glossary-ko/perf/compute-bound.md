---
title: 연산 제약이란 무엇인가?
---

연산 제약(Compute-bound) 상태인 [커널(Kernel)](/gpu-glossary/device-software/kernel)은 [CUDA 코어(CUDA Core)](/gpu-glossary/device-hardware/cuda-core) 또는 [텐서 코어(Tensor Core)](/gpu-glossary/device-hardware/tensor-core)의 [연산 대역폭(Arithmetic Bandwidth)](/gpu-glossary/perf/arithmetic-bandwidth)에 의해 성능이 제한됩니다.

![위 [루프라인 다이어그램](/gpu-glossary/perf/roofline-model)에서 파란색 수평선 아래에 위치한 [커널](/gpu-glossary/device-software/kernel)은 연산 제약 상태에 해당합니다. Williams, Waterman, Patterson (2008)의 논문 다이어그램을 참고하여 작성하였습니다.](https://modal-cdn.com/gpu-glossary/light-roofline-model.svg)

연산 제약 커널은 높은 [연산 강도(Arithmetic Intensity)](/gpu-glossary/perf/arithmetic-intensity)(로드하거나 저장하는 메모리 1바이트당 수행하는 산술 연산의 수가 많음)를 갖는 것이 특징입니다. 연산 제약 상태의 커널에서는 [산술 연산 파이프라인 활용률](/gpu-glossary/perf/pipe-utilization)이 주된 한계 요인으로 작용합니다.

엄밀히 말하면 연산 제약 여부는 [루프라인 모델](/gpu-glossary/perf/roofline-model)의 일부로서 개별 [커널](/gpu-glossary/device-software/kernel) 단위로 정의됩니다. 하지만 시각을 조금 넓히면 일반적인 워크로드를 구성하는 여러 [커널](/gpu-glossary/device-software/kernel) 전체를 아우르는 개념으로도 확장하여 이해할 수 있습니다.

대규모 확산 모델(Diffusion Model)의 추론 워크로드는 일반적으로 연산 제약 특성을 보입니다. 또한 현대 대규모 언어 모델(LLM)의 추론 워크로드에서는 각 가중치를 [공유 메모리](/gpu-glossary/device-software/shared-memory)에 한 번만 로드해 두고 여러 토큰에 걸쳐 재사용할 수 있는 배치 단위의 프롬프트 처리(Prefill) 단계에서 연산 제약이 자주 발생합니다.

[kipperrii](https://twitter.com/kipperrii)의 [트랜스포머 추론 연산(Transformer inference arithmetic)](https://kipp.ly/transformer-inference-arithmetic) 프레임워크를 참고하여, 연산 제약 상태의 트랜스포머 언어 모델 추론에서 토큰 간 최소 지연 시간(inter-token latency 또는 토큰당 생성 시간)을 간단히 추정해 보겠습니다. 16비트 정밀도로 저장되어 총 1 TB 크기를 갖는 500B(5,000억 개) 파라미터 모델을 가정합니다. 이 모델은 배치 요소당 약 1조(1 Trillion) 회의 부동소수점 연산(파라미터당 1회의 곱셈 및 1회의 누적 덧셈)을 수행합니다. 16비트 행렬 연산 대역폭이 1 PFLOPS인 GPU에서 실행한다고 가정하면, 연산 제약 상태일 때 배치 요소당 토큰 간 최소 지연 시간은 1밀리초(ms)가 됩니다.

이 GPU가 배치 크기 1에서 연산 제약 상태가 되려면 1 PB/s의 [메모리 대역폭](/gpu-glossary/perf/memory-bandwidth)이 필요합니다(1ms 안에 1 TB의 전체 가중치를 로드해야 하기 때문입니다). 그러나 현대 하드웨어의 [메모리 대역폭](/gpu-glossary/perf/memory-bandwidth)은 TB/s 수준에 머물러 있으므로, 실행 상태가 연산 제약 영역에 도달하기에 충분한 [연산 강도](/gpu-glossary/perf/arithmetic-intensity)를 확보하려면 수백 개의 입력을 묶은 배치가 필수적입니다.

LLM 추론에 관한 자세한 내용은 [LLM 엔지니어 연감(LLM Engineer's Almanac)](https://modal.com/llm-almanac/summary)을 참조하시기 바랍니다.
