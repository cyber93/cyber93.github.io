---
title:  "VSCode를 이용한 Linux Kernel 분석"
excerpt: "Linux Kernel 이야기"

toc: true
toc_sticky: true

categories:
  - Linux Kernel
tags:
  - vscode
  - Linux Kernel 이야기
---



리눅스 커널은 수많은 파일들로 구성되어 있습니다. vscode로 열어보면 평소에 사용하던 '함수의 정의로 이동'이 보이지 않습니다.

tag를 이용한 방법을 소개합니다.

------

**GNU Global**

 

정의를 찾기 위해 vscode의 C/C++ extension을 설치합니다.

![img](\assets\images\kernel-vs1.png)

이것을 설치하면 다음과 같이 정의로 이동 기능이 활성화된 것을 확인할 수 있습니다.

![img](\assets\images\kernel-vs2.png)

정의로 이동과 함께 추가해야 할 기능이 더 있습니다.

함수의 reference를 확인하는 기능을 추가해야 합니다. 많은 아키텍처와 파일들로 구성되어 있기에 정확한 곳을 찾는 것이 중요합니다. 

![img](\assets\images\kernel-vs3.png)

이 extension을 사용하기 위해서 global gnu를 설치합니다.

[www.gnu.org/software/global/globaldoc_toc.html](https://www.gnu.org/software/global/globaldoc_toc.html)

 

Tutorial

Specify the mapping of language names and plug-in parsers. Each part delimited by the comma consists of a language name, a colon, the shared object path, an optional colon followed by a function name. If the function name is not specified, ’parser’ is

www.gnu.org

설치를 하는 경우 Permission denied가 발생할 수 있습니다. sudo를 붙여도 전체에 권한을 주는 것은 위험하다며 오류를 반환합니다.

```
Error: Running Homebrew as root is extremely dangerous and no longer supported.
```

이와 같은 경우 폴더에 권한을 주어 해결합니다.

 

로그인 계정이 admin 이고 설치하고자 하는 경로가 /usr/local/bin/global 라면 다음과 같이 권한을 부여합니다.

 

```
sudo chown -R admin /usr/local/bin/global
```

 

gnu global을 설치한 후에 linux 폴더로 이동하여 다음 명령어를 입력합니다.

```
gtags
```

GPATH, GRTAGS, GTAGS 파일이 생성됩니다.

 

vscode는 이 세 파일을 사용하여 reference를 알려줍니다. 설치한 후에 vscode를 재실행합니다.

Peek Definition을 통해 다음과 같이 호출된 파일과 호출된 횟수를 확인할 수 있습니다.

![img](\assets\images\kernel-vs4.png)

두 extension과 gnu global을 설치하여 리눅스 커널을 vscode와 연동했습니다.

 

gnu global 홈페이지에서 API를 제공합니다. terminal을 사용하여 정보를 확인할 수 있습니다.

 

예를 들어, 함수를 찾기 위해서 global 명령어를 사용합니다.

```
global start_kernel		//global (함수 이름)
```

다음과 같은 결과를 출력합니다.

```
arch/alpha/boot/bootp.c
arch/alpha/boot/bootpz.c
arch/alpha/boot/main.c
init/main.c
```

 

심볼 이름을 까먹은 경우 다음과 같이 일부만 작성하여 전체를 확인합니다. -c는 complete입니다.

```
global -c kmem 
```

닫기

```
kmem_alloc
kmem_alloc_io
kmem_alloc_large
kmem_cache
kmem_cache_alloc
kmem_cache_alloc_bulk
kmem_cache_alloc_node
kmem_cache_alloc_node_trace
kmem_cache_alloc_trace
kmem_cache_cpu
kmem_cache_create
kmem_cache_create_usercopy
kmem_cache_debug
kmem_cache_debug_flags
kmem_cache_destroy
kmem_cache_double_free
kmem_cache_flags
kmem_cache_free
kmem_cache_free_bulk
kmem_cache_has_cpu_partial
kmem_cache_init
kmem_cache_init_late
kmem_cache_invalid_free
kmem_cache_node
kmem_cache_oob
kmem_cache_open
kmem_cache_order_objects
kmem_cache_release
kmem_cache_sanity_check
kmem_cache_shrink
kmem_cache_size
kmem_cache_zalloc
kmem_config
kmem_flags_convert
kmem_free
kmem_freepages
kmem_getpages
kmem_rcu_free
kmem_realloc
kmem_test
kmem_zalloc
kmem_zalloc_large
kmem_zone
kmem_zone_t
kmemdup
kmemdup_nul
kmemleak_alloc
kmemleak_alloc_percpu
kmemleak_alloc_phys
kmemleak_alloc_recursive
kmemleak_boot_config
kmemleak_clear
kmemleak_disable
kmemleak_do_cleanup
kmemleak_erase
kmemleak_free
kmemleak_free_part
kmemleak_free_part_phys
kmemleak_free_percpu
kmemleak_free_recursive
kmemleak_ignore
kmemleak_ignore_phys
kmemleak_init
kmemleak_late_init
kmemleak_load_module
kmemleak_no_scan
kmemleak_not_leak
kmemleak_not_leak_phys
kmemleak_object
kmemleak_open
kmemleak_scan
kmemleak_scan_area
kmemleak_scan_thread
kmemleak_seq_next
kmemleak_seq_show
kmemleak_seq_start
kmemleak_seq_stop
kmemleak_stop
kmemleak_test_exit
kmemleak_test_init
kmemleak_update_trace
kmemleak_vmalloc
kmemleak_warn
kmemleak_write
```



------

 

커널 분석에 필요한 플러그인을 추가로 설치합니다.

![img](\assets\images\kernel-vs5.png)

코드에 마우스를 올리면 커밋 내용을 볼 수 있습니다.

![img](\assets\images\kernel-vs6.png)
