---
title:  "[Tutorial][Bazel-MediaPipe] 221028 WSL Ubuntu 20.04 Bazel MediaPipe Android App build"
excerpt: "Android에 Mediapipe 탑재하기(6)"

toc: true
toc_sticky: true
published: false
categories:
  - Tutorial
tags:
  - Tutorial
  - Android
  - Bazel
  - Mediapipe
  - WSL
last_modified_at: 2022-10-28T20:17:00
---

Android에 머신러닝 탑재하기 여섯번째 시간입니다. Android에 Mediapipe를 탑재하는 방법은 총 두가지입니다. 첫번째는 Mediapipe 공식 홈페이지에 나와있는 대로 Bazel을 사용하여 빌드하는 것이고 두번째는 Mediapipe로 aar(안드로이드 라이브러리)를 빌드하여 탑재하는 것입니다. 이번 시간에는 첫번째 방식으로 진행합니다. Mediapipe 공식 홈페이지의 Android용 튜토리얼을 참고했습니다. MediaPipe 공식 홈페이지 Android용 튜토리얼은 아래 링크를 통해 확인할 수 있습니다.<br>
링크: <a href="https://google.github.io/mediapipe/getting_started/hello_world_android.html">https://google.github.io/mediapipe/getting_started/hello_world_android.html</a>

## (1) 프로젝트 준비 - 필요한 사항 점검하기

프로젝트를 진행하기 위해서 아래와 같이 환경이 구성되어 있어야 합니다.
1. Bazel 설치(<a href="/tutorial/1-tutorial-bazel-wsl-installation/">링크</a>)
2. MediaPipe 설치(<a href="/tutorial/1-tutorial-mediapipe-wsl-installation/">링크</a>)
3. Java 설치 및 JAVA_HOME 설정
4. Python 3 설치 및 PYTHON_BIN_PATH 설정
4. Andriod SDK 설치 - ADB 사용(<a href="/tutorial/1-tutorial-wsl-android-sdk-installation/">링크</a>)

## (2) Workspace 만들기

(2-1) 프로젝트 폴더를 만듭니다. 저는 폴더명을 "mediapipe-edge-detection"으로 정했습니다.

{% highlight shell linenos %}
sudo mkdir mediapipe-edge-detection
{% endhighlight %}

<p><img src="/assets/images/22102801.png" /></p>

(2-2) 이전에 MediaPipe를 설치할 때 MediaPipe GitHub Repository를 클론해둔 곳으로 이동합니다. Mediapipe는 Workspace 구성에 필요한 파일과 소스를 공식 Repository에서 제공합니다. 필요한 파일과 폴더를 프로젝트 폴더로 옮깁니다. 필요한 파일/폴더 목록은 아래에 정리했습니다.

|파일명|설명|
|:--|:--|
|mediapipe/java/com/google/mediapipe/components|이번 프로젝트에 사용할 components들이 들어있는 폴더|
|mediapipe/java/com/google/mediapipe/framework|components를 사용하기 위해 필요한 폴더|
|mediapipe/java/com/google/mediapipe/BUILD|components를 사용하기 위해 필요한 BUILD 파일|
|mediapipe/graphs/edge_detection|이번 프로젝트에 사용할 edge_detection graph 폴더|
|mediapipe/calculators|graph를 사용하기 위해 필요한 폴더|
|mediapipe/util|graph를 사용하기 위해 필요한 폴더|
|mediapipe/gpu|graph를 사용하기 위해 필요한 폴더|
|mediapipe/BUILD|graph를 사용하기 위해 필요한 파일|
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

<p><img src="/assets/images/22102802.png" /></p>

(3-2) "app" 폴더 안에 "java" 폴더와 "res" 폴더를 생성합니다.

{% highlight shell linenos %}
cd app
sudo mkdir java res
{% endhighlight %}

<p><img src="/assets/images/22102803.png" /></p>

(3-3) "java" 폴더 안에는 "com/example/mediapipe/edgedetection" 경로를 생성합니다.

{% highlight shell linenos %}
cd java
sudo mkdir -p com/example/mediapipe/edgedetection
{% endhighlight %}

<p><img src="/assets/images/22102804.png" /></p>

(3-4) "res" 폴더 안에는 "layout" 폴더와 "values" 폴더를 생성합니다.

{% highlight shell linenos %}
cd ..
cd res
sudo mkdir layout values
{% endhighlight %}

<p><img src="/assets/images/22102805.png" /></p>

## (4) 소스 코드 작성 - 카메라 권한 허용

