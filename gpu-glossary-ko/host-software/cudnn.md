---
title: cuDNN이란 무엇인가?
---

NVIDIA의 cuDNN(CUDA Deep Neural Network)은 GPU 가속 심층 신경망을 구축하기 위한 기본 연산(Primitives) 라이브러리입니다.

cuDNN은 인공신경망에서 자주 쓰이는 핵심 연산들을 고도로 최적화된 [커널](/gpu-glossary/device-software/kernel) 형태로 제공합니다. 대표적으로 합성곱(Convolution), 셀프 어텐션(스케일드 닷 프로덕트 어텐션 및 플래시 어텐션(Flash Attention) 포함), 행렬 곱셈, 다양한 정규화(Normalization), 풀링(Pooling) 연산 등이 있습니다.

cuDNN은 자매 라이브러리인 [cuBLAS](/gpu-glossary/host-software/cublas)와 함께 [CUDA 소프트웨어 플랫폼](/gpu-glossary/host-software/cuda-software-platform)의 애플리케이션 계층에서 핵심 축을 이룹니다. PyTorch 같은 딥러닝 프레임워크는 완전 연결 계층(Fully-Connected Layer)의 중심 연산인 행렬 곱셈처럼 범용 선형대수 작업에는 [cuBLAS](/gpu-glossary/host-software/cublas)를 주로 활용합니다. 반면 합성곱 계층, 정규화 루틴, 어텐션 구조처럼 신경망 특화 기본 요소에는 cuDNN을 적극 활용합니다.

최신 cuDNN 코드에서는 계산 과정을 연산 그래프(Operation Graph)로 표현합니다. 이는 선언형 [Graph API](https://docs.nvidia.com/deeplearning/cudnn/frontend/v1.14.0/developer/graph-api.html)를 지원하는 오픈 소스 [파이썬 및 C++ 프론트엔드 API](https://docs.nvidia.com/deeplearning/cudnn/frontend/latest/developer/overview.html)를 통해 구성할 수 있습니다(이는 [CUDA 그래프](/gpu-glossary/host-software/cuda-graph)와는 구별되는 별도의 개념입니다).

이 API를 사용하면 일련의 연산 단계를 그래프 형태로 정의할 수 있으며, cuDNN은 이를 분석하여 가장 중요한 최적화 기법 중 하나인 연산 융합(Operation Fusion)을 수행합니다. 연산 융합은 합성곱 + 편향(Bias) 더하기 + ReLU와 같이 연속된 연산들을 하나의 단일 작업으로 병합하여 단 하나의 [커널](/gpu-glossary/device-software/kernel)로 실행하는 기법입니다. 연산 융합은 연산 중간 결과를 외부 메모리로 내보내지 않고 [공유 메모리](/gpu-glossary/device-software/shared-memory)에 유지하므로 [메모리 대역폭](/gpu-glossary/perf/memory-bandwidth)의 낭비를 크게 줄여줍니다.

이러한 상위 프론트엔드는 하위 계층의 비공개 소스 [C 백엔드](https://docs.nvidia.com/deeplearning/cudnn/backend/latest/api/overview.html)와 연동됩니다. C 백엔드는 레거시 시스템 호환이나 C 언어의 외부 함수 인터페이스(FFI) 직접 연동을 지원하는 기본 API를 제공합니다.

특정 연산에 대해 cuDNN은 여러 가지 내부 구현체를 갖추고 있으며, 대상 [스트리밍 다중처리기(SM) 아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture), 데이터 타입, 입력 크기에 맞추어 가장 뛰어난 성능을 내는 구현체를 선택하는 내부 휴리스틱을 동작시킵니다.

cuDNN이 본격적으로 주목받게 된 계기는 암페어(Ampere) [SM 아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture) GPU에서 합성곱 신경망(CNN)을 폭발적으로 가속하면서부터였습니다. 다만 호퍼(Hopper) 및 최신 블랙웰(Blackwell) [SM 아키텍처](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture)에서 구동되는 대규모 트랜스포머(Transformer) 모델에 대해서는 NVIDIA 역시 [CUTLASS](/gpu-glossary/host-software/cutlass) 라이브러리를 더욱 강조하는 추세입니다.

cuDNN에 관한 세부 정보는 [공식 cuDNN 문서](https://docs.nvidia.com/deeplearning/cudnn/) 및 [오픈 소스 프론트엔드 API 저장소](https://github.com/NVIDIA/cudnn-frontend)에서 확인할 수 있습니다.
