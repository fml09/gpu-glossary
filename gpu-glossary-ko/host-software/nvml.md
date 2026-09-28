---
title: NVIDIA 관리 라이브러리(NVML)란 무엇인가?
abbreviation: NVML
---

NVIDIA 관리 라이브러리(NVIDIA Management Library, NVML)는 NVIDIA GPU의 상태를 모니터링하고 관리하는 데 사용되는 C 기반 API 라이브러리입니다. 이 라이브러리는 GPU의 소비 전력과 온도, 할당된 메모리 사용량, 디바이스의 전력 제한값 및 전력 제한 상태 등을 외부에 노출합니다. 전력 및 온도 측정값을 해석하는 방법을 비롯한 세부 지표에 관한 내용은 [Modal 문서의 관련 페이지](https://modal.com/docs/guide/gpu-metrics)에서 확인할 수 있습니다.

NVML의 기능은 [nvidia-smi](/gpu-glossary/host-software/nvidia-smi) 명령줄 유틸리티를 통해 가장 흔하게 접할 수 있으며, 파이썬의 [pynvml](https://pypi.org/project/pynvml/)이나 러스트의 [nvml_wrapper](https://docs.rs/nvml-wrapper/latest/nvml_wrapper/)와 같은 언어별 래퍼(Wrapper)를 통해 애플리케이션 코드에서도 직접 제어할 수 있습니다.