(4-1) 이번 프로젝트는 지난 번에 했던 Hello world 프로젝트(<a href="/tutorial/2-tutorial-mediapipe-wsl-helloworld/">링크</a>)를 확장하는 식으로 진행됩니다. 기존 파일이 없다면 다음 Repository에서 소스 코드를 확인할 수 있습니다. (<a href="https://github.com/younginshin115/mediapipe-hello-world">링크</a>)

아래 6개 파일을 각자 맞는 위치에 옮깁니다.

|파일명|위치|
|:--|:--|
|MainActivity.java|app/java/com/example/mediapipe/edgedetection|
|activity_main.xml|app/res/layout|
|AndroidManifest.xml|app|
|BUILD|app|
|colors.xml|app/res/values|
|styles.xml|app/res/values|

(4-2) "app" 경로 아래 "AndroidManifest.xml" 파일에 카메라 기능을 사용하기 위한 코드를 추가합니다.

{% highlight xml linenos %}
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.mediapipe.edgedetection"> <!-- package 명 수정 -->

  <!-- 카메라 기능 사용 -->
  <uses-permission android:name="android.permission.CAMERA" />
  <uses-feature android:name="android.hardware.camera" />

  <!-- 최소 SDK 버전을 21로 수정 -->
  <uses-sdk
      android:minSdkVersion="21"
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

(4-3) "app" 경로 아래 "BUILD" 파일에 카메라 권한 허용을 사용자에게 물어보기 위해 Mediapipe에서 제공하는 PemissionHelper의 Dependency를 추가합니다. 추가된 코드는 아래 10열에서 확인할 수 있습니다. 3열과 16열의 패키지명, 4열의 소스 경로, 15열의 이름, 19열의 앱 이름도 맞춰서 바꿔줍니다.

{% highlight shell linenos %}
android_library(
    name = "basic_lib",
    custom_package = "com.example.mediapipe.edgedetection",
    srcs = ["java/com/example/mediapipe/edgedetection/MainActivity.java"],
    manifest = "AndroidManifest.xml",
    resource_files = glob(["res/**"]),
    deps = [
        "//third_party:androidx_constraint_layout",
        "//third_party:androidx_appcompat",
        "//mediapipe/java/com/google/mediapipe/components:android_components",
    ],
)

android_binary(
    name = "edgedetection",
    custom_package = "com.example.mediapipe.edgedetection",
    manifest = "AndroidManifest.xml",
    manifest_values = {
        "appName": "MediaPipe Edge Detection",
        "mainActivity": ".MainActivity",
    },
    multidex = "native",
    deps = [
        ":basic_lib",
    ],
)
{% endhighlight %}

(4-4) "app/java/com/example/mediapipe/edgedetection" 경로 아래 "MainActivity.java" 파일에 사용자에게 카메라 권한 허용을 물어보는 코드를 추가합니다.

{% highlight java linenos %}
package com.example.mediapipe.edgedetection; // 패키지명 변경

import android.os.Bundle;
import androidx.appcompat.app.AppCompatActivity;
import com.google.mediapipe.components.PermissionHelper; // PermissionHelper 추가

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        PermissionHelper.checkAndRequestCameraPermissions(this); // 시작할 때 카메라 권한 허용을 확인하는 코드
    }

    // 사용자의 응답을 처리하는 코드
    @Override
    public void onRequestPermissionsResult(
        int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        PermissionHelper.onRequestPermissionsResult(requestCode, permissions, grantResults);
    }

    // 권한을 허용한 경우 startCamera 함수를 수행
    @Override
    protected void onResume() {
        super.onResume();
        if (PermissionHelper.cameraPermissionsGranted(this)) {
        startCamera();
        }
    }

    // 테스트를 위해 빈 startCamera 함수 생성
    public void startCamera() {}
}
{% endhighlight %}

(4-5) 카메라 권한을 문의하는 코드가 제대로 적용되었는지 테스트 빌드를 통해 확인합니다. 우선 CMD 창에서 아래와 같이 명령어를 입력하여 앱을 빌드합니다.

{% highlight shell linenos %}
bazel build -c opt --define MEDAPIPE_DISABLE_GPU=1 //app:edgedetection
{% endhighlight %}

<p><img src="/assets/images/22091803.png" /></p>
<p><img src="/assets/images/22091804.png" /></p>

> 만약 빌드 중 pip로 numpy가 설치되어있는데도 불구하고 Is numpy installed? 이슈가 발생한다면 아래 명령어로 numpy를 설치해주시기 바랍니다.

{% highlight shell linenos %}
sudo apt install python-numpy
{% endhighlight %}

