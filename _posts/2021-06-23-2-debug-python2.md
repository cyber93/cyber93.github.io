---
title:  "[Debug][Python] 210623 startJVM() got an unexpected keyword argument 'convertStrings'"
excerpt: "ML 기반 챗봇 Javas 개발기2"

toc: false
toc_sticky: false

categories:
  - Debug
tags:
  - Debug
  - Python
  - Konlpy
  - Javas
last_modified_at: 2021-06-23T10:36:00
---

<br>
ML 기반 챗봇 개발 중 채팅 메시지 분석을 위해 한글 토큰화 모듈인 Konlpy를 설치했습니다. 
설치 후 사용하는 데 아래와 같은 오류가 발생했습니다.

<p class="error_msg">startJVM() got an unexpected keyword argument 'convertStrings'</p>

해결방법은 Lib\site-packages\konlpy 폴더의 jvm.py 파일에서 convertString=True 를 주석처리 하는 것입니다.
해당 파일의 위치는 개발 환경마다 다를 수 있습니다.
저는 Windows에서 가상환경을 사용하여 개발 중이었고 해당 폴더를 아래 경로에서 발견하여 수정했습니다.

<p class="error_msg">venv\Lib\site-packages\konlpy\jvm.py</p>
