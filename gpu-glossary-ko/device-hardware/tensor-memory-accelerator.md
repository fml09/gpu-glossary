---
title: 텐서 메모리 가속기(TMA)란 무엇인가?
abbreviation: TMA
---

텐서 메모리 가속기(Tensor Memory Accelerator, TMA)는 [GPU RAM](/gpu-glossary/device-hardware/gpu-ram)에 저장된 다차원 배열 접근을 가속하기 위해 호퍼(Hopper) 및 블랙웰(Blackwell) [아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture) GPU에 도입된 특화 하드웨어입니다.

![H100 [스트리밍 다중처리기(SM)](/gpu-glossary/device-hardware/streaming-multiprocessor) 내부 아키텍처. 4개 하위 유닛이 공유하도록 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor) 하단에 배치된 텐서 메모리 가속기에 주목하십시오. NVIDIA의 [H100 백서](https://modal-cdn.com/gpu-glossary/gtc22-whitepaper-hopper.pdf)를 기반으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-gh100-sm.svg)

TMA는 [레지스터](/gpu-glossary/device-software/registers)/[레지스터 파일](/gpu-glossary/device-hardware/register-file)을 완전히 거치지 않고(bypass), [전역 메모리](/gpu-glossary/device-software/global-memory)/[GPU RAM](/gpu-glossary/device-hardware/gpu-ram)에서 [공유 메모리](/gpu-glossary/device-software/shared-memory)/[L1 데이터 캐시](/gpu-glossary/device-hardware/l1-data-cache)로 데이터를 직접 전송합니다.

TMA의 첫 번째 장점은 다른 연산 및 메모리 자원 소모를 줄여준다는 데 있습니다. TMA 하드웨어는 배열 데이터 접근 시 가장 흔히 나타나는 `addr = width * base + offset` 형태의 대규모 아핀(affine) 메모리 주소 계산을 다수의 베이스와 오프셋에 대해 병렬로 수행합니다. 이 작업을 TMA로 오프로드하면 [레지스터 파일](/gpu-glossary/device-hardware/register-file) 공간을 절약하여 "[레지스터 압박](/gpu-glossary/perf/register-pressure)"을 줄일 수 있고, [CUDA 코어](/gpu-glossary/device-hardware/cuda-core)가 제공하는 [연산 대역폭](/gpu-glossary/perf/arithmetic-bandwidth) 부담도 덜 수 있습니다. 이러한 절감 효과는 2차원 이상의 대용량(KB 단위) 배열에 접근할 때 더욱 두드러집니다.

두 번째 장점은 TMA 데이터 복사의 비동기 실행 모델에서 나옵니다. 단일 [CUDA 스레드](/gpu-glossary/device-software/thread)가 대규모 복사를 트리거한 다음 소속 [워프](/gpu-glossary/device-software/warp)로 곧바로 복귀해 다른 작업을 계속 수행할 수 있습니다. 해당 [스레드](/gpu-glossary/device-software/thread)와 같은 [스레드 블록](/gpu-glossary/device-software/thread-block)의 다른 스레드들은 복사가 완료된 후 비동기적으로 완료 여부를 감지하여 생산자-소비자 모델 방식으로 결과를 가져와 연산할 수 있습니다.

자세한 내용은 [Luo et al.의 호퍼 마이크로벤치마크 논문](https://arxiv.org/abs/2501.12084v1)의 TMA 관련 내용과 [NVIDIA 호퍼 튜닝 가이드](https://docs.nvidia.com/cuda/hopper-tuning-guide/index.html#tensor-memory-accelerator)를 참고하시기 바랍니다.

주의할 점은, 명칭과 달리 텐서 메모리 가속기가 [텐서 메모리](/gpu-glossary/device-hardware/tensor-memory)를 사용하는 연산을 가속하는 장치는 아니라는 사실입니다.
