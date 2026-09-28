---
title: NVIDIA Nsight Systems란 무엇인가?
---

NVIDIA Nsight Systems는 [CUDA C++](/gpu-glossary/host-software/cuda-c) 프로그램을 위한 시스템 수준의 성능 디버깅 도구입니다. 프로파일링, 시스템 트레이싱, 분석 기능을 그래픽 사용자 인터페이스(GUI) 환경에 결합하여 제공합니다.

어느 누구도 아침에 일어나 "오늘은 다루기 까다롭고 값비싼 하드웨어에서 독점 소프트웨어 스택을 써가며 프로그램을 짜야겠다"고 생각하지는 않습니다. GPU를 도입하는 이유는 일반적인 컴퓨팅 하드웨어로는 주어진 계산 문제를 만족스러운 성능으로 해결하기 어렵기 때문입니다. 따라서 [거의 모든 GPU 프로그램은 성능에 극도로 민감하며](/gpu-glossary/perf), Nsight Systems를 비롯하여 [CUDA 프로파일링 도구 인터페이스(CUPTI)](/gpu-glossary/host-software/cupti)를 기반으로 구축된 도구가 제공하는 성능 디버깅 워크플로는 매우 중대한 역할을 차지합니다.

공식 설명서는 [Nsight Systems 문서](https://docs.nvidia.com/nsight-systems/index.html)에서 확인할 수 있지만, [실제 사용 영상](https://www.youtube.com/watch?v=dUDGO66IadU)을 직접 살펴보는 편이 직관적인 이해에 더 큰 도움이 됩니다. Modal에서 GPU 애플리케이션을 프로파일링하는 상세한 방법은 [Modal 공식 예제 문서](https://modal.com/docs/examples/torch_profiling)를 참고하시기 바랍니다.
