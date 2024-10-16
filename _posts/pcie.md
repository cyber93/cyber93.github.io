---
title:  "PCIe Interface"
excerpt: "NVMe Spec. Rev. 1.4c 이야기 #3"

toc: true
toc_sticky: true

categories:
  - NVMe Storage Technology
tags:
  - NVMe Spec
  - PCIe
---

<br>

# (공사 중)PCIe Interface

## **Ice Breaking PCIe interface (not included in spec)**

2장이 시작되기 전 PCI 및 PCIe에 대한 기본적 지식이 필요함을 느껴 진행 한다. 

참조 자료:

1. 1. [mindshare에서 나온 PCIe pdf](https://www.mindshare.com/files/ebooks/PCI Express System Architecture.pdf)
   2. [PCIe 관련 블로그1 - 문c 블로그 - PCI subsytem](http://jake.dothome.co.kr/pci-1/)
   3. [PCIe driver development for Exynos SoC by Samsung](http://kossa.kr/materials/LinuxForum/15 PCIe Driver Development for Exynos SoC.pdf)
   4. [Linux PCI drivers - Free Electrons](https://bootlin.com/doc/legacy/pci-drivers/pci-drivers.pdf)
   5. [Overview of PCIe subsystem - Texas Instruments](https://prezi.com/abwyjs_zofyg/overview-of-pcie-subsystem/)
   6. [PCI system architecture, Local Bus, Tutorial - Porto University](https://paginas.fe.up.pt/~ee00013/microPCI/materialGroup04.html)
   7. [PCIe 관련 블로그2](http://melonicedlatte.com/computerarchitecture/2019/11/26/184500.html)
   8. http://egloos.zum.com/nimphaplz/v/5314763
   9. https://wiki.qemu.org/images/f/f6/PCIvsPCIe.pdf
   10. https://pcisig.com/sites/default/files/files/PCI_Express_Basics_Background.pdf#page=26
   11. https://slidesplayer.org/slide/14669145/
   12. https://wiki.osdev.org/PCI
   13. https://static.lwn.net/images/pdf/LDD3/ch12.pdf
   14. https://wiki.qemu.org/images/f/f6/PCIvsPCIe.pdf

### **Peripheral Component Interconnect Bus (\**PCI\**)**

**위키백과**
PCI 버스는 컴퓨터 메인보드에 주변 장치를 장착하는 데 쓰이는 컴퓨터 버스의 일종이다.
이 장치는 다음과 같이 두 가지 형태로 나뉜다.
 \1. 주기판 위에 바로 붙는 IC 형태 - PCI 스펙에서는 이러한 형태를 평면 장치(planar device)라고 부른다.
 \2. 소켓에 꽂아 쓰는 확장 카드 형태 - 사용자 입장에서 흔히 눈에 띄는 형태이다.내가 본 PCI의 경우 2번의 형태가 많았다. 주로 sound card, ethernet card 등을 꽂아 사용한다.

### **PCI 특징**

PCI는 이전의 ISA, MCA, EISA, VESA 버스를 대체하기 위해 설계 되었다. 

PCI는 아래의 특징을 가진다.

1. 1. 1. 빠른 전송 속도
         - 기존 bus들은 낮은 clock cycle (ISA: 4.7MHz, MCA: 10MHz, EISA: 8.3MHz, VESA: 33MHz)을 가진다.
         - PCI는 33MHz부터 266MHz 이상의 clock cycle을 가져 데이터의 빠른 전송이 가능하다.
      2. 단일화된 인터페이스
         - CPU와 PCI bus 사이에 system/PCI bus bridge를 설치해 여러 프로세서를 위한 설계가 가능하다.
         - PCI bus에 PCI/Expansion bus bridge를 설치해 기존의 버스를 사용할 수 있다.
      3. 동시에 여러개의 function을 지원 
         - 같은 PCI bus상에 여러 bus master가 존재할 수 있다.
         - 하나의 디바이스에 동시에 여러개의 function을 지원할 수 있다. 

### **PCI Express (PCIe)**

**위키백과**
PCIe는 앞서 언급한 버스 표준들과 비교하여 높은 시스템 버스 대역폭, 적은 I/O 핀 수, 적은 물리적 면적, 버스 장치들에게 더 뛰어난 성능 확장성, 상세한 오류 검출 및 보고 구조(Advanced Error Reporting (AER), 핫-플러그 기능등 여러 장점을 가지고 있다. 최근에는 하드웨어 I/O 가상화도 지원한다\

### **PCIe 특징**

PCIe는 기본적으로 PCI가 가지는 성능적 한계 및 신호 왜곡 등을 개선하기 위해 기존의 병렬 bus로 구성된 PCI의 bus를 serial bus로 바꿔 설계했다. PCIe는 PCI 슬롯을 그대로 사용하며 여러개의 슬롯을 공유하지 않고 1:1로 연결해 사용한다. 따라서, 여러 장치를 붙일 경우 PCIe switch를 사용해 다수의 End device를 연결할 수 있도록 한다.

PCIe의 장점은 아래와 같다.

1. Bandwidth의 증가
2. Bus clock cycle의 증가
3. Bus width의 감소
4. 회로의 단순화
5. PCI 디바이스 SW 호환성 유지

기본적으로 PCI는 병렬 bus를 이용하기 때문에 신호 왜곡에 취약하기 대문에 bus의 clock cycle을 올리기 어렵다. 이를 dual-simplex 방식의 serial bus로 바꿈으로써 bus width의 감소, clock cycle의 증가, 회로의 단순화 등의 장점을 가진다.

([참조: http://www.technoa.co.kr/news/articleView.html?idxno=38092)](http://www.technoa.co.kr/news/articleView.html?idxno=38092)

### **PCIe Lane**

**Link**

x1 ~ x32 lane을 하나로 묶어 사용할 수 있다. 

1개의 Lane은 TX와 RX로 dual-simplex 구성으로 되어 있으며, 1개의link는 End-to-End Device간 1:1로 통신이 가능하다.



![img](https://blog.kakaocdn.net/dn/dbKdRt/btqAYHrLLJq/o7XaF0ULsqSjVD9PhDGLT0/img.png)Lane과 Link 구분



**Dual-simplex**

단 방향으로의 데이터 bus가 2개 1개 set로 구성되어, 하나의 bus는 데이터를 보내고 다른 하나의 bus는 데이터를 받는 것에만 사용된다. PCIe는 기본적으로 2개의 데이터 전달 통로(lane)을 갖는다. 

*Full duplex도 존재 한다. 여러가지 방법이 존재 함.



![img](https://blog.kakaocdn.net/dn/bBHtfV/btqA05rEVyR/D2cCzgNvBk5ChISu01pTsK/img.jpg)Dual-simplex



### ***\*PCIe Topology\****



![img](https://blog.kakaocdn.net/dn/t29Kr/btqA2a0lPJx/KVITsXMGz8NGH3XRalhFH0/img.png)PCIe topology architecture



위 그림은 PCIe를 통해 데이터를 전송하고자 하는 architecture의 topology를 나타낸다.

PCIe로 port를 연결하기 위해서는 기본적으로 Root Complex (RC)가 필요하다. 

RC는 CPU와 PCIe bus들 사이의 interface를 말하며 CPU interface, DRAM interface 등 여러가지 구성요소와 여러 개의 칩들을 포함한다. RC의 위치는 CPU내 ([구글링 검색 결과](https://www.quora.com/Does-the-motherboard-of-a-computer-act-as-the-PCIe-root-complex)), 메인보드 Chipset 내에 존재 한다.

전체 fabric의 instance들은 다음 구성요소이다.

1. 1. Root Complex
   2. Multiple endpoints (I/O devices)
   3. Switch
   4. PCIe to PCI/PCI-X bridge



![img](https://blog.kakaocdn.net/dn/K8bow/btqAXoGxotb/9AzUeUE9KjfOLEV8Mcqwm0/img.jpg)PCIe switch를 통한 PCIe 확장 port



Switch의 경우 위 그림 처럼 PCIe slot을 여러개 확장해서 사용할수 있도록 한다.

### **PCIe Root Complex (RC)**



![img](https://blog.kakaocdn.net/dn/VoMmQ/btqA2aF3q3K/w7SGjHFXknXiDgMH2euvK0/img.png)Root Complex 구조



RC는 Host CPU와 연결하기 위한 host bridge device와 secondary device 연결을 위한 PCIe port가 존재 한다. 

### **Switch**



![img](https://blog.kakaocdn.net/dn/bfkWpe/btqA06xpjyP/ZKZpQ7i9DJu352O0INzet0/img.jpg)Switch 구조



1. 1. Switch는 하나의 RC에 여러 디바이스를 붙일 수 있도록 만든 장치이다.
   2. Swtich는 여러개의 Down-stream ports를 가지고 있지만 up-stream port는 하나만 존재 한다. 
   3. 모든 활성화된 switch port는 flow control을 지원해야 한다. (Packet router처럼 동작하며, 해당 packet의 주소와 다른 routing 정보를 통해 어떤 경로로 packet이 가야하는지 인식한다.)

### **Endpoint Device**

1. 1. Endpoint device는 아래의 3가지 종류가 존재한다.
   2. Legacy PCI endpoint
      - Type 0 configuration header를 제공하는 function이여야 함
      - I/O 요청 제공 및 발생 가능
      - 4GB (32bit 주소 register일때) 주소를 초과하는 memory transaction 불가
      - MSI/MSI-X를 지원
   3. PCIe Endpoint
      - Type 0 configuration space를 제공하는 function이여야 함
      - I/O 요청 불가
      - 4GB 주소 범위를 넘는 memory transaction 가능 (64bit addressing 가능)
      - MSI/MSI-X 지원
   4. Root Complex integrated endpoint
      - Type 0 configuration space를 제공하는 function이여야 함
      - Root complex 내부 로직에 구현된 root port
      - I/O 요청 불가
      - 전원 관리 구현 X
      - Hot plug 지원 X
      - MSI/MSI-X 지원
      - [**32bit BAR를 통해 memory resource를 지원 (End-point device의 BAR와 비교한 결과 존재)**](https://blog.naver.com/PostView.nhn?blogId=joa_quin&logNo=221511458504&parentCategoryNo=&categoryNo=19&viewDate=&isShowPopularPosts=true&from=search)

### **PCIe 계층 구조 및 데이터 전송**



![img](https://blog.kakaocdn.net/dn/vc0tH/btqA05kUSZ9/q2EiAmlnVQoHWJaznGHHU1/img.jpg)PCIe 5개 Layer



PCI 전송에는 3개(Transaction layer, Data link layer, Physical layer)의 layer가 필요하다. 

PCI -> PCIe 변화 (paralle bus -> serial bus)로 변경되는 부분은 Transaction, Data Link, Physical 계층이다.



![img](https://blog.kakaocdn.net/dn/6pN5q/btqA05eb8L7/5qDCiDorNraXLsG7sQJob1/img.jpg)Packet의 생성과 Data의 전달



 

아래는 데이터의 전송 과정을 요약했다.

1. 1. Transaction layer에서는 packet 단위의 전송을 위해 데이터 packet을 형성하고 이를 Link layer로 전달한다.
   2. Link layer에서 데이터의 신뢰도를 위한 CRC 코드를 전달 받은 데이터에 삽입한다.
   3. Physical layer에서 frame 붙여 데이터를 추가 후 8b/10b 인코딩을 해 데이터를 전송하기 위해 serialized data로 변환한다.



![img](https://blog.kakaocdn.net/dn/bCy9lv/btqAXQireZD/3PgCX4oO7Ml2a0izzwT4k0/img.jpg)



 

Transaction layer에서 생성된 데이터 packet은 header와 data로 구성되어 있다. 해당 packet이 Data link layer로 전달되고 Data link layer에서 packet의 순서를 정한 번호와 CRC코드가 앞뒤로 붙게 된다. 그리고 Physical layer로 도착한 데이터에 대해 앞뒤로 frame을 붙이고 해당 packet을 serialize하며 이때 8b/10b 인코딩 시키면 PCIe bus를 통해 데이터를 전달하게 된다. 하지만 이것은 PCIe Gen 1 기준으로 나타냈다. Gen 3만 보더라도 encoding scheme이 128b/130b로 늘어났기 때문이다. 전송되는 packet의 구성과 크기는 아래와 같다.



![img](https://blog.kakaocdn.net/dn/TcoBJ/btqA2brpMp7/sBYHrL4LPlZxCPI8fGBl8k/img.png)전달되는 packet의 구성



1. 1. Physical Layer
      - Framing
        - Start (1B)
        - End (1B)
   2. Data Link Layer
      - Sequence Number (2B)
      - LCRC (1DW)
   3.  Transaction Layer
      - Header (3~4DW)
      - Data (0~1024DW)
      - ECRC (1DW)

### **Transaction Layer**

Ttransaction layer에 발생되는 packet을 Transaction Layer Packets (TLP)라 부른다. TLP는 routing mechanism 및 규칙에 따라 하나의 링크에서 다른 링크로 전달된다. 아래는 TLP가 접근하는 4개의 address spcae를 말한다. 즉, PCIe 디바이스에서 packets이 전달될때 아래의 분리된 주소 공간이 사용된다. (TLP header를 통해 구분한다)

1. 1. Memory
      - Transaction types: Read, Write 
      - Purpose: transfter data to or from a location in the system memory map
   2. I/O
      - Transaction types: Read, Write
      - Purpose: Transfer data to or from a location in the system IO map
   3. Configuration
      - Transaction types: Read, Write
      - Purpose: Transfer data to or from a location in the configuration space of a PIC-compatible device
   4. Message
      - Transaction types: baseline, vendor-specific
      - Purpose: general in-band messaging and even reporting without consuming memory or IO address resoures



![img](https://blog.kakaocdn.net/dn/xyJxD/btqA2bEYzsb/jmKkJF0Vf5DBpBkPBTU7bK/img.png)TLP Header



 

### **Data Link Layer**

- 링크 트레이닝
  - 링크 폭
  - 링크 데이터 rate
  - Lane reversal (Lane 역 방향 순서 지원)
  - Polarity inversion (극성 반대 지원)
  - 멀티 레인에서 레인 to 레인 de‐skew
- 전원 관리
  - 트랜잭션 게층에서 요청한 전원 상태를 수락하고 물리 계층으로 전달한다.
  - 활성/재설정/연결 해제/전원 관리 상태를 트랜잭션 계층으로 전달한다.
- 데이터 프로텍션, 에러 체킹
  - CRC 생성
  - 에러 체킹
  - 재전송 메시리지대한 TLP 응답
  - 에러 리포팅 및 로깅을 위한 에러 인디케이션

### **Physical Layer**

- 인터페이스 초기화, 메인트넌스 제어 및 스테이터스 트래킹
  - 리셋/핫플러그 컨트롤
  - 전원 관리 인터커넥트
  - width & lane 매핑 협상
  - lane 극성 반전
- 심볼 및 오더
  - 8b/10b 엔코딩 및 디코딩
  - 임베디드 클럭 튜닝 및 정렬
- 심볼 전송 및 정렬
  - 전송 회로
  - 리셉션 회로
  - 수신 측 Elastic 버퍼
  - 수신 측 멀티 lane de-skew
