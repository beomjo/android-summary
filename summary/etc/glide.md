# Glide

## Index

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

- [Glide](#glide)
  - [Index](#index)
  - [Glide란?](#glide란)
  - [Glide 특징](#glide-특징)
  - [설정](#설정)
  - [기본 사용법](#기본-사용법)
  - [많이쓰는 함수](#많이쓰는-함수)
    - [override()](#override)
    - [placeHolder()](#placeholder)
    - [error()](#error)
    - [asGif()](#asgif)
    - [thumbnail()](#thumbnail)
    - [Scaling - fitCenter(), centerCrop()](#scaling-fitcenter-centercrop)
      - [fitCenter()](#fitcenter)
      - [centerCrop()](#centercrop)
    - [Animation - crossFade(), dontAnimate()](#animation-crossfade-dontanimate)
  - [캐싱](#캐싱)
    - [기본 정책](#기본-정책)
    - [메모리 캐싱](#메모리-캐싱)
    - [디스크 캐싱](#디스크-캐싱)
  - [콜백](#콜백)

<!-- /code_chunk_output -->


## Glide란?
Bump! 앱을 만든 Bumptech가 구글에 인수되면서 Bump앱에서 사용하던 이미지 로딩 라이브러리를 공개하였다.


## Glide 특징
- 사용하기 편리한 API
- PlaceHolder, Animation, Transformation
- 다양한 데이터 모델 지원 (content://, file://, http://, android.resource://)
- gif 지원  
- Junk를 최소화 하기 위해 비트맵 객체 재활용

## 설정
gradle(project)
```
repositories {
  google()
  mavenCentral()
}
```

gradle(module)
```
apply plugin: 'kotlin-kapt'

dependencies {
  implementation 'com.github.bumptech.glide:glide:4.12.0'
  kapt 'com.github.bumptech.glide:compiler:4.12.0'
}
```

manifest
```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="your.package.name"

    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />  <!-- DCIM 또는 Pictures와 같은 로컬 폴더에서 이미지를로드 --> 
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />  <!-- SdCard에 글라이드의 캐시를 저장 -->


    <application>
      ...
    </application>
</manifest>
```

ProGuard
```
-keep public class * implements com.bumptech.glide.module.GlideModule
-keep class * extends com.bumptech.glide.module.AppGlideModule {
 <init>(...);
}
-keep public enum com.bumptech.glide.load.ImageHeaderParser$** {
  **[] $VALUES;
  public *;
}
-keep class com.bumptech.glide.load.data.ParcelFileDescriptorRewinder$InternalRewinder {
  *** rewind();
}

# for DexGuard only
-keepresourcexmlelements manifest/application/meta-data@value=GlideModule
```


## 기본 사용법
```kotlin
 Glide.with(this)
          .load(imageUrl)
          .placeholder(R.drawable.ic_launcher_background)
          .fitCenter()
          .into(imageView)
```


## 많이쓰는 함수

### override()
이미지 사이즈 조절
```kotlin
 Glide.with(this)
      .load(imageUrl)
      .override(100, 100)
      .into(imageView)
```

### placeHolder()
이미지를 로딩하는동안 보일 이미지 설정  
resourceId 또는 Drawable을 설정한다.  
```kotlin
 Glide.with(this)
      .load(imageUrl)
      .placeholder(R.drawable.ic_launcher_background)
      .into(imageView)
```

### error()
 이미지 로딩에 실패했을 경우 실패 이미지를 지정해줄 수 있다.
```kotlin
Glide.with(this) 
      .load(imageUrl) 
      .error(R.drawable.error) 
      .into(imageView)
```

### asGif()
gif 로딩
```kotlin
Glide.with(this) 
      .asGif()     
      .load(gifUrl) 
      .into(imageView)
```

### thumbnail()
 원본 이미지를 썸네일로 사용한다.     
 지정한 % 비율만큼 미리 이미지를 가져와서 보여준다.     
 0.1f로 지정했다면 실제 이미지 크기 중 10%만 먼저 가져와서 흐릿하게 보여준다.   
```kotlin
Glide.with(this)
      .load(imageUrl)
      .thumbnail(0.1f)
      .into(imageView)
```

### Scaling - fitCenter(), centerCrop()

#### fitCenter()
실제 이미지가 이미지뷰의 사이즈와 다를 때, 이미지와 이미지뷰의 중간을 맞춰서 이미지 크기를 스케일링한다.   
```kotlin
Glide.with(this)
     .load(imageUrl)
     .fitCenter()
     .into(target);
```

#### centerCrop()
실제 이미지가 이미지뷰의 사이즈보다 클 때, 이미지뷰의 크기에 맞춰 이미지 중간부분을 잘라서 스케일링한다.  
```kotlin
Glide.with(this)
     .load(imageUrl)
     .centerCrop()
     .into(target);
```

### Animation - crossFade(), dontAnimate()
원본 이미지가 다 로드되고 나면 PlaceHolder 이미지가 원본 이미지로 교체되는데, 이 때 애니메이션 처리를 할 수 있다.  
```kotlin 
// Animation On
Glide.with(this)
     .load(imageUrl)
     .placeholder(R.mipmap.ic_launcher)
     .erro(R.mipmap.ic_error)        
     .crossFade()
     .into(target);


// Animation Off
Glide.with(this)
     .load(imageUrl)
     .placeholder(R.mipmap.ic_launcher)
     .erro(R.mipmap.ic_error)       
     .dontAnimate()
     .into(target);
```


## 캐싱

### 기본 정책
Glide는 기본적으로 메모리 & 디스크에 이미지를 캐싱하여 불필요한 네트워크 연결을 줄인다.  

### 메모리 캐싱
- 기본적으로 메모리 캐싱을 하기때문에, 메모리 캐싱을 위해 추가적으로 할 일은 없다  
- 메모리 캐싱을 끄려면 skipMemoryCache(true)를 호출한다
- 처음 메모리 캐싱을 한 후에, skipMemoryCache(true)로 캐싱을 중지하더라도, 그 전에 저장된 캐시는 그대로 남아있다
```kotlin
Glide.with(this)
     .load(imageUrl)
     .skipMemoryCache(true)
     .into(target);
```

### 디스크 캐싱
- 기본적인 개념은 메모리 캐시와 같다. Glide는 기본적으로 디스크 캐싱을 수행한다
- 디스크 캐싱을 끄려면 diskCacheStrategy(DiskCacheStrategy.NONE) 메서드를 호출한다
- diskCacheStrategy 메서드는 DiskCacheStrategy enum을 인수로 받는다
    - DiskCacheStrategy.NONE : 디스크 캐싱을 하지 않는다.
    - DiskCacheStrategy.SOURCE : 원본 이미지만 캐싱
    - DiskCacheStrategy.RESULT : 변형된 이미지만 캐싱
    - DiskCacheStrategy.ALL : 모든 이미지를 캐싱(기본)
- 메모리 캐싱과는 별개이므로, 둘다 사용하지 않을 경우 다음과 같이 둘다 꺼주어야 한다
```kotlin
Glide.with(this)
    .load(imageUrl)
    .diskCacheStrategy(DiskCacheStrategy.ALL)
    .into(imageView);
```
```kotlin
Glide.with(this)
     .load(imageUrl)
     .skipMemoryCache(true)
     .diskCacheStrategy(DiskCacheStrategy.NONE)
     .into(target);
```


## 콜백
```kotlin
Glide.with(this)
    .load(imageUrl)
    .listener( object : RequestListener<Drawable>{
        override fun onLoadFailed(
            e: GlideException?,
            model: Any?,
            target: Target<Drawable>?,
            isFirstResource: Boolean
        ): Boolean {
            // do Something
            return false
        }

        override fun onResourceReady(
            resource: Drawable?,
            model: Any?,
            target: Target<Drawable>?,
            dataSource: DataSource?,
            isFirstResource: Boolean
        ): Boolean {
            // do Something
            return true
        }
    })
    .into(imageView)
```