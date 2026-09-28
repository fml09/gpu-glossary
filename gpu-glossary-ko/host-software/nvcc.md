---
title: NVIDIA CUDA 컴파일러 드라이버(nvcc)란 무엇인가?
abbreviation: nvcc
---

NVIDIA CUDA 컴파일러 드라이버(NVIDIA CUDA Compiler Driver, `nvcc`)는 [CUDA C/C++](/gpu-glossary/host-software/cuda-c) 프로그램을 빌드하기 위한 툴체인입니다. 호스트 ABI를 준수하면서 GPU에서 실행될 [PTX](/gpu-glossary/device-software/parallel-thread-execution) 및 [SASS](/gpu-glossary/device-software/streaming-assembler) 코드를 함께 내장한 실행 바이너리(이른바 '팻 바이너리(Fat Binary)')를 생성합니다. 이러한 바이너리는 리눅스의 `readelf` 같은 표준 바이너리 분석 도구로 내용을 확인할 수 있으며, 전용 [CUDA 바이너리 유틸리티](/gpu-glossary/host-software/cuda-binary-utilities)를 사용해 세부적으로 조작할 수도 있습니다.

바이너리에 포함되는 [PTX](/gpu-glossary/device-software/parallel-thread-execution) 코드는 [컴퓨트 성능(Compute Capability)](/gpu-glossary/device-software/compute-capability)에 따라 버전이 나뉘며, `--gpu-architecture` 또는 `--gpu-code` 컴파일 옵션에 `compute_XYz` 값을 전달하여 대상 버전을 지정합니다.

바이너리에 포함되는 [SASS](/gpu-glossary/device-software/streaming-assembler) 코드는 [SM 아키텍처 버전](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture)에 따라 버전이 구분되며, `--gpu-architecture` 또는 `--gpu-code` 옵션에 `sm_XYz` 값을 지정하여 생성합니다. `--gpu-code` 옵션에 `compute_XYz` 값을 전달하면 지정한 [PTX](/gpu-glossary/device-software/parallel-thread-execution)와 일치하는 버전의 [SASS](/gpu-glossary/device-software/streaming-assembler) 코드가 함께 생성됩니다.

호스트 CPU 코드의 컴파일은 시스템의 호스트 C/C++ 컴파일러(예: `gcc` 컴파일러 드라이버)가 담당합니다. 이때 컴파일러 드라이버는 [NVIDIA GPU 드라이버](/gpu-glossary/host-software/nvidia-gpu-drivers)와 같은 하드웨어 드라이버와는 전혀 다른 소프트웨어라는 점에 유의해야 합니다.

`nvcc`의 공식 설명서는 [NVIDIA nvcc 설명서](https://docs.nvidia.com/cuda/cuda-compiler-driver-nvcc/)에서 확인할 수 있습니다.
