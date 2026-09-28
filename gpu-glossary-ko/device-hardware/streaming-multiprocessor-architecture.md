---
title: 스트리밍 다중처리기 아키텍처란 무엇인가?
---

[스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)는 [SASS(Streaming Assembler)](/gpu-glossary/device-software/streaming-assembler) 코드 호환성을 정의하는 특정 "아키텍처" 버전으로 관리됩니다.

!["Hopper" SM90 아키텍처를 채택한 스트리밍 다중처리기. NVIDIA의 [H100 백서](https://modal-cdn.com/gpu-glossary/gtc22-whitepaper-hopper.pdf)를 기반으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-gh100-sm.svg)

!["Tesla" 초기 SM 아키텍처를 적용한 스트리밍 다중처리기. [Fabien Sanglard의 블로그](https://fabiensanglard.net/cuda)를 참고하여 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-tesla-sm.svg)

대부분의 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor) 버전은 주 버전(major version)과 부 버전(minor version)이라는 두 가지 숫자로 이루어집니다.

주 버전은 대체로 GPU 아키텍처 제품군과 거의 일치합니다. 예를 들어 모든 `6.x` SM 버전은 파스칼(Pascal) 아키텍처에 속합니다. 일부 NVIDIA 문서에서는 이러한 관계를 [직접적으로 명시](https://docs.nvidia.com/cuda/ptx-writers-guide-to-interoperability/index.html)하기도 합니다. 그러나 예외도 존재합니다. 예를 들어 에이다(Ada Lovelace) GPU의 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor) 아키텍처 버전은 `8.9`로, 암페어(Ampere) GPU와 동일한 주 버전을 갖습니다.

[NVIDIA CUDA 컴파일러 드라이버](/gpu-glossary/host-software/nvcc)인 `nvcc`를 호출할 때 [SASS](/gpu-glossary/device-software/streaming-assembler) 컴파일 대상이 될 목표 [SM](/gpu-glossary/device-hardware/streaming-multiprocessor) 버전을 지정할 수 있습니다. 주 버전이 서로 다르면 바이너리 호환성은 명시적으로 보장되지 않습니다. 부 버전 간 호환성에 관한 자세한 내용은 [nvcc](/gpu-glossary/host-software/nvcc) [문서](https://docs.nvidia.com/cuda/cuda-compiler-driver-nvcc/index.html#gpu-feature-list)에서 확인할 수 있습니다.
