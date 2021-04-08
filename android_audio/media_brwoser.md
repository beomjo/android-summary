# MediaBrowser with Audio


## Index
- [MediaBrowser with Audio](#mediabrowser-with-audio)
  - [Overview](#overview)
  - [MediaBrowserClient](#mediabrowserclient)
    - [MediaBrowserService와의 연결 Connection Callback 설정하기](#mediabrowserservice와의-연결-connection-callback-설정하기)
    - [MediaController Callback 생성 및 등록](#mediacontroller-callback-생성-및-등록)
    - [MediaBrowserService에 연결하기](#mediabrowserservice에-연결하기)
  - [MediaBrowserService](#mediabrowserservice)
    - [MediaSession 초기화](#mediasession-초기화)
      - [MediaSession 생성 및 초기화, 및 토큰 설정](#mediasession-생성-및-초기화-및-토큰-설정)
      - [MediaSession Callback 설정](#mediasession-callback-설정)
      - [MediaBrowserServiceCompat Code](#mediabrowserservicecompat-code)
    - [MediaBrowser Client와 연결 관리](#mediabrowser-client와-연결-관리)
      - [onGetRoot()로 Client와 연결 제어](#ongetroot로-client와-연결-제어)
      - [onLoadChildren()으로 Client에 콘텐츠 전달](#onloadchildren으로-client에-콘텐츠-전달)
    - [포그라운드 서비스에서 MediaStyle 알림 사용](#포그라운드-서비스에서-mediastyle-알림-사용)

## Overview
오디오 앱을 만들 때 가장 기본적인 아키텍처는 클라이언트/서버 이다.
플레이어와 미디어세션은 `MediaBrowserService`내에서 구현되며 
UI와 미디어 컨트롤러는 `MediaBrowser`와 함께 Activity 등에서 실행된다.

![image](https://user-images.githubusercontent.com/39984656/114038174-9a019280-98bc-11eb-88b1-620aa3c538bb.png)


## MediaBrowserClient
`MediaBrowser` 를 사용하여 `MediaBrowserService` 와 연결하는 역할을 한다.

### MediaBrowserService와의 연결 Connection Callback 설정하기
클라이언트(`MediaBrowser`)를 생성한 후 ConnectionCallback의 인스턴스를 만들어 등록한다.
`MediaBrowserCompat.ConnectionCallback`
```kotlin
 private val connectionCallbacks = object : MediaBrowserCompat.ConnectionCallback() {
        override fun onConnected() {
            val mediaController = MediaControllerCompat(context,mediaBrowser.sessionToken)
        }

        override fun onConnectionSuspended() {
        }

        override fun onConnectionFailed() {
        }
}
```

### MediaController Callback 생성 및 등록
`MediaBrowserService`에서 다시 전달받을 `MediaSession`의 상태를 표시해야할때 
아래 콜백을 통해서 `MediaBrowserService`에 있는 `MediaSession`의 상태를 얻을 수 있다.
```kotlin
private var controllerCallback = object : MediaControllerCompat.Callback() {

    override fun onMetadataChanged(metadata: MediaMetadataCompat?) {}

    override fun onPlaybackStateChanged(state: PlaybackStateCompat?) {}
}
```

컨트롤러에 콜백 등록하기
```kotlin
lateinit var mediaController : MediaControllerCompat? =null

mediaController.registerCallback(controllerCallback)
```

### MediaBrowserService에 연결하기
Activity등이 생성되었을 때 MediaBrowserService에 연결해야한다.
ConnectionCallback과 함께 생성한다.

```kotlin
    class MediaPlayerActivity : AppCompatActivity() {

        private lateinit var mediaBrowser: MediaBrowserCompat

        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            mediaBrowser = MediaBrowserCompat(
                    this,
                    ComponentName(this, MediaPlaybackService::class.java),
                    connectionCallbacks,
                    null // optional Bundle
            )
        }

        public override fun onStart() {
            super.onStart()
            mediaBrowser.connect()
        }

        public override fun onResume() {
            super.onResume()
            volumeControlStream = AudioManager.STREAM_MUSIC
        }

        public override fun onStop() {
            super.onStop()
            MediaControllerCompat.getMediaController(this)?.unregisterCallback(controllerCallback)
            mediaBrowser.disconnect()
        }
    }
```


## MediaBrowserService 
`MediaBrowserService` 는 Player를 관리하고, 미디어(영상, 음악)을 준비하며, 사용하는 영역이다.
미디어를 제어할 수 있는 알림(Notification) 또한 Service에서 생성하고 관리한다.

### MediaSession 초기화
`MediaBrowserService` 의 `onCreate()` 수명주기 콜백 메서드에서 초기화를위해 아래와 같은 단계를 실행해야한다.
- MediaSession 생성 및 초기화
- MediaSession Token 설정
- MediaSession Callback 설정

#### MediaSession 생성 및 초기화, 및 토큰 설정
```kotlin
 override fun onCreate() {
        super.onCreate()
        mediaSession = MediaSessionCompat(this, baseContext.getString(R.string.app_name)).apply {
            setCallback(callback)
            setSessionToken(sessionToken)
            isActive = true
        }
    }
```

#### MediaSession Callback 설정
```kotlin
 private val callback = object : MediaSessionCompat.Callback() {
        override fun onPrepareFromUri(uri: Uri?, extras: Bundle?) { ... }

        override fun onPlay() { ... }

        override fun onPause() { ... }

        override fun onStop() { ... }
    }
```
callback은 MediaSession에 등록한다.
이 등록된 callback은 MediaBrowser의 Controller에서 제어하였을때, 선언한 callback 메서드들이 실행된다.
[사용할 수 있는 CallBack Method](https://developer.android.com/reference/kotlin/android/support/v4/media/session/MediaSessionCompat.Callback)

#### MediaBrowserServiceCompat Code
```kotlin
class MediaPlayService : MediaBrowserServiceCompat() : {

    private var mediaSession: MediaSessionCompat? = null

    private val callback = object : MediaSessionCompat.Callback() {
        override fun onPrepareFromUri(uri: Uri?, extras: Bundle?) { ... }

        override fun onPlay() { ... }

        override fun onPause() { ... }

        override fun onStop() { ... }
    }

    override fun onCreate() {
        super.onCreate()
        mediaSession = MediaSessionCompat(this, baseContext.getString(R.string.app_name)).apply {
            setCallback(callback)
            setSessionToken(sessionToken)
            isActive = true
        }
    }

}
```

### MediaBrowser Client와 연결 관리
`MediaBrowserService`에는 클라이언트(`MediaBrowser`)와 연결을 처리하는 두 가지 메서드가 있다.
- `onGetRoot()` 
- `onLoadChildren()`

#### onGetRoot()로 Client와 연결 제어
`onGetRoot()` 메서드는 `BrowserRoot` 반환한다.
메서드가 null을 반환하면 클라이언트와 연결이 거부된다.

클라이언트(`MediaBrowser`)가 `MediaBrowserService`에 연결하도록 허용하려면 
반드시 `BrowserRoot`를 반환해야 한다.

```kotlin
override fun onGetRoot(
        clientPackageName: String,
        clientUid: Int,
        rootHints: Bundle?
    ): BrowserRoot? {
        return if (TextUtils.equals(clientPackageName, packageName)) {
            BrowserRoot(MY_MEDIA_ROOT_ID, null)
        } else null
    }
```

#### onLoadChildren()으로 Client에 콘텐츠 전달
클라이언트(`MediaBrowser`)는 연결 이후 `MediaBrowserCompat.subscribe()`를 호출하여 `BrowserRoot`를 순회할 수 있다.
`subscribe()`메소드는 `onLoadChildren()` 콜백을 서비스로 전송하고
그러면 `MediaBrowser.MediaItem` 객체 목록이 반환된다.

```kotlin
  override fun onLoadChildren(
            parentMediaId: String,
            result: MediaBrowserServiceCompat.Result<List<MediaBrowserCompat.MediaItem>>
    ) {
        //  Browsing not allowed
        if (MY_EMPTY_MEDIA_ROOT_ID == parentMediaId) {
            result.sendResult(null)
            return
        }

        val mediaItems = emptyList<MediaBrowserCompat.MediaItem>()

        if (MY_MEDIA_ROOT_ID == parentMediaId) {
            mediaItems.add(...)
        } else {
           
        }
        result.sendResult(mediaItems)
    }
```

### 포그라운드 서비스에서 MediaStyle 알림 사용
[MediaStyle 알림 만들기](https://github.com/beomjo/android-study/blob/main/notification/expandable/overview.md#media-controls-notification%EB%AF%B8%EB%94%94%EC%96%B4-%EC%BB%A8%ED%8A%B8%EB%A1%A4-%EC%95%8C%EB%A6%BC)
```kotlin
val notification = createNotification()
 startForeground(NOTIFICATION_ID, notification)
```