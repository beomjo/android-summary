# Kotlin DSL + buildSrc 으로 의존성 관리

## Kotlin DSL이란?
DSL이란 <sup>Domain Specific Language</sup>의 약어로 특정 분야에 최적화된 프로그래밍 언어를 뜻한다.   
상용구 코드를 최소화 하기 위해 명령형 코드 대신 선언적 코드 형식을 따른다.  
Kotlin DSL은 코틀린의 언어적인 특징으로 가독성이 좋고 간략한 코드를 사용하여 Gradle 스크립팅을 하는 것을 목적으로 하는 DSL이다.  

## Kotlin DSL 장, 단점
### 장점
- IDE 지원 향상된 편집환경
    - Code highlighting
    - 자동완성 지원
    - 코드 탐색
    - 오류 코드 강조
    - 변수 리펙토링 가능
- 익숙한 Kotlin언어 사용
    - 러닝 커브 낮음
- 멀티 모듈사용시 중복 의존성 선언 필요없어짐

### 단점 
- 빌드 캐시가 Invalidation 되거나 클린 빌드시에 Groovy DSL보다 느림
- 새로운 라이브러리 버전 Inspection 기능 미지원
- Java8이상에서 동작


## Kotlin DSL 사용
기존 `*.gradle` 파일을 `*.gradle.kts` 로 변경한다.
```
ex) build.gradle -> build.gradle.kts
```  

