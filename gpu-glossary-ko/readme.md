---
title: README
---

<pre class="text-xs md:text-base font-mono whitespace-pre">
 ██████╗ ██████╗ ██╗   ██╗
██╔════╝ ██╔══██╗██║   ██║
██║  ███╗██████╔╝██║   ██║
██║   ██║██╔═══╝ ██║   ██║
╚██████╔╝██║     ╚██████╔╝
 ╚═════╝ ╚═╝      ╚═════╝
 ██████╗ ██╗      ██████╗ ███████╗███████╗ █████╗ ██████╗ ██╗   ██╗
██╔════╝ ██║     ██╔═══██╗██╔════╝██╔════╝██╔══██╗██╔══██╗╚██╗ ██╔╝
██║  ███╗██║     ██║   ██║███████╗███████╗███████║██████╔╝ ╚████╔╝
██║   ██║██║     ██║   ██║╚════██║╚════██║██╔══██║██╔══██╗  ╚██╔╝
╚██████╔╝███████╗╚██████╔╝███████║███████║██║  ██║██║  ██║   ██║
 ╚═════╝ ╚══════╝ ╚═════╝ ╚══════╝╚══════╝╚═╝  ╚═╝╚═╝  ╚═╝   ╚═╝
 </pre>

저희는 [Modal](https://modal.com)에서 GPU를 활용한 작업을 진행하며 겪었던 문제를 해결하기 위하여 이 용어집을 작성하였습니다. 기존 기술 문서는 여러 갈래로 분산되어 있어서, [스트리밍 다중처리기 아키텍처(Streaming Multiprocessor Architecture)](/gpu-glossary/device-hardware/streaming-multiprocessor-architecture)와 [컴퓨트 성능(Compute Capability)](/gpu-glossary/device-software/compute-capability), 그리고 [nvcc 컴파일러 플래그](/gpu-glossary/host-software)처럼 전체 기술 스택의 여러 계층에 걸쳐 있는 개념들을 유기적으로 연결하여 이해하기가 어려웠습니다.

이에 따라 저희는 [NVIDIA 공식 문서](https://docs.nvidia.com/cuda/pdf/PTX_Writers_Guide_To_Interoperability.pdf)를 면밀히 검토하고, 활발한 [Discord 커뮤니티](https://discord.gg/gpumode)에서 개발자들과 교류하며, [전문 서적](https://www.amazon.com/Professional-CUDA-Programming-John-Cheng/dp/1118739329)을 직접 참조하여 하드웨어부터 소프트웨어까지 전체 스택을 한곳에 아우르는 용어집을 정리하였습니다.

이 용어집은 일반적인 PDF 문서나 Discord 대화, 단행본 서적과 달리 상호 연결된 하이퍼텍스트 문서입니다. 모든 페이지가 서로 유기적으로 연결되어 있으므로, [CUDA 프로그래밍 모델](/gpu-glossary/host-software/cuda-c) 문서를 읽다가 마주친 [스레드(Thread)](/gpu-glossary/device-software/thread) 개념을 더욱 깊이 이해하기 위해 바로 [워프 스케줄러(Warp Scheduler)](/gpu-glossary/device-hardware/warp-scheduler) 문서로 건너가서 읽어보실 수 있습니다.

또한 목차 순서에 맞추어 순차적으로 읽어나가실 수도 있습니다. 페이지 사이를 이동하려면 키보드의 방향키나 각 페이지 하단의 화살표를 이용하시거나, 데스크톱 화면의 사이드바 또는 모바일 화면의 메뉴에 마련된 목차를 이용해 주시기 바랍니다.

이 용어집의 원본 저장소는 [GitHub](https://github.com/modal-labs/gpu-glossary)에서 확인하실 수 있습니다.
