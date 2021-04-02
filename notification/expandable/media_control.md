# Create an Media Control Style Notification

## MediaStyle 알림 만들기
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