<p><img src="/assets/images/22102806.png" /></p>



> 만약 빌드 중 file '@bazel_tools//tools/cpp:toolchain_utils.bzl' does not contain symbol 'use_cpp_toolchain' (did you mean 'find_cpp_toolchain'?) 이슈가 발생한다면 아래 WORKSPACE의 아래 부분을 다음과 같이 수정합니다. 해결 방법은 다음 링크를 참고했습니다. (<a href="https://github.com/google/mediapipe/issues/3457#issuecomment-1164620044">링크</a>)

<p><img src="/assets/images/22091802.png" /></p>

수정 전
{% highlight shell linenos %}
http_archive(
    name = "rules_cc",
    strip_prefix = "rules_cc-main",
    urls = ["https://github.com/bazelbuild/rules_cc/archive/main.zip"],
)
{% endhighlight %}

수정 후
{% highlight shell linenos %}
http_archive(
    name = "rules_cc",
    strip_prefix = "rules_cc-8bb0eb5c5ccd96b91753bb112096bb6993d16d13",
    urls = ["https://github.com/bazelbuild/rules_cc/archive/8bb0eb5.zip"],
)
{% endhighlight %}

> 만약 빌드 중 Error applying patch C:/users/{유저명}/androidstudioprojects/mediapipe-edge-detection/third_party/com_google_protobuf_fixes.diff: Incorrect Chunk: the chunk content doesn't match the target 이슈가 발생한다면 아래와 같이 WORKSPACE를 수정합니다.

<p><img src="/assets/images/22091805.png" /></p>

수정 전
{% highlight shell linenos %}
http_archive(
    name = "com_google_protobuf",
    sha256 = "87407cd28e7a9c95d9f61a098a53cf031109d451a7763e7dd1253abf8b4df422",
    strip_prefix = "protobuf-3.19.1",
    urls = ["https://github.com/protocolbuffers/protobuf/archive/v3.19.1.tar.gz"],
    patches = [
        "@//third_party:com_google_protobuf_fixes.diff"
    ],
    patch_args = [
        "-p1",
    ],
)
{% endhighlight %}

수정 후
{% highlight shell linenos %}
http_archive(
    name = "com_google_protobuf",
    sha256 = "87407cd28e7a9c95d9f61a098a53cf031109d451a7763e7dd1253abf8b4df422",
    strip_prefix = "protobuf-3.19.1",
    urls = ["https://github.com/protocolbuffers/protobuf/archive/v3.19.1.tar.gz"],
)
{% endhighlight %}

(4-6) 아래 명령어로 빌드한 앱을 모바일 디바이스에 설치한다.

{% highlight shell linenos %}
adb install .\bazel-bin\app\edgedetection.apk
{% endhighlight %}

<p><img src="/assets/images/22091806.png" /></p>

설치한 앱을 시작했을 때 아래 이미지처럼 권한을 확인하는 창이 나타나면 이번 단계는 성공적으로 완료된 것이다.

<p><img style="min-width: 250px;" width="30%" src="/assets/images/22091807.jpg" /></p>

## (5) 소스 코드 작성 - 카메라 연결

(5-1) 카메라 뷰를 보여주기 위해 SurfaceView를 사용합니다. "app/res/layout" 경로 아래 "activity_main.xml" 파일에서 기존에 있던 TextView를 삭제하고 FrameLayout을 추가합니다. 

{% highlight xml linenos %}
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

  <FrameLayout
    android:id="@+id/preview_display_layout"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:layout_weight="1">
    <TextView
        android:id="@+id/no_camera_access_view"
        android:layout_height="fill_parent"
        android:layout_width="fill_parent"
        android:gravity="center"
        android:text="@string/no_camera_access" />
  </FrameLayout>

</androidx.constraintlayout.widget.ConstraintLayout>
{% endhighlight %}

안에 있는 TextView는 사용자가 카메라 권한을 허용하지 않았을 때 나타납니다. 해당 TextView에 들어갈 문구를 "res/values" 아래 "strings.xml" 파일을 생성하여 지정해줍니다.

{% highlight xml linenos %}
<resources>
    <string name="no_camera_access" translatable="false">카메라 권한을 허용해주세요.</string>
</resources>
{% endhighlight %}

(5-2) "app/java/com/example/mediapipe/edgedetection" 경로 아래 "MainActivity.java" 파일에 SurfaceView를 사용하기 위한 코드를 추가합니다.

{% highlight java linenos %}
package com.example.mediapipe.edgedetection;

