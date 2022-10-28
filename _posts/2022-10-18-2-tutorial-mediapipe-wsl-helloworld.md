---
title:  "[Tutorial][Bazel-MediaPipe] 221018 WSL Ubuntu 20.04 Bazel MediaPipe - Hello world"
excerpt: "Android에 Mediapipe 탑재하기(4)"

toc: true
toc_sticky: true
categories:
  - Tutorial
tags:
  - Tutorial
  - Android
  - Bazel
  - Mediapipe
  - Helloworld
  - WSL
  - Ubuntu
last_modified_at: 2022-10-18T21:40:00
---

이번 시간에는 Mediapipe 공식 홈페이지에 나와있는 Hello world 예제를 하나하나 따라하며 Bazel 빌드까지 완료합니다. MediaPipe 공식 홈페이지 Android용 튜토리얼은 아래 링크를 통해 확인할 수 있습니다.<br>
링크: <a href="https://google.github.io/mediapipe/getting_started/hello_world_android.html">https://google.github.io/mediapipe/getting_started/hello_world_android.html</a>

## (1) 프로젝트 준비

프로젝트를 진행하기 위해서 아래와 같이 환경이 구성되어 있어야 합니다.
1. Bazel 설치(<a href="/tutorial/1-tutorial-bazel-wsl-installation/">링크</a>)
2. MediaPipe 설치(<a href="/tutorial/1-tutorial-mediapipe-wsl-installation/">링크</a>)
3. Java 설치 및 JAVA_HOME 설정
4. Andriod SDK 설치 - ADB 사용(<a href="/tutorial/1-tutorial-wsl-android-sdk-installation/">링크</a>)

## (2) Workspace 만들기

(2-1) 프로젝트 폴더를 만듭니다. 저는 편의를 위하여 폴더명을 "mediapipe-hello-world"로 정했습니다.

{% highlight shell linenos %}
sudo mkdir mediapipe-hello-world
{% endhighlight %}

<p><img src="/assets/images/22101601.png" /></p>

(2-2) 이전에 MediaPipe를 설치할 때 MediaPipe GitHub Repository를 클론해둔 곳으로 이동합니다. Mediapipe는 Workspace 구성에 필요한 파일과 소스를 공식 Repository에서 제공합니다. 필요한 파일과 폴더를 프로젝트 폴더로 옮깁니다. 필요한 파일/폴더 목록은 아래에 정리했습니다.

|파일명|설명|
|:--|:--|
|thrid_party|빌드 스크립트를 모아둔 폴더.|
|.bazelrc|Run Command 파일. 빌드 시 자동 실행될 스크립트.|
|.bazelversion|빌드에 사용할 버전을 명시한 파일.|
|.gitignore|(선택) Git 업로드 시 무시할 파일 목록.|
|LICENSE|라이선스 관련 파일.|
|requirements.txt|빌드에 필요한 Python 패키지를 명시한 파일.|
|setup.py|빌드에 필요한 개발 환경을 구축해주는 Python 파일.|
|setup_android_sdk_and_ndk.sh|Android SDK와 NDK를 설정하는 쉘스크립트 파일|
|setup_opencv.sh|OpenCV를 설정하는 쉘스크립트 파일|
|WORKSPACE|개발에 필요한 패키지를 자동으로 다운로드 받아 개발 환경을 구축해주는 파일|

<p><img src="/assets/images/22101602.png" /></p>

(2-3) 안드로이드 SDK와 NDK 개발환경을 설정합니다. 환경 설정에는 경로 내의 setup_android_sdk_and_ndk.sh	파일을 사용합니다.

{% highlight shell linenos %}
bash setup_android_sdk_and_ndk.sh
{% endhighlight %}

<p><img src="/assets/images/22101604.png" /></p>

<p><img src="/assets/images/22101605.png" /></p>

## (3) 프로젝트 구조 만들기

(3-1) 프로젝트 폴더에 "app" 폴더를 만듭니다.

{% highlight shell linenos %}
sudo mkdir app
{% endhighlight %}

<p><img src="/assets/images/22101606.png" /></p>

(3-2) "app" 폴더 안에 "java" 폴더와 "res" 폴더를 생성합니다.

{% highlight shell linenos %}
cd app
sudo mkdir java res
{% endhighlight %}

<p><img src="/assets/images/22101607.png" /></p>

(3-3) "java" 폴더 안에는 "com/example/mediapipe/helloworld" 경로를 생성합니다.

{% highlight shell linenos %}
cd java
sudo mkdir -p com/example/mediapipe/helloworld
{% endhighlight %}

