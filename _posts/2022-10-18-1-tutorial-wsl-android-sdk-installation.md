---
title:  "[Tutorial][Android] 221018 WSL Ubuntu 20.04 Android SDK 설치"
excerpt: "Android에 Mediapipe 탑재하기(3)"

categories:
  - Tutorial
tags:
  - Tutorial
  - Android
  - SDK
  - WSL
  - Ubuntu
last_modified_at: 2022-10-18T20:27:00
---

Ubuntu 20.04에 Android SDK를 설치하는 방법입니다. 저는 Android SDK에 포함된 ADB(Android Debug Bridge)를 사용하기 위해 설치했습니다.

## (1) SDK를 설치할 폴더를 생성합니다.

{% highlight shell linenos %}
sudo mkdir -p Android/cmdline-tools
{% endhighlight %}

## (2) Google Developer의 Android Studio 페이지(<a herf="https://developer.android.com/studio#command-tools">링크</a>)에서 Linux용 커맨드 라인 툴을 다운로드 받습니다.

<p><img src="/assets/images/22101801.png" /></p>

## (3) zip 파일을 압축 해제합니다.

{% highlight shell linenos %}
sudo unzip {파일명}
{% endhighlight %}

<p><img src="/assets/images/22101803.png" /></p>

## (4) 압축해제해서 생긴 cmdline-tools 폴더를 1번에서 생성한 디렉토리에 tools라는 이름으로 복사합니다.

{% highlight shell linenos %}
sudo cp -r cmdline-tools Android/cmdline-tools/tools
{% endhighlight %}

<p><img src="/assets/images/22101804.png" /></p>

## (5) 이 경로를 ~/.bashrc 파일에 path로 등록해줍니다.

{% highlight shell linenos %}
sudo vi ~/.bashrc
{% endhighlight %}

파일 가장 아래에 아래와 같이 작성하고 저장합니다.

{% highlight shell linenos %}
export JAVA_HOME={자바 경로}
export ANDROID_HOME={1에서 생성한 Android 파일 경로}
PATH="$JAVA_HOME/bin:$ANDROID_HOME/cmdline-tools/tools/bin:$PATH"
{% endhighlight %}

<p><img src="/assets/images/22101806.png" /></p>

그리고 저장한 파일을 적용합니다.

{% highlight shell linenos %}
source ~/.bashrc
{% endhighlight %}

## (6) 아래 명령어로 SDK Manager가 제대로 설치되었는지 확인합니다.

{% highlight shell linenos %}
sdkmanager --version
{% endhighlight %}

<p><img src="/assets/images/22101807.png" /></p>

## (7) 아래 명령어를 친 뒤 y를 눌러서 Android 약관에 동의합니다.

{% highlight shell linenos %}
sdkmanager --licenses
{% endhighlight %}

## (8) (필요한 경우) ANDROID_HOME의 권한을 허용합니다.

{% highlight shell linenos %}
sudo chown -R {유저명} $ANDROID_HOME
{% endhighlight %}

<p><img src="/assets/images/22101808.png" /></p>

## (9) Platform tool을 설치합니다. 버전은 본인 컴퓨터의 개발 환경에 맞춰서 바꿔주세요. 저는 33버전을 설치했습니다. 

{% highlight shell linenos %}
sdkmanager "platform-tools" "platforms;android-{안드로이드 버전}"
{% endhighlight %}

<p><img src="/assets/images/22101809.png" /></p>

> 현재 어떤 버전을 설치 할 수 있는 지 알고 싶다면 sdkmanager --list 명령어를 실행하세요.

## (10) Build tool을 설치합니다. 마찬가지로 버전은 개발 환경에 맞춰서 바꿔서 설치합니다.

{% highlight shell linenos %}

sdkmanager "build-tools;{Build tool 버전}"
{% endhighlight %}

<p><img src="/assets/images/22101810.png" /></p>

ANDROID_HOME으로 설정한 경로로 들어가면 필요한 툴들이 모두 설치되어있음을 확인할 수 있습니다.

<p><img src="/assets/images/22101811.png" /></p>

## (11) ~/.bashrc 파일에 platform tool 경로를 추가합니다.

{% highlight shell linenos %}
sudo vi ~/.bashrc
{% endhighlight %}

PATH를 아래와 같이 수정합니다.

{% highlight shell linenos %}
export JAVA_HOME={자바 경로}
export ANDROID_HOME={1에서 생성한 Android 파일 경로}
PATH="$JAVA_HOME/bin:$ANDROID_HOME/cmdline-tools/tools/bin:$ANDROID_HOME/platform-tools:$PATH"
{% endhighlight %}

<p><img src="/assets/images/22101812.png" /></p>

그리고 저장한 파일을 적용합니다.

{% highlight shell linenos %}
source ~/.bashrc
{% endhighlight %}

## (12) Android SDK 설치가 완료되었습니다.