import android.os.Bundle;
import android.graphics.SurfaceTexture; // SurfaceTexture import
import android.view.SurfaceView; // SurfaceView import
import android.view.View; // View import
import android.view.ViewGroup; // ViewGroup import
import androidx.appcompat.app.AppCompatActivity;
import com.google.mediapipe.components.PermissionHelper;

public class MainActivity extends AppCompatActivity {

    private SurfaceTexture previewFrameTexture; // SurfaceTexture 오브젝트 추가
    private SurfaceView previewDisplayView; // SurfaceView 오브젝트 추가

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        previewDisplayView = new SurfaceView(this); // 카메라 허용 전에 카메라 Frame을 보여줄 SurfaceView 바인딩
        setupPreviewDisplayView();

        PermissionHelper.checkAndRequestCameraPermissions(this);
    }

    @Override
    public void onRequestPermissionsResult(
        int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        PermissionHelper.onRequestPermissionsResult(requestCode, permissions, grantResults);
    }

    @Override
    protected void onResume() {
        super.onResume();
        if (PermissionHelper.cameraPermissionsGranted(this)) {
        startCamera();
        }
    }

    // View를 추가하기 위한 setupPreviewDisplayView 함수 생성
    private void setupPreviewDisplayView() {
        previewDisplayView.setVisibility(View.GONE);
        ViewGroup viewGroup = findViewById(R.id.preview_display_layout);
        viewGroup.addView(previewDisplayView);
    }

    public void startCamera() {}
}
{% endhighlight %}

(5-3) CameraX를 이용하기 위해 "app" 경로 아래 "BUILD" 파일에 관련 Dependency를 추가합니다. 추가한 열은 11열에서 확인할 수 있습니다.

{% highlight shell linenos %}
android_library(
    name = "basic_lib",
    custom_package = "com.example.mediapipe.edgedetection",
    srcs = ["java/com/example/mediapipe/edgedetection/MainActivity.java"],
    manifest = "AndroidManifest.xml",
    resource_files = glob(["res/**"]),
    deps = [
        "//third_party:androidx_constraint_layout",
        "//third_party:androidx_appcompat",
        "//mediapipe/java/com/google/mediapipe/components:android_components",
        "//mediapipe/java/com/google/mediapipe/components:android_camerax_helper",
    ],
)

android_binary(
    name = "edgedetection",
    custom_package = "com.example.mediapipe.edgedetection",
    manifest = "AndroidManifest.xml",
    manifest_values = {
        "appName": "MediaPipe Edge Detection",
        "mainActivity": ".MainActivity",
    },
    multidex = "native",
    deps = [
        ":basic_lib",
    ],
)
{% endhighlight %}

(5-4) 이제 비워두었던 StartCamera 함수를 구현합니다.

{% highlight java linenos %}
package com.example.mediapipe.edgedetection;

import android.os.Bundle;
import android.graphics.SurfaceTexture;
import android.view.SurfaceView;
import android.view.View;
import android.view.ViewGroup;
import androidx.appcompat.app.AppCompatActivity;
import com.google.mediapipe.components.PermissionHelper;
import com.google.mediapipe.components.CameraXPreviewHelper; // CameraXPreviewHelper import

public class MainActivity extends AppCompatActivity {

    private SurfaceTexture previewFrameTexture;
    private SurfaceView previewDisplayView;
    private CameraXPreviewHelper cameraHelper; // CameraXPreviewHelper 오브젝트 추가

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        previewDisplayView = new SurfaceView(this);
        setupPreviewDisplayView();

        PermissionHelper.checkAndRequestCameraPermissions(this);
    }

    @Override
    public void onRequestPermissionsResult(
        int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        PermissionHelper.onRequestPermissionsResult(requestCode, permissions, grantResults);
    }

    @Override
    protected void onResume() {
        super.onResume();
        if (PermissionHelper.cameraPermissionsGranted(this)) {
        startCamera();
        }
    }

    private void setupPreviewDisplayView() {
        previewDisplayView.setVisibility(View.GONE);
        ViewGroup viewGroup = findViewById(R.id.preview_display_layout);
        viewGroup.addView(previewDisplayView);
    }

    // startCamera 함수 작성
    public void startCamera() {
        cameraHelper = new CameraXPreviewHelper();
        // 카메라 리스너 설정
        cameraHelper.setOnCameraStartedListener(
            surfaceTexture -> {
                previewFrameTexture = surfaceTexture;
                // 카메라 시작하면 SurfaceView 활성화
                previewDisplayView.setVisibility(View.VISIBLE);
        });
    }
}
{% endhighlight %}

