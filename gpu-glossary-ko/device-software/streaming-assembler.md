---
title: 스트리밍 어셈블러(SASS)란 무엇인가?
abbreviation: SASS
---

[스트리밍 어셈블러(Streaming ASSembler)](https://stackoverflow.com/questions/9798258/what-is-sass-short-for)(SASS)는 NVIDIA GPU에서 실행되는 프로그램을 위한 어셈블리 형식입니다. 사람이 읽을 수 있는 형태로 작성 가능한 가장 저수준의 코드 형식에 해당합니다. [NVIDIA CUDA 컴파일러 드라이버](/gpu-glossary/host-software/nvcc)인 `nvcc`가 [PTX](/gpu-glossary/device-software/parallel-thread-execution)와 함께 출력하는 형식 중 하나입니다. SASS는 실행 과정에서 디바이스별 바이너리 마이크로코드로 변환됩니다. '스트리밍 어셈블러'라는 명칭의 '스트리밍(Streaming)'은 이 어셈블리 언어가 대상으로 삼는 하드웨어인 [스트리밍 다중처리기(SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)에서 유래한 것으로 추정됩니다.

SASS는 특정 NVIDIA GPU의 [SM 아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture)에 종속되어 버전이 관리됩니다. [컴퓨트 성능](/gpu-glossary/device-software/compute-capability) 항목도 함께 참조하십시오.

Hopper GPU의 SM90a 아키텍처를 대상으로 하는 대표적인 SASS 명령어 예시는 다음과 같습니다.

- `FFMA R0, R7, R0, 1.5 ;`: 혼합 부동소수점 곱셈-덧셈(Fused Floating point Multiply Add) 연산으로, 레지스터 7(`R7`)과 레지스터 0(`R0`)의 값을 곱한 뒤 `1.5`를 더하여 결과를 레지스터 0(`R0`)에 저장합니다.
- `S2UR UR4, SR_CTAID.X ;`: [협력형 스레드 배열(CTA)](/gpu-glossary/device-software/cooperative-thread-array) 인덱스의 `X`축 값을 특수 레지스터(Special Register, `SR`)에서 균일 레지스터(Uniform Register, `UR4`)로 복사합니다.

CPU 어셈블리어보다도 훨씬 더, 이러한 'GPU 어셈블러'를 사람이 직접 손으로 작성하는 일은 극히 드뭅니다. 그보다는 고수준 [CUDA C/C++](/gpu-glossary/host-software/cuda-c) 코드나 인라인 [PTX](/gpu-glossary/device-software/parallel-thread-execution)를 프로파일링하고 수정하는 과정에서 컴파일러가 생성한 SASS를 [확인하고 분석하는 방식](https://docs.nvidia.com/gameworks/content/developertools/desktop/ptx_sass_assembly_debugging.htm)이 훨씬 흔하며, 특히 최고 수준의 성능을 요구하는 커널을 개발할 때 널리 쓰입니다. [CUDA C/C++](/gpu-glossary/host-software/cuda-c), SASS, [PTX](/gpu-glossary/device-software/parallel-thread-execution)를 함께 비교해 보는 기능은 [Godbolt](https://godbolt.org/z/5r9ej3zjW)에서 지원합니다. 성능 디버깅 워크플로를 중심으로 한 SASS의 상세한 내용은 아룬 드뫼르(Arun Demeure)의 [발표 영상](https://www.youtube.com/watch?v=we3i5VuoPWk)을 참조하십시오.

SASS는 공개 문서화가 _극도로_ 제한적입니다. [NVIDIA CUDA 바이너리 유틸리티 문서](https://docs.nvidia.com/cuda/cuda-binary-utilities/index.html#instruction-set-ref)에 명령어 목록이 나열되어 있기는 하지만 구체적인 동작 시맨틱은 정의되어 있지 않습니다. ASCII 어셈블러에서 바이너리 옵코드 및 피연산자로 변환되는 매핑 규칙 또한 전혀 공개되어 있지 않지만, 일부 연구자와 엔지니어들에 의해 리버스 엔지니어링된 사례가 있습니다([Maxwell](https://github.com/NervanaSystems/maxas), [Lovelace](https://kuterdinel.com/nv_isa_sm89/)).
