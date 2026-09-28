---
title: 메모리 제약이란 무엇인가?
---

메모리 제약(Memory-bound) 상태인 [커널(Kernel)](/gpu-glossary/device-software/kernel)은 GPU의 [메모리 대역폭(Memory Bandwidth)](/gpu-glossary/perf/memory-bandwidth)에 의해 성능이 제한됩니다.

![위와 같은 루프라인 다이어그램은 프로그램의 성능이 연산 능력, 메모리 대역폭, 또는 다른 요인 중 무엇에 의해 병목 현상을 겪고 있는지 식별하는 데 도움을 줍니다. Williams, Waterman, Patterson (2008)의 논문 다이어그램을 참고하여 작성하였습니다.](https://modal-cdn.com/gpu-glossary/light-roofline-model.svg)

구체적으로 이러한 커널은 [GPU RAM](/gpu-glossary/device-hardware/gpu-ram)과 [스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)의 [로컬 캐시](/gpu-glossary/device-hardware/l1-data-cache) 사이의 [대역폭](/gpu-glossary/perf/memory-bandwidth)에 의해 제한됩니다. GPU 성능 최적화의 주된 관심사가 되는 문제들은 대체로 [작업 집합 크기(Working Set Size)](https://en.wikipedia.org/wiki/Working_set_size)가 [메모리 계층 구조](/gpu-glossary/device-software/memory-hierarchy)의 상위 캐시 용량보다 훨씬 크기 때문입니다.

메모리 제약 커널은 하드웨어 [루프라인 모델](/gpu-glossary/perf/roofline-model)의 변곡점에 비해 상대적으로 낮은 [연산 강도(Arithmetic Intensity)](/gpu-glossary/perf/arithmetic-intensity)(이동한 1바이트당 연산 횟수가 적음)를 가집니다.

엄밀히 말하면 메모리 제약 여부는 [루프라인 모델](/gpu-glossary/perf/roofline-model)의 일부로서 개별 [커널](/gpu-glossary/device-software/kernel) 단위로 정의됩니다. 하지만 시각을 조금 넓히면 일반적인 워크로드를 구성하는 여러 [커널](/gpu-glossary/device-software/kernel) 전체를 아우르는 개념으로도 확장하여 이해할 수 있습니다.

현대 대규모 언어 모델(LLM) 추론 워크로드는 디코드/출력 생성(Decode) 단계에서 가중치를 매 순전파마다 한 번씩 읽어와야 하므로 종종 메모리 제약 상태에 놓입니다. 다중 토큰 예측(multi-token prediction)이나 투기적 디코딩(speculative decoding)을 사용하지 않는 한, 이러한 가중치 로드는 매 출력 토큰마다 1회씩 발생합니다. 따라서 메모리 제약 상태인 트랜스포머 기반 대규모 언어 모델 추론에서 토큰 간 최소 지연 시간(inter-token latency 또는 토큰당 생성 시간)을 손쉽게 산출할 수 있습니다.

16비트 정밀도로 저장되어 총 1 TB 크기를 갖는 500B(5,000억 개) 파라미터 모델을 가정해 보겠습니다. [메모리 대역폭](/gpu-glossary/perf/memory-bandwidth)이 10 TB/s인 단일 GPU에서 추론을 실행한다면, 100밀리초(ms)마다 한 번씩 가중치를 로드할 수 있으므로 이것이 토큰 간 지연 시간의 이론적 하한선이 됩니다. 여러 입력을 하나로 묶는 배치(batch) 처리를 적용하면, 로드된 파라미터당 수행되는 부동소수점 연산 수([연산 강도](/gpu-glossary/perf/arithmetic-intensity))를 선형적으로 증가시킬 수 있습니다. 이는 이론상 [연산 제약](/gpu-glossary/perf/compute-bound) 지점에 도달할 때까지 지연 시간의 추가 증가 없이 처리량을 배치 크기에 비례하여 선형적으로 향상시킬 수 있음을 의미합니다.

LLM 추론에 관한 자세한 내용은 [LLM 엔지니어 연감(LLM Engineer's Almanac)](https://modal.com/llm-almanac/summary)을 참조하시기 바랍니다.
