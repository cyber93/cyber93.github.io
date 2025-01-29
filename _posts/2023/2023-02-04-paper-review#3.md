---
title:  "Analysis of I/O Response Time Throughout NVMe Driver Implementation Architectures"
excerpt: "Paper Review of NVMe Storage Technology #3"

toc: true
toc_sticky: true

categories:
  - NVMe Storage Technology
tags:
  - Paper Review of NVMe Storage Technology
---

<br>

> [IEMEK '17](https://doi.org/10.14372/IEMEK.2017.12.3.139)

# (공사 중)Analysis of I/O Response Time Throughout NVMe Driver Implementation Architectures

## **Abstract**





## **1. Introduction**

최근에는 솔리드 스테이트 드라이브(SSD)의 가속화로 인한 성능 병목 현상을 극복하기 위해 새로운 호스트 컨트롤러 인터페이스 표준인 비휘발성 메모리 익스프레스(NVMe)가 적용되고 있습니다. 최근 서버와 PC에 NVMe 기반 PCI Express(PCIe) SSD를 적용하여 AHCI 기반 SATA SSD보다 성능을 획기적으로 개선한 사례가 보고되고 있습니다.

또한 스마트폰과 같은 차세대 모바일 디바이스에서도 레거시 eMMC 플래시 스토리지를 NVMe 기반 스토리지로 교체하는 것이 고려되고 있습니다. Linux 커널에는 NVMe 지원을 위한 드라이버가 포함되어 있으며, 커널 버전이 높아짐에 따라 NVMe 드라이버 코드의 구현이 변경되었습니다. 그러나 모바일 디바이스에는 최신 버전의 Android 운영 체제(OS)가 탑재되어 있는 경우가 많으며, 이 경우 NVMe 드라이버의 최신 기능을 사용할 수 없습니다.

따라서 각기 다른 NVMe 드라이버 구현의 다양한 기능이 안드로이드 OS에서 제대로 평가되지 않습니다. 이 백서에서는 다양한 Linux 커널 버전에 대한 NVMe 드라이버의 응답 시간을 분석합니다.



## **2. Background**

![img](/assets/images/paper3-1.png)

![img](/assets/images/paper3-2.png)





## **3.  NVMe I/O 계층 구현 변경점**



## **4. 구현 변경에 따른 응답시간 비교**



## **5. 측정 결과**



## **6. 결론 및 향후 연구**

