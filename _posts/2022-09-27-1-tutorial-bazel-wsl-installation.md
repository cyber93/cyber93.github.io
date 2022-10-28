---
title:  "[Tutorial][Bazel] 220927 WSL Ubuntu 20.04에 Bazel 설치하기"
excerpt: "Android에 Mediapipe 탑재하기(1)"

toc: true
toc_sticky: true

categories:
  - Tutorial
tags:
  - Tutorial
  - Bazel
  - WSL
  - Ubuntu
last_modified_at: 2022-09-27T21:18:00
---

Bazel 공식 홈페이지의 안내에 따라 WSL에 Bazel을 설치합니다.<br>
(참조: <a href="https://docs.bazel.build/versions/main/install-ubuntu.html">https://docs.bazel.build/versions/main/install-ubuntu.html</a>)

## (1) 패키지에 Bazel을 추가합니다.

{% highlight shell linenos %}
sudo apt install apt-transport-https curl gnupg
curl -fsSL https://bazel.build/bazel-release.pub.gpg | gpg --dearmor > bazel.gpg
sudo mv bazel.gpg /etc/apt/trusted.gpg.d/
echo "deb [arch=amd64] https://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
{% endhighlight %}

## (2) Bazel을 설치합니다.

{% highlight shell linenos %}
sudo apt update && sudo apt install bazel
{% endhighlight %}

<p><img src="/assets/images/22092701.png" /></p>

## (3) 최신 버전으로 업데이트합니다.

{% highlight shell linenos %}
sudo apt update && sudo apt full-upgrade
{% endhighlight %}

<p><img src="/assets/images/22092702.png" /></p>

## (4) 설치가 완료되었다면 아래 명령어로 Bazel이 제대로 설치되었는지 확인합니다.

{% highlight shell linenos %}
bazel --version
{% endhighlight %}

<p><img src="/assets/images/22092703.png" /></p>
