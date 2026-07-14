# Player Refactoring & Enhancement Reference

This document records the optimization checklist for the Android Player app (`VideoPlayer.kt` and `ImagePlayer.kt`), outlining the changes needed to prevent native memory leaks, CPU spikes, and aspect-ratio distortion.

---

## 1. Video Player Enhancements (`VideoPlayer.kt`)

* **Goal:** Avoid recreating the `ExoPlayer` instance on every single video path change. Recreating the player is expensive and leads to stutters/crashes on cheap boxes.
* **Proposed Implementation:**
  ```kotlin
  @OptIn(UnstableApi::class)
  @Composable
  fun VideoPlayer(
      videoPath: String,
      onVideoEnded: () -> Unit,
      isMuted: Boolean = false,
      modifier: Modifier = Modifier
  ) {
      val context = LocalContext.current

      // 1. Create player ONCE — survives recompositions and path changes
      val exoPlayer = remember {
          val renderersFactory = DefaultRenderersFactory(context)
              .setEnableDecoderFallback(true)

          val loadControl = DefaultLoadControl.Builder()
              .setBufferDurationsMs(20000, 50000, 2500, 5000)
              .build()

          ExoPlayer.Builder(context, renderersFactory)
              .setLoadControl(loadControl)
              .build().apply {
                  playWhenReady = true
                  repeatMode = Player.REPEAT_MODE_OFF // Off to allow STATE_ENDED
                  videoScalingMode = C.VIDEO_SCALING_MODE_SCALE_TO_FIT
              }
      }

      // Track if we've already fired onVideoEnded for this item
      var endedCalled by remember { mutableStateOf(false) }

      // 2. Listen for playback completion inside a clean DisposableEffect
      DisposableEffect(exoPlayer) {
          val listener = object : Player.Listener {
              override fun onPlaybackStateChanged(state: Int) {
                  if (state == Player.STATE_ENDED && !endedCalled) {
                      endedCalled = true
                      onVideoEnded()
                  }
              }
          }
          exoPlayer.addListener(listener)
          onDispose { exoPlayer.removeListener(listener) }
      }

      // Reset ended flag when path changes
      LaunchedEffect(videoPath) {
          endedCalled = false
      }

      // 3. Swap media source when path changes — NEVER recreate the player
      LaunchedEffect(videoPath) {
          if (videoPath.isNotEmpty()) {
              exoPlayer.stop()
              exoPlayer.clearMediaItems()
              val uri = Uri.fromFile(File(videoPath))
              exoPlayer.setMediaItem(MediaItem.fromUri(uri))
              exoPlayer.prepare()
              exoPlayer.play()
          }
      }

      // Dynamic volume
      LaunchedEffect(isMuted) {
          exoPlayer.volume = if (isMuted) 0.0f else 1.0f
      }

      // 4. Release player ONLY when this composable leaves composition entirely
      DisposableEffect(Unit) {
          onDispose {
              exoPlayer.stop()
              exoPlayer.clearMediaItems()
              exoPlayer.release()
          }
      }

      AndroidView(
          factory = { ctx ->
              PlayerView(ctx).apply {
                  layoutParams = ViewGroup.LayoutParams(
                      ViewGroup.LayoutParams.MATCH_PARENT,
                      ViewGroup.LayoutParams.MATCH_PARENT
                  )
                  useController = false
                  resizeMode = AspectRatioFrameLayout.RESIZE_MODE_FILL
                  player = exoPlayer
              }
          },
          modifier = modifier.fillMaxSize()
      )
  }
  ```

---

## 2. Image Player Enhancements (`ImagePlayer.kt`)

* **Goal:** Prevent visual "loading flash" glitches on local file loads, and avoid image aspect ratio distortion.
* **Proposed Implementation:**
  ```kotlin
  SubcomposeAsyncImage(
      model = imageRequest,
      contentDescription = "Signage Image",
      contentScale = ContentScale.Crop, // Crop preserves aspect ratio and fills screen
      modifier = modifier.fillMaxSize(),
      // No loading slot — local files load instantly; avoids progress indicator flash
      error = {
          Box(
              modifier = Modifier.fillMaxSize(),
              contentAlignment = Alignment.Center
          ) {
              Text(
                  text = "Image error",
                  color = MaterialTheme.colorScheme.error
              )
          }
      }
  )
  ```
