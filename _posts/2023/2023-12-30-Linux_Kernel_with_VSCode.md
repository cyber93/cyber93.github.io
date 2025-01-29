---
title:  "VSCode를 이용한 Linux Kernel 분석"
excerpt: "Linux Kernel 이야기"

toc: true
toc_sticky: true

categories:
  - Linux Kernel
tags:
  - VSCode
  - Linux Kernel 이야기
---

<br>

> 참고: https://joolib.tistory.com/18
>
> VSCode를 이용한 Linux Kernel 분석 방법에 대해 정리합니다.
>
> Extention Backup: https://github.com/cyber93/linux-kernel-vscode

Linux OS가 설치된 PC상에 VSCode를 설치하고 아래 소개된 Extention을 이용합니다.

# VSCode를 이용한 Linux Kernel 분석

## 1. C/C++

Linux Kernel은 수많은 파일들로 구성되어 있습니다. 

파일 내에 호출되는 **함수의 정의**로 이동하기 위해 C/C++ Extension을 설치합니다.

![img](\assets\images\kernel-vs1.png)

이것을 설치하면 아래 그림과 같이 정의로 이동 기능이 활성화됩니다.

![img](\assets\images\kernel-vs2.png)



## 2. C++ Intellisense & GNU Global

많은 아키텍처와 파일들로 구성되어 있기에 정확한 곳을 찾는 것이 중요합니다.

함수 정의로 이동과 함께 **함수의 참조**를 확인하는 기능을 추가하기 위해 C++ Intellisense Extenstion을 설치합니다.

![img](\assets\images\kernel-vs3.png)



이 Extension을 사용하기 위해서 Code Tagging 툴인 [GNU Global](https://www.gnu.org/software/global/global.html)을 설치합니다.

GNU Global을 설치한 후에 Linux Kernel이 설치된 폴더로 이동하여 다음 명령어를 입력합니다.

```
gtags
```

Tagging에 시간이 좀 걸린 후, GPATH, GRTAGS, GTAGS 파일이 생성됩니다.



VSCode를 재실행하며, 이 세 파일을 사용하여 함수의 참조를 알려 줍니다.

Peek Definition을 통해 다음과 같이 호출된 파일과 호출된 횟수를 확인할 수 있습니다.

![img](\assets\images\kernel-vs4.png)



> **global 명령**
>
> 
>
> **함수**를 찾기 위해서 global 명령어를 사용합니다.
>
> ```
> #global 함수이름
> global start_kernel
> ```
>
> 다음과 같은 결과를 출력합니다.
>
> ```
> arch/alpha/boot/bootp.c
> arch/alpha/boot/bootpz.c
> arch/alpha/boot/main.c
> init/main.c
> ```
>
>  
>
> **심볼 이름을 모르는 경우**, 다음과 같이 일부만 작성하여 확인할 수 있습니다.
>
>  -c는 해당 Prefix로 시작하는 심볼을 출력합니다.
>
> ```
> global -c kmem 
> ```
>
> 다음과 같은 결과를 출력합니다.
>
> ```
> kmem_alloc
> kmem_alloc_io
> kmem_alloc_large
> kmem_cache
> kmem_cache_alloc
> kmem_cache_alloc_bulk
> kmem_cache_alloc_node
> kmem_cache_alloc_node_trace
> kmem_cache_alloc_trace
> kmem_cache_cpu
> kmem_cache_create
> kmem_cache_create_usercopy
> ...
> kmemleak_seq_start
> kmemleak_seq_stop
> kmemleak_stop
> kmemleak_test_exit
> kmemleak_test_init
> kmemleak_update_trace
> kmemleak_vmalloc
> kmemleak_warn
> kmemleak_write
> ```
>



## 3. GitLens

Linux Kernel 분석에 필요한 Extention을 추가로 설치합니다.

![img](\assets\images\kernel-vs5.png)

코드에 마우스를 올리면 커밋 내용을 볼 수 있습니다.

![img](\assets\images\kernel-vs6.png)