(5-5) 카메라를 사용하기 위해서는 전면 카메라인지 후면카메라인지 선언해주어야 합니다. 이번 프로젝트에서는 후면 카메라를 사용합니다. 우선 "app" 경로 아래 "AndroidManifest.xml" 파일에 cameraFacingFront 메타데이터를 추가합니다.

{% highlight xml linenos %}
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.mediapipe.edgedetection">

  <uses-permission android:name="android.permission.CAMERA" />
  <uses-feature android:name="android.hardware.camera" />

  <uses-sdk
      android:minSdkVersion="21"
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
      <!-- 메타데이터 추가 -->
      <meta-data android:name="cameraFacingFront" android:value="${cameraFacingFront}"/>
  </application>

</manifest>
{% endhighlight %}

(5-6) 추가한 메타데이터를 이용해서 "app" 경로 아래 "BUILD" 파일에 후면 카메라 사용(전면 카메라 사용 안함)을 명시합니다. 추가한 코드는 22열에서 확인할 수 있습니다.

{% highlight shell linenos %}
android_library(
    name = "basic_lib",
    custom_package = "com.example.mediapipe.edgedetection",
    srcs = ["java/com/example/mediapipe/edgedetection/MainActivity.java"],
    manifest = "AndroidManifest.xml",
    resource_files = glob(["res/**"]),
    deps = [
        "//third_party:androidx_constraint_layout",
        "//third_party:androidx_appcompat",
        "//mediapipe/java/com/google/mediapipe/components:android_components",
        "//mediapipe/java/com/google/mediapipe/components:android_camerax_helper",
    ],
)

android_binary(
    name = "edgedetection",
    custom_package = "com.example.mediapipe.edgedetection",
    manifest = "AndroidManifest.xml",
    manifest_values = {
        "appName": "MediaPipe Edge Detection",
        "mainActivity": ".MainActivity",
        "cameraFacingFront": "False",
    },
    multidex = "native",
    deps = [
        ":basic_lib",
    ],
)
{% endhighlight %}

(5-7) 명시한 정보를 "app/java/com/example/mediapipe/edgedetection" 경로 아래 "MainActivity.java" 파일에 적용합니다.

{% highlight java linenos %}
package com.example.mediapipe.edgedetection;

import android.content.pm.ApplicationInfo; // ApplicationInfo import
import android.content.pm.PackageManager; // PackageManager import
import android.content.pm.PackageManager.NameNotFoundException; // NameNotFoundException import
import android.os.Bundle;
import android.graphics.SurfaceTexture;
import android.util.Log; // Log import
import android.view.SurfaceView;
import android.view.View;
import android.view.ViewGroup;
import androidx.appcompat.app.AppCompatActivity;
import com.google.mediapipe.components.CameraHelper; // CameraHelper import
import com.google.mediapipe.components.PermissionHelper;
import com.google.mediapipe.components.CameraXPreviewHelper;

public class MainActivity extends AppCompatActivity {
    private static final String TAG = "MainActivity"; // 로그용 태그 추가

    private SurfaceTexture previewFrameTexture;
    private SurfaceView previewDisplayView;
    private CameraXPreviewHelper cameraHelper;
    private ApplicationInfo applicationInfo; // ApplicationInfo 오브젝트 추가

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // 카메라 설정 읽어오기
        try {
            applicationInfo =
                getPackageManager().getApplicationInfo(getPackageName(), PackageManager.GET_META_DATA);
        } catch (NameNotFoundException e) {
            Log.e(TAG, "Cannot find application info: " + e);
        }

        previewDisplayView = new SurfaceView(this);
        setupPreviewDisplayView();

        PermissionHelper.checkAndRequestCameraPermissions(this);
    }

    @Override
    public void onRequestPermissionsResult(
        int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        PermissionHelper.onRequestPermissionsResult(requestCode, permissions, grantResults);
    }

    @Override
    protected void onResume() {
        super.onResume();
        if (PermissionHelper.cameraPermissionsGranted(this)) {
        startCamera();
        }
    }

    private void setupPreviewDisplayView() {
        previewDisplayView.setVisibility(View.GONE);
        ViewGroup viewGroup = findViewById(R.id.preview_display_layout);
        viewGroup.addView(previewDisplayView);
    }

    public void startCamera() {
        cameraHelper = new CameraXPreviewHelper();
        cameraHelper.setOnCameraStartedListener(
            surfaceTexture -> {
                previewFrameTexture = surfaceTexture;
                previewDisplayView.setVisibility(View.VISIBLE);
        });

        // 카메라 설정을 읽어온 뒤 카메라 켜기
        CameraHelper.CameraFacing cameraFacing =
            applicationInfo.metaData.getBoolean("cameraFacingFront", false)
                ? CameraHelper.CameraFacing.FRONT
                : CameraHelper.CameraFacing.BACK;
        cameraHelper.startCamera(this, cameraFacing, /*unusedSurfaceTexture=*/ null);
    }
}
{% endhighlight %}

