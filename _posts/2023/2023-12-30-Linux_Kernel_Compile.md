---
title:  "Linux Kernel Compile (리눅스 커널 컴파일)"
excerpt: "Linux Kernel 이야기"

toc: true
toc_sticky: true

categories:
  - Linux Kernel
tags:
  - vscode
  - Linux Kernel 이야기
---


이 포스트에서는 우분투에서 리눅스 커널을 컴파일 하는 법에 대하여 정리하였습니다. 커널 컴파일을 하는 경우는 보통, 해당 우분투의 커널의 버전을 바꾸거나, 커널을 수정한 뒤 그것을 반영하기 위해 하는 등이 있을 것 같습니다. 이 포스트에서는 커널 버전을 바꿔보도록 하겠습니다.  

## 현재 커널 버전 확인
 우분투에서는 다음 명령어를 통해 현재 커널의 버전을 확인할 수 있습니다.  
```
$ uname -r
```
저의 경우 Ubuntu 20.04.1 LTS를 설치했을 때 기본으로 제공되는 5.4.0-47-generic 입니다.  
![Linux_kernel_compile_image1]({{site.url}}/assets/images/20200913/Linux_kernel_compile_1.jpg)  

## 설치할 커널 소스 다운로드
[리눅스 커널 홈페이지](ttps://www.kernel.org/)에서 원하는 커널 버전을 다운로드 합니다. 이 예제에서는 [목록에서](https://kernel.org/pub/linux/kernel/v5.x/linux-5.4.59.tar.gz) 5.4.59 버전을 다운 받았습니다.  
![Linux_kernel_compile_image2]({{site.url}}/assets/images/20200913/Linux_kernel_compile_2.jpg)  

## 다운로드한 커널 소스 이동 및 압축 해제
다운로드한 커널 소스를 **'/usr/src'** 위치로 옮겨준 뒤, 압축을 해제합니다. ( root user로 진행했습니다.)  
![Linux_kernel_compile_image3]({{site.url}}/assets/images/20200913/Linux_kernel_compile_3.jpg)  

## 컴파일 준비
압축 해제한 폴더로 이동해서 다음과 같이 컴파일에 필요한 준비를 진행합니다.  

### 0. 필요한 패키지 및 툴 설치
다음 명령어를 통해 필요한 패키지와 툴들을 설치합니다.  
```
$ sudo apt-get update
$ sudo apt-get install build-essential libncurses5 libncurses5-dev bin86 kernel-package libssl-dev bison flex libelf-dev
```

### 1. 커널 Configuration 파일(.config) 생성
 저는 현재 커널의 Configuration 파일을 복사해서 사용하겠습니다( 새로 만들어도 되지만 옵션 설정이 귀찮기 때문에.. ). 현재 커널의 configuration 파일은 **'/boot/{현재 커널 버전}'** 으로 저장되어 있습니다. 이것을 다음과 같은 명령어로 새롭게 컴파일할 커널의 폴더(위에서 알집해제한 것)에 .config라는 이름으로 복사합니다.  
 ```c
 $ cp /boot/config-5.4.0-47-generic ./.config
 ```

### 2. 생성한 Configuration 파일로 커널 설정
  위에서 생성한 configuration 파일을 기반으로 새롭게 설치할 커널의 설정을 진행해 줍니다. 아래 명령어를 사용합니다.  
  ```
  $ sudo make menuconfig
  ```
  > 처음 커널 컴파일하는 경우 이 부분에서 많은 에러가 날 수 있습니다. 하지만 대부분 package나 dependence 한 것들이 없어서 나는 것($ ~~~ : not found) 일 것이니 직접 구글링을 통해 해결하리라 믿습니다. 커널 아마 대부분 sudo apt-get install {에러난것} 으로 설치하면 해결 될 것입니다.  


그러면 다음과 같은 화면을 보실 수 있습니다.  
![Linux_kernel_compile_image4]({{site.url}}/assets/images/20200913/Linux_kernel_compile_4.jpg)  

해당 화면에서 **load** 를 선택하면 아래 화면이 나오고, **ok** 를 선택한 뒤, **Exit**, **Yes** 를 통해 나오시면 됩니다.  
![Linux_kernel_compile_image5]({{site.url}}/assets/images/20200913/Linux_kernel_compile_5.jpg)  

## 커널 컴파일 진행
이제 준비는 끝났고 실제 컴파일을 진행할 차례입니다.  

### 1. CPU 코어 갯수 확인
커널 컴파일은 오랜시간이 걸리기 때문에, 빠른 빌드를 위해 현재 사용하고 있는 컴퓨터의 CPU코어 갯수를 확인하고 최대로 사용하겠습니다. 아래 명령어를 통해 확인하시면 됩니다.  
```
grep -c processor /proc/cpuinfo
```
저는 아래 그림과 같이 8개 입니다.  
![Linux_kernel_compile_image6]({{site.url}}/assets/images/20200913/Linux_kernel_compile_6.jpg)  

### 2. 커널 컴파일에 필요한 툴 설치
사실 에러가 날때마다 바로바로 찾아서 추가했던지라 커널 컴파일에 관련된 툴과 패키지는 어떤 것들이 있는지 정확히 모릅니다. 제가 참고한 다른 블로그에서 아래 것들을 설치하고 진행하라고 하길래 추가해 봅니다.  
  ```
sudo apt-get install ncurses-dev libssl-dev kernel-package
  ```
> 만약 configuration과 관련된 선택 사항이 나온다면 아마 local configuration 사용하는 것으로 선택하시면 될 것 입니다.

### 3. 커널 컴파일 실행
 커널 컴파일에는 두가지 방법이 있는 것을 알고 있습니다. 하나는 **make-kpkg**를 통해 커널 이미지를 만들고 이를 **dpkg**를 통해 등록하는법. 다른 하나는 **make modules**, **make modules_install**, **make install** 순으로 진행시켜 커널을 컴파일하고 자동으로 등록하는 법 입니다. 리눅스 커널관련 공부를 시작할 때 많이 쓰는 팽귄책에서는 후자를 사용했던 것 같은데 저는 전자를 사용해 보도록 하겠습니다. (제가 참고한 블로그 중 하나인 [3]에서 후자 방법을 사용합니다.)  

위에서 확인한 코어 개수를 사용하여 아래 명령어를 통해 커널을 컴파일 해줍니다.  
 ```
 $ sudo make-kpkg -j{#} --initrd --revision=1.0 kernel_image
 ```

 {#} 이 부분에 코어 갯수를 넣어주시면 됩니다. 저는 8개이기 때문에 아래 그림과 같이 했고 컴파일이 시작되는 것을 볼 수 있습니다.  
![Linux_kernel_compile_image7]({{site.url}}/assets/images/20200913/Linux_kernel_compile_7.jpg)  
 > --revision=1.0 은 만드는 커널 이미지의 유저의 버전을 정할 수 있습니다. 보통 커널을 수정하여 사용할 때 각 커널이 다름을 표시하기 위해 사용합니다.  

완료되면 아래와 같은 화면을 보실 수 있고,
![Linux_kernel_compile_image8]({{site.url}}/assets/images/20200913/Linux_kernel_compile_8.jpg)  

상위 폴더 (저희의 경우 **/usr/src**에서 다음과 같은 커널 이미지(.deb 파일)을 보실 수 있습니다.
![Linux_kernel_compile_image9]({{site.url}}/assets/images/20200913/Linux_kernel_compile_9.jpg)  

### 4. 커널 이미지 설치(등록)
커널 이미지가 있는 폴더에서(**/usr/src**) 다음 명령어를 실행해 주시면 됩니다.  
```
$ sudo dpkg -i {커널이미지파일명}
```
저의 경우는 다음과 같습니다.  
![Linux_kernel_compile_image10]({{site.url}}/assets/images/20200913/Linux_kernel_compile_10.jpg)  
> 아마 출력문이 다르실 것 입니다. 이 이미지는 명령어 사용 예제로만 보시고, 실제 사용시 마지막에 done로 종료되는 것만 보시면 됩니다!  

### 5. Reboot
지금까지 새로운 커널 설치를 완료했습니다. 정상적으로 설치가 되었다면 재부팅한 위에 위에서와 같은 방식으로 커널의 버전을 확인했을 때 커널 버전이 바뀐 것을 확인할 수 있습니다. 저의 경우는 다음과 같이 바뀐 것을 확인할 수 있었습니다.  
![Linux_kernel_compile_image11]({{site.url}}/assets/images/20200913/Linux_kernel_compile_11.jpg)  

## Conclusion
 이 포스트는 리눅스의 컴파일을 다루어 보았습니다. 학부 연구생 시절 커널 관련해서 제일 처음 해봤던 것인데, 간단하지만 뭔가에 막혀 되게 오래 걸렸던 기억이 있습니다. 꼭 블로그를 만들게 되면 이것을 처음 포스트로 하자 했었는데 이렇게 이루게 되어 기쁩니다ㅎㅎ 
 다음번엔 커널을 공부하며 가지고 놀아볼 때 대부분 가장 처음으로 해보는 커널에 systemcall 추가를 다루어 볼까 합니다.  

## Reference
포스트를 작성하면서 참고한 블로그들을 레퍼런스합니다.  

[1] <https://harryp.tistory.com/839>
[2] <https://marcokhan.tistory.com/246>
[3] <https://5equal0.tistory.com/entry/Linux-Kernel-Kernel-%EB%B9%8C%EB%93%9C-%EB%B0%8F-%EC%84%A4%EC%B9%98>
