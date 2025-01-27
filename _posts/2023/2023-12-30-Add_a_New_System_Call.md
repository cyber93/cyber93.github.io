---
title:  "Add a New System Call (시스템콜 추가하기)"
excerpt: "Linux Kernel 이야기"

toc: true
toc_sticky: true

categories:
  - NVDIMM Storage Technology
tags:
  - System call
  - Linux Kernel 이야기
---

 이 포스트는 새로운 user system call을 정의하고 그것을 리눅스 커널에 추가하는 과정을 다룹니다. 

 커널을 공부하고 가지고 놀아볼 때 보통 가장 처음으로 해보는 것이 시스템 콜 구현입니다. 하지만 대부분의 커널 책이 지금 사용하는 커널 보다 낮은 커널 버전을 기준으로 하고 있어 최신 커널 버전에서의 시스템 콜 구현을 정리해 보고자 합니다. 이 포스팅에서는 저번 포스팅에서 컴파일 했던  linux kernel 5.4.59을 사용하여 진행합니다.  

## Introduction
 Linux Kernel에 System call 추가는 다음과 같은 것들을 수행해야합니다.  
 * 시스템 콜 함수 구현 (Define a new system call)  
 * 시스템 콜의 Makefile에 등록 (Update Makefile)  
 * 시스템 콜 테이블에 등록 (Update syscall Table)  
 * 시스템 콜 헤더파일에 등록 (Update system call header file)  
 * 커널 컴파일 및 재부팅 (Kernel Compile)  

위 과정을 하나씩 보여드리도록 하겠습니다.  

## 시스템 콜 함수 구현 (Define a new system call)  

 새로 추가하려는 시스템 콜이 호출되었을 때 어떤 작업을 할 지 정의하도록 하겠습니다.   
 시스템 콜 함수는 컴파일할 커널 소스코드의 특정 위치(**/usr/src/linux-{커널버전}/kernel**)에 구현을 해야합니다. 저의 경우 **/usr/src/linux-5.4.59/kernel** 이 되겠습니다. 해당 위치에 가보시면 아래와 같이 여러 시스템 콜들의 구현을 확인할 수 있습니다.  
 > 사실 이 위치에 구현되어 있는 makefile을 이용하여 쉽게 추가하기 위해서이지 '무조건 여기다!'는 아닙니다. 하지만 유저가 새로 만드는 시스템 콜은 이 위치에 추가하는게 일반적입니다.  

![syscall_define_1]({{site.url}}/assets/images/syscall_define_200919/syscall_define_1.jpg)

해당 위치에 저는 sys_hello()이라는 새로운 시스템을 구현하도록 하겠습니다.  
```
$ vi sys_hello.c

#include <linux/kernel.h>

asmlinkage long __sys_hello(void)
{
        printk("Hello world\n");
        return 0;
}
```
![syscall_define_2]({{site.url}}/assets/images/syscall_define_200919/syscall_define_2.jpg)  

위 그림에서 보시는 것 처럼 구현을 했고, 해당 시스템 콜이 호출되면 커널 메세지에 hello world를 출력해 줍니다. **asmlinkage** 는 어셈블리언어로 구현된 함수에서 호출할 수 있도록 해주는 키워드로 시스템 콜 구현에 필수적으로 들어가야 합니다.  
> printk 는 커널 메세지를 출력해주는 함수로, 커널 메세지는 **demsg**라는 커멘드로 확인할 수 있습니다. 후에 테스트에서 보도록 하겠습니다.  