<p><img src="/assets/images/22101608.png" /></p>

(3-4) "res" 폴더 안에는 "layout" 폴더와 "values" 폴더를 생성합니다.

{% highlight shell linenos %}
cd ..
cd res
sudo mkdir layout values
{% endhighlight %}

<p><img src="/assets/images/22101609.png" /></p>

## (4) 소스 코드 작성

(4-1) "app/java/com/example/mediapipe/helloworld" 경로 안에 "MainActivity.java"라는 파일 생성 후 아래 코드를 작성합니다.

{% highlight java linenos %}
package com.example.mediapipe.helloworld;

import android.os.Bundle;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
  }
}
{% endhighlight %}

(4-2) "app/res/layout" 경로 안에 "activity_main.xml"라는 파일 생성 후 아래 코드를 작성합니다.

{% highlight xml linenos %}
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

  <TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Hello World!"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintRight_toRightOf="parent"
    app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
{% endhighlight %}

(4-3) "app" 경로 안에 "AndroidManifest.xml"라는 파일 생성 후 아래 코드를 작성합니다.

{% highlight xml linenos %}
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.mediapipe.helloworld">

  <uses-sdk
      android:minSdkVersion="19"
      android:targetSdkVersion="31" />

  <application
      android:allowBackup="true"
      android:label="${appName}"
      android:supportsRtl="true"
      android:theme="@style/AppTheme">
      <activity
          android:name="${mainActivity}"
          android:exported="true"
          android:screenOrientation="portrait">
          <intent-filter>
              <action android:name="android.intent.action.MAIN" />
              <category android:name="android.intent.category.LAUNCHER" />
          </intent-filter>
      </activity>
  </application>

</manifest>
{% endhighlight %}

(4-4) "app" 경로 안에 "BUILD"라는 파일 생성 후 아래 코드를 작성합니다. "BUILD" 파일은 빌드 시 타겟을 명시해주는 파일입니다.

{% highlight shell linenos %}
android_library(
    name = "basic_lib",
    custom_package = "com.example.mediapipe.helloworld",
    srcs = ["java/com/example/mediapipe/helloworld/MainActivity.java"],
    manifest = "AndroidManifest.xml",
    resource_files = glob(["res/**"]),
    deps = [
        "//third_party:androidx_constraint_layout",
        "//third_party:androidx_appcompat",
    ],
)

android_binary(
    name = "helloworld",
    custom_package = "com.example.mediapipe.helloworld",
    manifest = "AndroidManifest.xml",
    manifest_values = {
        "appName": "MediaPipe Hello World",
        "mainActivity": ".MainActivity",
    },
    multidex = "native",
    deps = [
        ":basic_lib",
    ],
)
{% endhighlight %}

(4-5) 마지막으로 아래 두 파일을 "app/res/values" 디렉토리에 생성합니다.

colors.xml
{% highlight xml linenos %}
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="colorPrimary">#008577</color>
    <color name="colorPrimaryDark">#00574B</color>
    <color name="colorAccent">#D81B60</color>
</resources>
{% endhighlight %}

styles.xml
{% highlight xml linenos %}
<resources>
    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>
</resources>
{% endhighlight %}

## (5) 앱 빌드

(5-1) 아래와 같이 명령어를 입력하여 앱을 빌드합니다.

{% highlight shell linenos %}
bazel build -c opt --define MEDAPIPE_DISABLE_GPU=1 //app:helloworld
{% endhighlight %}

<p><img src="/assets/images/22101611.png" /></p>

<p><img src="/assets/images/22101612.png" /></p>

## (6) 앱 설치

(6-1) 프로젝트 폴더에 "bazel-bin/app" 경로로 들어가면 helloworld.apk 파일을 확인할 수 있습니다.

<p><img src="/assets/images/22101613.png" /></p>

(6-2) bazel-bin/app 폴더로 이동한 뒤 아래 명령어로 모바일 디바이스에 앱을 설치합니다.

{% highlight shell linenos %}
adb install helloworld.apk
{% endhighlight %}

<p class="code"><img src="/assets/images/22101813.png" /></p>

(6-3) 아래처럼 BUILD 파일에 설정한 이름대로 앱이 설치된 것을 확인할 수 있고 설치된 앱을 실행해보면 Hellow World가 정상적으로 출력되는 것 또한 확인할 수 있습니다.

<img width="30%" src="/assets/images/22010221.jpg" /> <img width="30%" src="/assets/images/22010222.jpg" />