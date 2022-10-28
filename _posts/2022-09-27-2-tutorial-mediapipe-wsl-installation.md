---
title:  "[Tutorial][Bazel] 220927 WSL Ubuntu 20.04에 Mediapipe 설치하기"
excerpt: "Android에 Mediapipe 탑재하기(2)"

toc: true
toc_sticky: true

categories:
  - Tutorial
tags:
  - Tutorial
  - Mediapipe
  - WSL
  - Ubuntu
last_modified_at: 2022-09-27T21:40:00
---

Mediapipe 공식 홈페이지의 안내에 따라 WSL에 Mediapipe를 설치합니다.<br>
(참조: <a href="https://google.github.io/mediapipe/getting_started/install.html#installing-on-debian-and-ubuntu">https://google.github.io/mediapipe/getting_started/install.html#installing-on-debian-and-ubuntu</a>)

## (1) Mediapipe Repository를 클론합니다.

{% highlight shell linenos %}
git clone https://github.com/google/mediapipe.git
{% endhighlight %}

<p><img src="/assets/images/22092704.png" /></p>

## (2) 클론한 디렉토리로 이동합니다.

{% highlight shell linenos %}
cd mediapipe
{% endhighlight %}

## (3) 패키지 매니저를 이용해 컴파일된 OpenCV 라이브러리를 설치합니다.

{% highlight shell linenos %}
sudo apt-get install -y \
    libopencv-core-dev \
    libopencv-highgui-dev \
    libopencv-calib3d-dev \
    libopencv-features2d-dev \
    libopencv-imgproc-dev \
    libopencv-video-dev
{% endhighlight %}

<p><img src="/assets/images/22092705.png" /></p>

## (4) 제대로 설치되었는지 확인하기 위해 Mediapipe에서 제공하는 Hello World를 빌드합니다.

{% highlight shell linenos %}
bazel run --define MEDIAPIPE_DISABLE_GPU=1 mediapipe/examples/desktop/hello_world:hello_world
{% endhighlight %}

<p><img src="/assets/images/22092706.png" /></p>

아래처럼 Hello World가 나타나면 정상적으로 설치가 완료된 것입니다.

<p><img src="/assets/images/22092707.png" /></p>