## 시스템 콜의 Makefile에 등록 (Update Makefile)
 위에서 구현한 새로운 시스템 콜(sys_hello.c)을 **`/usr/src/linux-5.4.59/kernel/Makefile`**[^1] 에 등록하여 다른 시스템 콜들과 함께 컴파일 될 수 있도록 해주어야 합니다. 해당 파일을 여신뒤, 오브잭트 파일이 나열된 곳에 추가할 추가할 시스템 콜을 추가해주면 됩니다. 구현한 시스템 콜이 **`sys_hello.c'** 이기 때문에 **'sys_hello.o'** 라고 추가했습니다.  
![syscall_define_3]({{site.url}}/assets/images/syscall_define_200919/syscall_define_3.jpg)  

## 시스템 콜 테이블에 등록 (Update syscall Table)

 구현한 시스템 콜을 시스템 콜 테이블에 등록을 해주어야 합니다. 시스템 콜이 호출되면 커널은 이 시스템 콜 테이블을 통해 호출한 시스템 콜을 찾아 run해줍니다.  
 시스템 콜 테이블은 ``/usr/src/linux-5.4.59/arch/x86/entry/syscalls`` 위치에 있습니다.   

![syscall_define_4]({{site.url}}/assets/images/syscall_define_200919/syscall_define_4.jpg)  

 해당 파일을 열어보면 다양한 시스템 콜들을 볼 수 있습니다. 해당 파일은 ``<number> <abi> <name> <entry point>`` 순으로 구성되어 있습니다. 파일 주석 중에  
 >  don't use numbers 387 through 423, add new calls after the last 'common' entry
 라는 말이 있습니다. 저의 커널 버전에선 마지막 번호가 435번이기 때문에 436번에 추가하도록 하겠습니다. 추가는 다음과 같이 하면 됩니다.  
![syscall_define_5]({{site.url}}/assets/images/syscall_define_200919/syscall_define_5.jpg)

**이때 설정해준 시스템 콜 번호를 꼭 기억해야 합니다!**  

## 시스템 콜 헤더파일에 등록 (Update system call header file)  
 마지막으로 구현한 시스템 콜을 시스템 콜 헤더파일에 prototype 을 설정해 줍니다. 시스템 콜의 해더파일은 **/usr/src/linux-5.4.59/include/linux/syscalls.h**에 정의되어 있습니다.  
![syscall_define_6]({{site.url}}/assets/images/syscall_define_200919/syscall_define_6.jpg)
 해당 파일의 마지막에 있는 **#endif 앞에** 다음과 같이 자신의 함수 정의에 맞게 작성해 줍니다.  
![syscall_define_7]({{site.url}}/assets/images/syscall_define_200919/syscall_define_7.jpg)

## 커널 컴파일 및 재부팅 (Kernel Compile)
위 과정을 완료했으면 커널을 컴파일하여 등록한 뒤 재부팅을 해줍니다. 이 과정은 이전 포스팅에서 다루었음으로 생략하겠습니다. 이전에 컴파일한 커널 이미지와 중복을 피하기 위해 revision만 2로 변경했습니다.  
![syscall_define_8]({{site.url}}/assets/images/syscall_define_200919/syscall_define_8.jpg)
> 이미 컴파일한 커널 이미지가 있다면 다른 방법을 통해 간단하게 컴파일 할 수 있다고 알고 있습니다. 레퍼런스한 블로그 중 [3]에서 그렇게 하는데 저는 사용할 줄 몰라 다루지 않겠습니다.  

 컴파일이 완료되었다면 재부팅해줍니다.  


## Test
 위 과정을 통해 구현하고 새롭게 추가한 시스템 콜을 실행해 보겠습니다.  
 테스트는 아래 코드를 이용해서 진행했습니다.  
```c
$ vi syscall_test.c

#include <stdio.h>
#include <linux/kernel.h>
#include <sys/syscall.h>
#include <unistd.h>
int main()
{
         long int amma = syscall(436);
         printf("System call sys_hello returned %ld\n", amma);
         return 0;
}
```
 커널 메세지를 확인한 후 위 **`syscall_test.c`**을 실행한 후 다시한번 확인해 보겠습니다.   
```
$ dmesg
```
![syscall_define_9]({{site.url}}/assets/images/syscall_define_200919/syscall_define_9.jpg)
```
$ a.out
```
![syscall_define_10]({{site.url}}/assets/images/syscall_define_200919/syscall_define_10.jpg)
```
$ dmesg
```
![syscall_define_11]({{site.url}}/assets/images/syscall_define_200919/syscall_define_11.jpg)


 구현한 시스템 콜이 올바르게 작동하는 것을 확인할 수 있었습니다.  

## Conclusion

 이번 포스팅에서는 5.x 버전의 커널에 간단한 시스템 콜을 구현하고 실행시켜봤습니다. 커널 메세지를 출력하는 간단한 코드를 사용했는데, 개인의 필요에 따라 이 부분을 바꿔주며 사용하면 될 것 같습니다.

## Reference
참고한 페이지를 참조합니다.  
[1] <https://medium.com/anubhav-shrimal/adding-a-hello-world-system-call-to-linux-kernel-dad32875872i>  
[2] <https://harryp.tistory.com/69>  
[3] <https://isme2n.github.io/devlog/2017/05/30/linux-systemcall-add/>  

[^1]: 본인의 커널 버전에 맞게 위치를 사용해주세요!
