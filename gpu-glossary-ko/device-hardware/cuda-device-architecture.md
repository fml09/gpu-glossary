---
title: CUDA 디바이스 아키텍처란 무엇인가?
---

CUDA는 *컴퓨트 통합 디바이스 아키텍처(Compute Unified Device Architecture)*의 약어입니다. 맥락에 따라 "CUDA"는 여러 가지 서로 다른 대상을 가리킬 수 있습니다. 상위 수준의 디바이스 아키텍처 자체를 뜻하기도 하고, 이러한 설계를 갖춘 아키텍처를 위한 [병렬 프로그래밍 모델](/gpu-glossary/device-software/cuda-programming-model)을 의미하기도 하며, C와 같은 고급 언어를 확장하여 해당 프로그래밍 모델을 지원하는 [소프트웨어 플랫폼](/gpu-glossary/host-software/cuda-software-platform)을 지칭하기도 합니다.

CUDA의 비전은 [Lindholm et al., 2008](https://www.cs.cmu.edu/afs/cs/academic/class/15869-f11/www/readings/lindholm08_tesla.pdf) 백서에 잘 정리되어 있습니다. 이 백서는 NVIDIA 공식 문서에서 다루는 다양한 주장, 다이어그램, 특정 표현의 원천이 되는 자료이므로 일독을 추천합니다.

여기서는 CUDA의 *디바이스 아키텍처* 측면에 초점을 맞춥니다. '컴퓨트 통합 디바이스 아키텍처'의 핵심 특징은 이전 GPU 아키텍처에 비해 구조가 단순하다는 점입니다.

GeForce 8800 및 이로부터 파생된 Tesla 데이터센터 GPU가 출시되기 전까지, NVIDIA GPU는 소프트웨어 셰이더 단계를 이종의 특화 하드웨어 유닛에 매핑하는 복잡한 파이프라인 셰이더 아키텍처로 설계되었습니다. 이러한 구조는 소프트웨어와 하드웨어 엔지니어 모두에게 큰 부담이었습니다. 소프트웨어 엔지니어는 프로그램을 고정된 파이프라인에 억지로 맞춰야 했고, 하드웨어 엔지니어는 파이프라인 단계 간 부하 비율을 예측해 하드웨어를 설계해야 했기 때문입니다.

![고정 파이프라인 디바이스 아키텍처(G71) 다이어그램. 프래그먼트 셰이딩과 버텍스 셰이딩을 처리하기 위한 별도의 프로세서 그룹이 존재하는 점에 주목하십시오. [Fabien Sanglard의 블로그](https://fabiensanglard.net/cuda/)를 참고하여 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-fixed-pipeline-g71.svg)

통합 아키텍처를 채택한 GPU 디바이스는 훨씬 단순합니다. 하드웨어 유닛이 완전히 균일하며, 각 유닛이 다양한 연산을 두루 수행할 수 있습니다. 이러한 유닛을 [스트리밍 다중처리기(Streaming Multiprocessor, SM)](/gpu-glossary/device-hardware/streaming-multiprocessor)라고 부르며, 주요 하위 구성 요소로는 [CUDA 코어(CUDA Core)](/gpu-glossary/device-hardware/cuda-core)와 최신 GPU에 탑재된 [텐서 코어(Tensor Core)](/gpu-glossary/device-hardware/tensor-core)가 있습니다.

![컴퓨트 통합 디바이스 아키텍처(G80) 다이어그램. 프로세서 유형 구분이 사라진 점에 주목하십시오. 모든 핵심 연산은 다이어그램 중앙의 동일한 [스트리밍 다중처리기](/gpu-glossary/device-hardware/streaming-multiprocessor)에서 수행되며, 여기에 버텍스, 지오메트리, 픽셀 스레드를 위한 명령어가 전달됩니다. [Peter Glazkowsky의 2009년 Fermi 아키텍처 백서](https://www.nvidia.com/content/pdf/fermi_white_papers/p.glaskowsky_nvidia%27s_fermi-the_first_complete_gpu_architecture.pdf)를 기반으로 수정하였습니다.](https://modal-cdn.com/gpu-glossary/light-cuda-g80.svg)

CUDA 하드웨어 아키텍처의 역사와 설계를 알기 쉽게 소개한 자료로는 Fabien Sanglard의 [블로그 글](https://fabiensanglard.net/cuda/)이 있습니다. 이 글은 NVIDIA의 [Fermi 컴퓨트 아키텍처 백서](https://www.nvidia.com/content/pdf/fermi_white_papers/nvidia_fermi_compute_architecture_whitepaper.pdf)처럼 신뢰할 수 있는 우수한 자료들을 인용하고 있습니다. 또한 Tesla 아키텍처를 소개한 [Lindholm et al.(2008)](https://www.cs.cmu.edu/afs/cs/academic/class/15869-f11/www/readings/lindholm08_tesla.pdf) 백서는 완성도가 높고 매우 상세합니다. [NVIDIA의 Tesla P100 백서](https://images.nvidia.com/content/pdf/tesla/whitepaper/pascal-architecture-whitepaper.pdf)는 학술 논문 형태는 아니지만, NVLink나 [온패키지 고대역폭 메모리](/gpu-glossary/device-hardware/gpu-ram)처럼 오늘날 대규모 신경망 워크로드에서 핵심적인 여러 기능의 도입 배경을 상세히 기록하고 있습니다.
