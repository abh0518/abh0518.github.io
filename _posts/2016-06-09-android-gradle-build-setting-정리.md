---
title: "Android Gradle Build Setting 정리"
date: 2016-06-09 11:27:59 +0900
categories: [프로그래밍]
---

안드로이드 프로젝트 진행하며 사용한 그래들 빌드 세팅 정리

# **Application Bulid 설정**

1. 프로젝트의 gradle 버전 업데이트
   1. 프로젝트 root디렉토리의 build.gradle 파일 수정하여 gradle 버전을 1.5 이상으로 올려준다. 프로젝트를 module 단위로 관리할것 아니면 상관 없지만 그냥 기분으로라도 올려준다.

      ```
      buildscript {
          repositories {
              jcenter()
          }
          dependencies {
              //그래들 버전이 1.5 이상이어야 module의 aar 파일명 변경(baseName) 옵션이 정상 동작 한다
              classpath 'com.android.tools.build:gradle:1.5.0'
          }
      }
      ```
2. app/build.gradle파일 수정 및 Flavor 리소스 분리
   1. defaultConfig에 setProperty로 ouput 파일의 이름을 설정해준다. 파일 이름이 versionName과 versionCode가 추가되도록 하면 빌드 후 apk별 버전 관리가 쉬워진다.
   2. Flavor를 추가한다. dev, staging, product 세개의 Flavor로 나누가 각각 개발용, QA용, 상용 배포용 으로 나누어 빌드 할 수 있도록 한다. 대충 아래같은 느낌적 느낌.

      ```
      apply plugin: 'com.android.application'
      
      android {
          compileSdkVersion 23
          buildToolsVersion "23.0.2"
      
          defaultConfig {
              applicationId "net.abh0518.androidbuildsetting"
              minSdkVersion 16
              targetSdkVersion 16
              versionCode 1
              versionName "1.0"
              //apk파일 이름에 version 정보가 들어가도록 내용 추가
              setProperty("archivesBaseName", "app-v${versionName}c${versionCode}")
          }
          buildTypes {
              release {
                  minifyEnabled false
                  proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
              }
          }
      
          //빌드별 Flavor 설정을 해준다.
          productFlavors{
              dev{
                  applicationId "net.abh0518.androidbuildsetting.dev"
              }
              staging{
                  applicationId "net.abh0518.androidbuildsetting.staging"
              }
              product{
                  applicationId "net.abh0518.androidbuildsetting.product"
              }
          }
      }
      ```
   3. Flavor에 맞게 리소스 파일 분리
      app/src 디렉토리 아래에 flavor과 동일한 이름의 디렉토리들을 생성하고 실행 환경에 따른 리소스 파일을 분리한다. main이 기본이고 동일한 구성으로 변경 사항들만 dev, staging, product쪽에 추가해준다. 리소스들은 빌드시 선택된 Flavor값들이 main리소스에 override된다. 대충 아래와 같은 느낌. Flavor 관련 자세한 설명은 구글 문서의 Flavor 항목을 참고하자. (<https://developer.android.com/studio/build/build-variants.html?hl=ko>)

      ![app - AndroidBuildSetting - [~:AndroidStudioProjects:AndroidBuildSetting] 2016-06-09 10-35-32](/assets/images/2016/06/app-AndroidBuildSetting-AndroidStudioProjectsAndroidBuildSetting-2016-06-09-10-35-32.png)
3. 빌드
   1. Android Studio를 사용한다면 Build Variant 메뉴를 사용해서 빌드될 Flavor를 선택할 수 있다.
   2. Termianal사용시는 프로젝트 root에서 gradlew assemble${Flaver name}${BuildType} 명령으로 빌드할 수 있다.

      ```
      ./gradlew assembledevDebug //dev를 Debug로 빌드
      ./gradlew assemblestagingDebug //staging을 Debug로 빌드 
      ./gradlew assembleproductDebug //product를 Debug로 빌드
      ./gradlew assembleproductRelease // pruduct를 Release로 빌드, 릴리즈의 경우 gradle에 signing 설정이 되어있지 않아면 unsigned apk가 나온다.
      ```

# **Library Module Bulid 설정**

다른 파트들과 협업을 하면서 우리의 코드를 안드로이드 라이브러리(aar)파일로 만들어 배포하기 위한 빌드 설정이다.

1. 프로젝트에 Android Library 모듈을 추가한다.
2. 버전 관리를 위해 library module의 java source에 Version.java 를 추가해 준다. 이유는 Android Module을 빌드할때는 Application빌드와 달리 Gradle의 versionCode와 versionName 항목들이 무시되기 때문이다. 따라서 별도로 Version정보 클래스를 제공하지 않으면 Library를 사용하는 사람 입장에서 버전을 확일할 방법이 없어진다. 적당한 위치에 아래처럼 만들어주면 된다.
   1. 대충 이런 느낌

      ```
      package net.abh0518.applibrary;
      
      public class Version {
          public final static int versionCode = 1;
          public final static String versionName = "1.0.0";
      }
      ```
3. Library 모듈의 build.gradle 파일을 수정 (여기서는 app\_library/build.gradle 파일)
   1. Version.java코드에서 버전정보를 읽어들여 output파일 명에 버전 정보를 추가한다.
   2. Application 의 설정과 동일하게 Flavor를 나누어서 관리하면 좋다. 사용자에게 개발버전, QA버전, 상용버전을 나누어서 제공할 수 있으면 좋지 않을까? 단 Flavor를 추가한 이후 defaultPublishConfig를 반드시 설정해야하는데 이것은 같은 프로젝트 내에서 모듈을 include할때 선택할 Flavor가 된다. Android Studio버그인지 빌드 시스템의 문제인지는 모르겠는데 Studio의 Build Vriant에서 선택해 놓은 Module의 Flavor가 참조하는 쪽의 Flaovr와 연결되진 않고 그냥defaultPublishConfig 설정에 맞춰 빌드되어 참조된다. 근데 뭐 난 Library모듈을 프로젝트 내부에서 쓰진 않는 상황이라 무시하고 그냥 dev로 잡아버렸다.
   3. Flavor를 설정했다면 module의 resource 파일 설정도 Application때와 동일하게 설정해 준다.
   4. Library Module의 build.gradle은 대충 이런 느낌....

      ```
      apply plugin: 'com.android.library'
      
      // Android Library 프로젝트는 그래들에서 제공하는 버전 코드와 버전 네임 항목을 무시하므로 버전 관리를 위해서 별도 파일 처리를 해준다.
      def moduleVersionCode  = 0
      def moduleVersionName = "0.9.0"
      
      // Java 소스 파일에서 버전 정보를 읽어온다.
      File versionFile = file('src/main/java/net/abh0518/applibrary/Version.java')
      versionFile.eachLine { line ->
          //버전 코드 가져오기
          def group = (line =~ /versionCode( )*=( )*+/)
          if(group.hasGroup() && group.size() > 0){
              moduleVersionCode = group.toString().replaceAll("versionCode( )*=( )*", "")
          }
      
          //버전 네임 가져오기
          group = (line =~ /versionName( )*=( )*\".*\"/)
          if(group.hasGroup() && group.size() > 0){
              moduleVersionName = group.toString().replaceAll("(versionName( )*=( )*)|\"", "")
          }
      }
      
      android {
          compileSdkVersion 23
          buildToolsVersion "23.0.2"
      
          defaultConfig {
              minSdkVersion 16
              targetSdkVersion 16
              //output 파일 이름 설정
              setProperty("archivesBaseName", "library-v${moduleVersionName}c${moduleVersionCode}")
          }
      
          buildTypes {
              release {
      //            minifyEnabled false
      //            debuggable false
      //            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
              }
              debug{
      //            minifyEnabled false
      //            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
                  debuggable true
              }
          }
          productFlavors{
              dev{
              }
              staging{
              }
              product{
              }
          }
          //라이브러리 모듈에 Flavor를 추가했을 경우 아래 옵션으로 기본 Flavor를 설정해 주어야 Android Studio에서 Run이 정상 동작을 한다.
          defaultPublishConfig "devDebug"
      }
      
      dependencies {
          compile fileTree(dir: 'libs', include: ['*.jar'])
      }
      ```
4. 빌드해 봅시다.
   1. Studio에서 별도로 aar로 빌드하는 방법은 찾지 못했다. terminal에서 application을 빌드하면 aar은 무더기로 함께 빌드 되어 있었다. module만 따로 빌드하는 방법은 찾지 못했다. 빌드된 aar파일은  "projectRoot/$moduleDir/build/outputs/aar" 에 있다.
   2. Terminal에서 aar만 따로 빌드 할수 있긴 하다. "./gradlew :${moduleName}:aR" 명령으로 가능하다. aar파일 위치는 A와 동일.

      ```
      ./gradleW :app_library:aR
      ```
5. 배포된 aar 사용하기, 사용하는 application의 gradle 설정을 아래처럼 하면 된다고 한다. (귀찮아서 테스트 안해봄)

   ```
   dependencies {
       compile 'package.name.of.your.aar:myaar@aar'
   }
   
   repositories{
       flatDir{
           dirs 'libs'
       }
   }
   ```

# **귀찮으니까 그냥 한번에 빌드**

1. 고객쪽에 Application dev, staging, product판과 Library Module dev, staging, product 판을 자주 전달해야하는데 매번 flavor 별로 따로 빌드하는게 귀찮다. project rootdptj shell로 그냥 한번에 끝내자. 대충 아래 느낌으로....

   ```
   #!/bin/bash
   
   //ouput 파일을 모아놓을 장소
   package_app_dir="release_app_package"
   package_module_dir="release_module_package"
   
   # Application의 build.gradle 파일에서 버전 정보를 가져온다.
   appVersionCode=$(echo "$fullName"  | sed -n '/versionCode /p' ./app/build.gradle)
   appVersionName=$(echo "$fullName"  | sed -n '/versionName /p' ./app/build.gradle)
   [[ $appVersionCode =~ + ]]
   appVersionCode=${BASH_REMATCH}
   [[ $appVersionName =~ +.+ ]]
   appVersionName=${BASH_REMATCH}
   
   # 패키지 디렉토리 및 그래들 빌드 환경을 초기화
   rm -rf $package_app_dir
   mkdir $package_app_dir
   rm -rf $package_module_dir
   mkdir $package_module_dir
   ./gradlew clean
   
   # 스테이징 디버그, 상용 디버그, 배포용 빌드
   ./gradlew assembledevDebug
   find ./app -name '*-debug.apk' -exec mv {} ./$package_app_dir/ \;
   
   ./gradlew assemblestagingDebug
   find ./app -name '*-debug.apk' -exec mv {} ./$package_app_dir/ \;
   
   ./gradlew assembleproductDebug
   find ./app -name '*-debug.apk' -exec mv {} ./$package_app_dir/ \;
   
   ./gradlew assembleproductRelease
   find ./app -name '*-unsigned.apk' -exec mv {} ./$package_app_dir/ \;
   
   ./gradleW :app_library:aR
   find ./app_library -name '*.aar' -exec mv {} ./$package_module_dir/ \;
   ```

GitHub Link : <https://github.com/abh0518/android_build_setting>
