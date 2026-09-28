---
title: "cuTile BASIC이란 무엇인가?"
---

cuTile BASIC은 [BASIC 프로그래밍 언어](https://modal-cdn.com/BASIC_Oct64.pdf) 환경에서 [CUDA 타일 프로그래밍 모델](/gpu-glossary/device-software/cuda-tile-programming-model)을 구현한 프로젝트입니다.

BASIC은 '초심자용 범용 기호 명령 부호(Beginner's All-purpose Symbolic Instruction Code)'의 약칭입니다. 1960년대에 쉬운 사용성과 대화형 인터랙티브 프로그래밍을 목적으로 설계되었으며, 윌리엄 게이츠 3세(빌 게이츠)를 포함한 초기 개인용 마이크로컴퓨터 프로그래머들 사이에서 큰 인기를 끌었습니다.

cuTile BASIC은 공식적으로 [만우절 장난](https://developer.nvidia.com/blog/cuda-tile-programming-now-available-for-basic/)으로 발표되었습니다. 비록 유쾌한 장난감 프로젝트이지만 타일 프로그래밍 모델의 핵심 기능을 실제로 충실히 구현하고 있으며, 모델 자체의 뛰어난 범용성을 증명하는 사례입니다. 아래에 소개하는 벡터 덧셈 cuTile BASIC 커널은 [Modal 노트북 예제](https://modal.com/notebooks/modal-labs/examples/nb-151VgRNHYEDuKSfxJRjV5N)를 통해 B200 GPU에서 직접 구동해 볼 수 있습니다. 실제로 cuTile BASIC 프로젝트의 일부는 이러한 노트북 환경을 활용하여 개발되었습니다.

```basic
10 REM Vector Add: C = A + B
20 INPUT N, A(), B()
30 DIM A(N), B(N), C(N)
40 TILE A(128), B(128), C(128)
50 LET C(BID) = A(BID) + B(BID)
60 OUTPUT C
70 END
```
