# Expandable Notification(확장형 알림)

## Large image notification(큰 이미지 추가 알림)
<img src="https://user-images.githubusercontent.com/39984656/113369370-2b907200-939c-11eb-96bf-42c8b4175127.png" width="700" height="700">


## Large block of text notification (큰 텍스트 블록 알림)
<img src="https://user-images.githubusercontent.com/39984656/113369422-482caa00-939c-11eb-99b2-bb0b6d0e3aa7.png" width="700" height="700">


## Inbox-style notification(받은편지함 스타일 알림)
<img src="https://user-images.githubusercontent.com/39984656/113369697-e15bc080-939c-11eb-97c6-135f64b01a5f.png" width="700" height="500">


## Conversation in a notification(대화 표시 알림)
<img src="https://user-images.githubusercontent.com/39984656/113369781-18ca6d00-939d-11eb-923c-9ac18fcb3e93.png" width="700" height="500">


## Media controls notification(미디어 컨트롤 알림)
<img src="https://user-images.githubusercontent.com/39984656/113369820-38619580-939d-11eb-8f69-e0b7eab1557e.png" width="700" height="500">

```kotlin
    private fun createNotification(
        context: Context,
        track: Track,
    ): Notification {
        val pendingIntent = PendingIntent.getActivity(
            context,
            0,
            Intent(context, PlayerActivity::class.java).apply {
                putExtra(PlayerActivity.KEY_PLAYER_TRACK, track)
                putExtra(PlayerActivity.KEY_BOTTOM_PLAYER_CLICK, true)
            },
            PendingIntent.FLAG_UPDATE_CURRENT
        )

        val pausePendingInt =
            PendingIntent.getService(
                context, 0,
                Intent(context, PlayerService::class.java)
                0,
            )

        val stopPendingIntent =
            PendingIntent.getService(
                context, 0,
                Intent(context, PlayerService::class.java).apply {
                    this.action = PlayerAction.STOP.value
                },
                0,
            )
        val stopAction = NotificationCompat.Action(
            R.drawable.ic_close_white,
            "",
            stopPendingIntent,
        )

        val appName = context.getString(R.string.app_name)
        val mediaSession = MediaSessionCompat(context, appName).apply {
            setMetadata(
                MediaMetadataCompat.Builder()
                    .putString(MediaMetadata.METADATA_KEY_TITLE, track.title)
                    .putString(MediaMetadata.METADATA_KEY_ARTIST, track.desc)
                    .build()
            )
        }

        return NotificationCompat.Builder(context, CHANNEL_ID)
            .setSmallIcon(R.drawable.ic_launcher_background)
            .setVisibility(NotificationCompat.VISIBILITY_PUBLIC)
            .addAction(NotificationCompat.Action(
                R.drawable.ic_pause_white,
                "",
                pausePendingInt,
            ))
            .addAction(stopAction)
            .setStyle(
                androidx.media.app.NotificationCompat.MediaStyle()
                    .setShowActionsInCompactView(0, 1)
                    .setMediaSession(mediaSession.sessionToken)
            )
            .setOngoing(true)
            .setContentTitle(context.getString(R.string.app_name))
            .setTicker(appName)
            .setContentIntent(pendingIntent)
            .build()
    }
```

- setOngoing(boolean ongoing) : 알림 리스트에서 사용자가 그것을 클릭하거나 좌우로 드래그해도 사라지지 않음 설정
- setTicker(CharSequence text) : 알림이 상태 바에 뜰 때 그 곳에 나타나는 텍스트
- setStyle(NotificationCompat.Style style) : 알림 스타일 적용(MediaStyle, InboxStyle, BigTextStyle, BigPictureStyle, MessagingStyle, Etc..)
- setVisibility(int visibility) SystemUI가 신뢰할 수없는 상황 (즉, 보안 잠금 화면에서)에서 알림의 존재와 내용을 표시하는 방법과시기에 영향을주는이 알림의 가시성 범위 설정