## (6) 소스 코드 작성 - 화면 구현

(6-1) 카메라 화면을 실시간으로 보여주기위해 "app" 경로 아래 "BUILD" 파일에 두가지를 추가 해아합니다. 첫번째는 OpenGL ES texture입니다.  관련 Dependency를 추가합니다. 추가한 코드는 12열에서 확인할 수 있습니다.

{% highlight shell linenos %}
android_library(
    name = "basic_lib",
    custom_package = "com.example.mediapipe.edgedetection",
    srcs = ["java/com/example/mediapipe/edgedetection/MainActivity.java"],
    manifest = "AndroidManifest.xml",
    resource_files = glob(["res/**"]),
    deps = [
        "//third_party:androidx_constraint_layout",
        "//third_party:androidx_appcompat",
        "//mediapipe/java/com/google/mediapipe/components:android_components",
        "//mediapipe/java/com/google/mediapipe/components:android_camerax_helper",
        "//mediapipe/java/com/google/mediapipe/glutil",
    ],
)

android_binary(
    name = "edgedetection",
    custom_package = "com.example.mediapipe.edgedetection",
    manifest = "AndroidManifest.xml",
    manifest_values = {
        "appName": "MediaPipe Edge Detection",
        "mainActivity": ".MainActivity",
        "cameraFacingFront": "False",
    },
    multidex = "native",
    deps = [
        ":basic_lib",
    ],
)
{% endhighlight %}

(6-2) "app/java/com/example/mediapipe/edgedetection" 경로 아래 "MainActivity.java" 파일에 OpenGL ES texture를 사용하기 위한 ExternalTextureConverter와 EglManager 를 추가합니다.

{% highlight java linenos %}
package com.example.mediapipe.edgedetection;

import android.content.pm.ApplicationInfo;
import android.content.pm.PackageManager;
import android.content.pm.PackageManager.NameNotFoundException;
import android.os.Bundle;
import android.graphics.SurfaceTexture;
import android.util.Log;
import android.util.Size; // Size import
import android.view.SurfaceHolder; // SurfaceHolder import
import android.view.SurfaceView;
import android.view.View;
import android.view.ViewGroup;
import androidx.appcompat.app.AppCompatActivity;
import com.google.mediapipe.components.CameraHelper;
import com.google.mediapipe.components.PermissionHelper;
import com.google.mediapipe.components.CameraXPreviewHelper;
import com.google.mediapipe.components.ExternalTextureConverter; // ExternalTextureConverter import
import com.google.mediapipe.glutil.EglManager; // EglManager import

public class MainActivity extends AppCompatActivity {
    private static final String TAG = "MainActivity";

    private SurfaceTexture previewFrameTexture;
    private SurfaceView previewDisplayView;
    private CameraXPreviewHelper cameraHelper;
    private ApplicationInfo applicationInfo;
    private EglManager eglManager; // EglManager 오브젝트 추가
    private ExternalTextureConverter converter; // ExternalTextureConverter 오브젝트 추가

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        try {
            applicationInfo =
                getPackageManager().getApplicationInfo(getPackageName(), PackageManager.GET_META_DATA);
        } catch (NameNotFoundException e) {
            Log.e(TAG, "Cannot find application info: " + e);
        }

        previewDisplayView = new SurfaceView(this);
        setupPreviewDisplayView();

        eglManager = new EglManager(null); // eglManage 초기화