아래 링크를 참고하여 kts파일로 변환하자    
- [KTS로 이전하기](https://developer.android.com/studio/build/migrate-to-kts)  
- [Google 도움말 레시피](https://developer.android.com/studio/build/gradle-tips)
- [Gradle Kotlin Dsl 입문서](https://docs.gradle.org/current/userguide/kotlin_dsl.html)

## buildSrc
의존성 관리와 IDE 자동완성 지원을 위해 kotlin 코드를 가지는 buildSrc 모듈을 만들 수 있다. 

Gradle 문서를 보면  
> Gradle이 수행되면 buildSrc 디렉토리가 존재하는지 체크한다.   
> 이 경우 Gradle은 자동적으로 코드를 컴파일하고 테스트한 뒤 당신의 빌드 스크립트의 classpath에 넣는다.
> 이 방법은 유지 보수, 리팩토링 및 코드 테스트가 더 쉽다.

즉, buildSrc는 빌드 로직을 포함할 수 있는 Gradle 프로젝트 루트 디렉토리로   
buildSrc과 Kotlin DSL을 사용해서 매우 적은 구성으로 커스텀 빌드 코드를 작성하고 전체 프로젝트에서 이 로직을 공유 할 수 있다.   
buildSrc를 변경하면 전체 프로젝트의 빌드캐시가 무효화되기 때문에 잦은 수정을 피해야 한다.  


## 설정하기
- rootDir에 buildSrc Directory 생성
- buildSrc/build.gradle.kts 생성
- buildSrc/src/main/kotlin/Dependency.kt 생성
- build.gradle(:app) -> build.gradle.kts 변경
- build.gradle(:project) -> build.gradle.kts 변경
- setting.gradle -> setting.gradle.kts 변경

### rootDir에 buildSrc Directory 생성
![image](https://user-images.githubusercontent.com/39984656/127188737-28565465-dcd8-425a-ba8e-70832158c095.png)

### buildSrc/build.gradle.kts 생성
Gradle을 실행하면 buildSrc라는 디렉토리가 존재하는지 검사한다.  
존재한다면 자동으로 `buildSrc/build.gradle.kts` 코드를 컴파일하고 테스트한 뒤에 빌드 스크립트의 클래스패스에 집어 넣는다.  

**buildSrc/build.gradle.kts**
```kotlin
plugins {
    `kotlin-dsl` // kotlin dsl 설정
}

repositories {
    google()
    mavenCentral()
}

object PluginVersion {
    const val GRADLE = "4.2.2"
    const val KOTLIN = "1.5.21"

}

dependencies {
    implementation("com.android.tools.build:gradle:${PluginVersion.GRADLE}")
    implementation("org.jetbrains.kotlin:kotlin-gradle-plugin:${PluginVersion.KOTLIN}")
}

```

### buildSrc/src/main/kotlin/Dependency.kt 생성
`buildSrc/src/main/kotlin` 경로에 `Dependency.kt` 파일을 생성한다.  

```kotlin
object Dependency {

    object Kotlin {
        const val SDK = "org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.5.21"
    }

    object AndroidX {
        const val MATERIAL = "androidx.compose.material:material:1.0.0-rc02"
        const val CONSTRAINT_LAYOUT = "androidx.constraintlayout:constraintlayout:2.1.0-rc01"
        const val APP_COMPAT = "androidx.appcompat:appcompat:1.3.1"
    }

    object KTX {
        const val CORE = "androidx.core:core-ktx:1.7.0-alpha01"
    }

    object Test {
        const val JUNIT = "junit:junit:4.13.2"
        const val ANDROID_JUNIT_RUNNER = "AndroidJUnitRunner"
    }

    object AndroidTest {
        const val TEST_RUNNER = "androidx.test:runner:1.4.0"
        const val ESPRESSO_CORE = "androidx.test.espresso:espresso-core:3.4.0"
    }
}
```  

### build.gradle(:app) -> build.gradle.kts 변경

**기존  build.gradle(:app)**  
```gradle
plugins {
    id 'com.android.application'
    id 'kotlin-android'
}

android {
    compileSdkVersion 30
    buildToolsVersion "30.0.3"

    defaultConfig {
        applicationId "io.beomjo.kakao.search"
        minSdkVersion 16
        targetSdkVersion 30
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = '1.8'
    }
}

dependencies {

    implementation `org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version`
    implementation 'androidx.core:core-ktx:1.6.0'
    implementation 'androidx.appcompat:appcompat:1.3.0'
    implementation 'com.google.android.material:material:1.4.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.0.4'
    testImplementation 'junit:junit:4.+'
    androidTestImplementation 'androidx.test.ext:junit:1.1.3'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.4.0'
}
```

**변경 build.gradle.kts(:app)**  
```kotlin
plugins {
    android
    `kotlin-android`
}

android {
    compileSdkVersion(30)
    buildToolsVersion = "30.0.3"

    defaultConfig {
        applicationId = "io.beomjo.sample"
        minSdkVersion(21)
        targetSdkVersion(30)
        vectorDrawables.useSupportLibrary = true
        versionCode = 1
        versionName = "1"

        testInstrumentationRunner = Dependency.Test.ANDROID_JUNIT_RUNNER
    }

    buildTypes {
        getByName("release") {
            minifyEnabled(false)
            proguardFiles(
                getDefaultProguardFile(
                    "proguard-android-optimize.txt"
                ),
                "proguard-rules.pro"
            )
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
    }

    kotlinOptions {
        jvmTarget = JavaVersion.VERSION_1_8.toString()
    }
}

dependencies {
    implementation(Dependency.Kotlin.SDK)
    implementation(Dependency.KTX.CORE)
    implementation(Dependency.AndroidX.APP_COMPAT)
    implementation(Dependency.AndroidX.MATERIAL)
    implementation(Dependency.AndroidX.CONSTRAINT_LAYOUT)

    testImplementation(Dependency.Test.JUNIT)

    androidTestImplementation(Dependency.AndroidTest.TEST_RUNNER)
    androidTestImplementation(Dependency.AndroidTest.ESPRESSO_CORE)
}
```  

### build.gradle(:project) -> build.gradle.kts 변경

**기존 build.gradle(:project)**
```
buildscript {
    ext.kotlin_version = "1.5.21"
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath "com.android.tools.build:gradle:4.2.2"
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

allprojects {
    repositories {
        google()
        mavenCentral()
        jcenter() 
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```
 
**변경 build.gradle.kts(:project)**
```kotlin
subprojects {
    repositories {
        google()
        mavenCentral()
    }
}

tasks.register("clean", Delete::class) {
    delete(rootProject.buildDir)
}

```

### setting.gradle -> setting.gradle.kts 변경
**기존 setting.gradle**
```
rootProject.name = "My Project"
include ':app'
```

**변경 setting.gradle.kts**
```kotlin
rootProject.name = "My Project"
include(":app")

```
