# Glide

## Glide란?

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