        PermissionHelper.checkAndRequestCameraPermissions(this);
    }

    @Override
    public void onRequestPermissionsResult(
        int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        PermissionHelper.onRequestPermissionsResult(requestCode, permissions, grantResults);
    }

    @Override
    protected void onResume() {
        super.onResume();
        converter = new ExternalTextureConverter(eglManager.getContext()); // ExternalTextureConverter 초기화
        if (PermissionHelper.cameraPermissionsGranted(this)) {
            startCamera();
        }
    }

    // 잠깐 멈췄을 때 실시간 재생도 멈추기 위해 onPause 함수 추가
    @Override
    protected void onPause() {
        super.onPause();
        converter.close();
    }

    private void setupPreviewDisplayView() {
        previewDisplayView.setVisibility(View.GONE);
        ViewGroup viewGroup = findViewById(R.id.preview_display_layout);
        viewGroup.addView(previewDisplayView);

        // 화면 사이즈를 계산해서 적용하기 위한 코드 추가
        previewDisplayView
        .getHolder()
        .addCallback(
            new SurfaceHolder.Callback() {
            @Override
            public void surfaceCreated(SurfaceHolder holder) {}

            @Override
            public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
                Size viewSize = new Size(width, height);
                Size displaySize = cameraHelper.computeDisplaySizeFromViewSize(viewSize);

                converter.setSurfaceTextureAndAttachToGLContext(
                    previewFrameTexture, displaySize.getWidth(), displaySize.getHeight());
            }

            @Override
            public void surfaceDestroyed(SurfaceHolder holder) {}
            });
    }

    public void startCamera() {
        cameraHelper = new CameraXPreviewHelper();
        cameraHelper.setOnCameraStartedListener(
            surfaceTexture -> {
                previewFrameTexture = surfaceTexture;
                previewDisplayView.setVisibility(View.VISIBLE);
        });

        CameraHelper.CameraFacing cameraFacing =
            applicationInfo.metaData.getBoolean("cameraFacingFront", false)
                ? CameraHelper.CameraFacing.FRONT
                : CameraHelper.CameraFacing.BACK;
        cameraHelper.startCamera(this, cameraFacing, /*unusedSurfaceTexture=*/ null);
    }
}
{% endhighlight %}

(6-3) "app" 경로 아래 "BUILD" 파일에 추가해야하는 두번째는 MediaPipe graph입니다. MediaPipe graph는 MediaPipe에서 제공하는 framework입니다. JNI 코드를 사용하기 위해 아래와 같이 코드를 추가합니다. 추가된 코드는 16~29열과 43열에서 확인할 수 있습니다.

{% highlight shell linenos %}
android_library(
    name = "basic_lib",
    custom_package = "com.example.mediapipe.edgedetection",
    srcs = ["java/com/example/mediapipe/edgedetection/MainActivity.java"],
    manifest = "AndroidManifest.xml",
    resource_files = glob(["res/**"]),
    deps = [
        "//third_party:androidx_constraint_layout",
        "//third_party:androidx_appcompat",
        "//mediapipe/java/com/google/mediapipe/components:android_components",
        "//mediapipe/java/com/google/mediapipe/components:android_camerax_helper",
        "//mediapipe/java/com/google/mediapipe/glutil",
    ],
)

cc_binary(
    name = "libmediapipe_jni.so",
    linkshared = 1,
    linkstatic = 1,
    deps = [
        "//mediapipe/java/com/google/mediapipe/framework/jni:mediapipe_framework_jni",
    ],
)

cc_library(
    name = "mediapipe_jni_lib",
    srcs = [":libmediapipe_jni.so"],
    alwayslink = 1,
)

android_binary(
    name = "edgedetection",
    custom_package = "com.example.mediapipe.edgedetection",
    manifest = "AndroidManifest.xml",
    manifest_values = {
        "appName": "MediaPipe Edge Detection",
        "mainActivity": ".MainActivity",
        "cameraFacingFront": "False",
    },
    multidex = "native",
    deps = [
        ":basic_lib",
        ":mediapipe_jni_lib",
    ],
)
{% endhighlight %}

(6-4) MediaPipe graph 중 어떤 graph를 사용할 지 "app" 경로 아래 "BUILD" 파일에 명시해줍니다. 추가된 코드는 21열과 34~37열, 44~46열에서 확인할 수 있습니다.

{% highlight shell linenos %}
android_library(
    name = "basic_lib",
    custom_package = "com.example.mediapipe.edgedetection",
    srcs = ["java/com/example/mediapipe/edgedetection/MainActivity.java"],
    manifest = "AndroidManifest.xml",
    resource_files = glob(["res/**"]),
    deps = [
        "//third_party:androidx_constraint_layout",
        "//third_party:androidx_appcompat",
        "//mediapipe/java/com/google/mediapipe/components:android_components",
        "//mediapipe/java/com/google/mediapipe/components:android_camerax_helper",
        "//mediapipe/java/com/google/mediapipe/glutil",
    ],
)

cc_binary(
    name = "libmediapipe_jni.so",
    linkshared = 1,
    linkstatic = 1,
    deps = [
        "//mediapipe/graphs/edge_detection:mobile_calculators",
        "//mediapipe/java/com/google/mediapipe/framework/jni:mediapipe_framework_jni",
    ],
)

cc_library(
    name = "mediapipe_jni_lib",
    srcs = [":libmediapipe_jni.so"],
    alwayslink = 1,
)

android_binary(
    name = "edgedetection",
    assets = [
        "//mediapipe/graphs/edge_detection:mobile_gpu.binarypb",
    ],
    assets_dir = "",
    custom_package = "com.example.mediapipe.edgedetection",
    manifest = "AndroidManifest.xml",
    manifest_values = {
        "appName": "MediaPipe Edge Detection",
        "mainActivity": ".MainActivity",
        "cameraFacingFront": "False",
        "binaryGraphName": "mobile_gpu.binarypb",
        "inputVideoStreamName": "input_video",
        "outputVideoStreamName": "output_video",
    },
    multidex = "native",
    deps = [
        ":basic_lib",
        ":mediapipe_jni_lib",
    ],
)
{% endhighlight %}

(6-5) 이제 MediaPipe graph를 "app/java/com/example/mediapipe/edgedetection" 경로 아래 "MainActivity.java" 파일에 적용합니다.

{% highlight java linenos %}
package com.example.mediapipe.edgedetection;

import android.content.pm.ApplicationInfo;
import android.content.pm.PackageManager;
import android.content.pm.PackageManager.NameNotFoundException;
import android.os.Bundle;
import android.graphics.SurfaceTexture;
import android.util.Log;
import android.util.Size;
import android.view.SurfaceHolder;
import android.view.SurfaceView;
import android.view.View;
import android.view.ViewGroup;
import androidx.appcompat.app.AppCompatActivity;
import com.google.mediapipe.components.CameraHelper;
import com.google.mediapipe.components.PermissionHelper;
import com.google.mediapipe.components.CameraXPreviewHelper;
import com.google.mediapipe.components.ExternalTextureConverter;
import com.google.mediapipe.glutil.EglManager;

public class MainActivity extends AppCompatActivity {
    private static final String TAG = "MainActivity";

    private SurfaceTexture previewFrameTexture;
    private SurfaceView previewDisplayView;
    private CameraXPreviewHelper cameraHelper;
    private ApplicationInfo applicationInfo;
    private EglManager eglManager;
    private ExternalTextureConverter converter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        try {
            applicationInfo =
                getPackageManager().getApplicationInfo(getPackageName(), PackageManager.GET_META_DATA);
        } catch (NameNotFoundException e) {
            Log.e(TAG, "Cannot find application info: " + e);
        }

        previewDisplayView = new SurfaceView(this);
        setupPreviewDisplayView();

        eglManager = new EglManager(null);

        PermissionHelper.checkAndRequestCameraPermissions(this);
    }

    @Override
    public void onRequestPermissionsResult(
        int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        PermissionHelper.onRequestPermissionsResult(requestCode, permissions, grantResults);
    }

    @Override
    protected void onResume() {
        super.onResume();
        converter = new ExternalTextureConverter(eglManager.getContext()); // ExternalTextureConverter 초기화
        if (PermissionHelper.cameraPermissionsGranted(this)) {
            startCamera();
        }
    }

    @Override
    protected void onPause() {
        super.onPause();
        converter.close();
    }

    private void setupPreviewDisplayView() {
        previewDisplayView.setVisibility(View.GONE);
        ViewGroup viewGroup = findViewById(R.id.preview_display_layout);
        viewGroup.addView(previewDisplayView);

        previewDisplayView
        .getHolder()
        .addCallback(
            new SurfaceHolder.Callback() {
            @Override
            public void surfaceCreated(SurfaceHolder holder) {}

            @Override
            public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
                Size viewSize = new Size(width, height);
                Size displaySize = cameraHelper.computeDisplaySizeFromViewSize(viewSize);

                converter.setSurfaceTextureAndAttachToGLContext(
                    previewFrameTexture, displaySize.getWidth(), displaySize.getHeight());
            }

            @Override
            public void surfaceDestroyed(SurfaceHolder holder) {}
            });
    }

    public void startCamera() {
        cameraHelper = new CameraXPreviewHelper();
        cameraHelper.setOnCameraStartedListener(
            surfaceTexture -> {
                previewFrameTexture = surfaceTexture;
                previewDisplayView.setVisibility(View.VISIBLE);
        });

        CameraHelper.CameraFacing cameraFacing =
            applicationInfo.metaData.getBoolean("cameraFacingFront", false)
                ? CameraHelper.CameraFacing.FRONT
                : CameraHelper.CameraFacing.BACK;
        cameraHelper.startCamera(this, cameraFacing, /*unusedSurfaceTexture=*/ null);
    }
}
{% endhighlight %}