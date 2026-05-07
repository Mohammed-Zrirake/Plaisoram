# Plaisoram — Complete Folder Structure

> **Generated:** 2026-05-07 13:54:41
>
> **Excluded (auto-generated / not developer-authored):**
> `.git` · `node_modules` · `vendor` · `.next` · `build` · `out` · `.gradle`
> `var` · `.idea` · `.vscode` · `.kotlin` · `__pycache__`

---

## Web Project — plaisoram_web

```
plaisoram_web/
├── .env.local
├── .github/
│   └── workflows/
│       └── release.yml
├── .gitignore
├── .release-please-manifest.json
├── AGENTS.md
├── CHANGELOG.md
├── CLAUDE.md
├── README.md
├── biome.json
├── components.json
├── global.css
├── next-env.d.ts
├── next.config.ts
├── package.json
├── pnpm-lock.yaml
├── pnpm-workspace.yaml
├── postcss.config.mjs
├── public/
│   ├── file.svg
│   ├── globe.svg
│   ├── images/
│   │   └── Theme.svg
│   ├── next.svg
│   ├── vercel.svg
│   └── window.svg
├── release-please-config.json
├── src/
│   ├── actions/
│   │   └── auth.ts
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── layout.tsx
│   │   │   ├── login/
│   │   │   │   └── page.tsx
│   │   │   └── signup/
│   │   │       └── page.tsx
│   │   ├── (dashboard)/
│   │   │   ├── devices/
│   │   │   │   ├── add/
│   │   │   │   │   └── page.tsx
│   │   │   │   └── page.tsx
│   │   │   ├── layout.tsx
│   │   │   ├── media/
│   │   │   │   ├── components/
│   │   │   │   │   └── FolderModal.tsx
│   │   │   │   └── page.tsx
│   │   │   ├── page.tsx
│   │   │   └── playlists/
│   │   │       ├── editLayout/
│   │   │       │   ├── components/
│   │   │       │   │   ├── DevicePickerModal.tsx
│   │   │       │   │   ├── MediaPickerModal.tsx
│   │   │       │   │   └── TVCanvas.tsx
│   │   │       │   └── page.tsx
│   │   │       └── page.tsx
│   │   ├── api/
│   │   │   └── [...slug]/
│   │   │       └── route.ts
│   │   └── layout.tsx
│   ├── components/
│   │   ├── Dashboard/
│   │   │   ├── ActionCard.tsx
│   │   │   ├── GettingStartedWidget.tsx
│   │   │   └── UsageStatsWidget.tsx
│   │   ├── Layout/
│   │   │   └── Dashboard/
│   │   │       ├── Sidebar.tsx
│   │   │       └── TopHeader.tsx
│   │   └── ui/
│   │       ├── accordion.tsx
│   │       ├── alert-dialog.tsx
│   │       ├── alert.tsx
│   │       ├── aspect-ratio.tsx
│   │       ├── avatar.tsx
│   │       ├── badge.tsx
│   │       ├── breadcrumb.tsx
│   │       ├── button-group.tsx
│   │       ├── button.tsx
│   │       ├── calendar.tsx
│   │       ├── card.tsx
│   │       ├── carousel.tsx
│   │       ├── chart.tsx
│   │       ├── checkbox.tsx
│   │       ├── collapsible.tsx
│   │       ├── combobox.tsx
│   │       ├── command.tsx
│   │       ├── context-menu.tsx
│   │       ├── dialog.tsx
│   │       ├── direction.tsx
│   │       ├── drawer.tsx
│   │       ├── dropdown-menu.tsx
│   │       ├── empty.tsx
│   │       ├── field.tsx
│   │       ├── form.tsx
│   │       ├── hover-card.tsx
│   │       ├── input-group.tsx
│   │       ├── input-otp.tsx
│   │       ├── input.tsx
│   │       ├── item.tsx
│   │       ├── kbd.tsx
│   │       ├── label.tsx
│   │       ├── menubar.tsx
│   │       ├── native-select.tsx
│   │       ├── navigation-menu.tsx
│   │       ├── pagination.tsx
│   │       ├── popover.tsx
│   │       ├── progress.tsx
│   │       ├── radio-group.tsx
│   │       ├── resizable.tsx
│   │       ├── scroll-area.tsx
│   │       ├── select.tsx
│   │       ├── separator.tsx
│   │       ├── sheet.tsx
│   │       ├── sidebar.tsx
│   │       ├── skeleton.tsx
│   │       ├── slider.tsx
│   │       ├── sonner.tsx
│   │       ├── spinner.tsx
│   │       ├── switch.tsx
│   │       ├── table.tsx
│   │       ├── tabs.tsx
│   │       ├── textarea.tsx
│   │       ├── toggle-group.tsx
│   │       ├── toggle.tsx
│   │       └── tooltip.tsx
│   ├── hooks/
│   │   └── use-mobile.ts
│   ├── lib/
│   │   ├── api.ts
│   │   ├── layouts.ts
│   │   └── utils.ts
│   ├── proxy.ts
│   ├── styles/
│   │   ├── favicon.ico
│   │   └── globals.css
│   └── utils/
├── tools.md
└── tsconfig.json
```

---

## Server Project — Plaisoram_Server

```
Plaisoram_Server/
├── .editorconfig
├── .env
├── .env.dev
├── .env.local
├── .env.test
├── .github/
│   └── workflows/
│       └── release.yml
├── .gitignore
├── .releaserc.json
├── assets/
│   ├── app.js
│   ├── controllers/
│   │   ├── csrf_protection_controller.js
│   │   └── hello_controller.js
│   ├── controllers.json
│   ├── stimulus_bootstrap.js
│   └── styles/
│       └── app.css
├── bin/
│   ├── console
│   └── phpunit
├── compose.override.yaml
├── compose.yaml
├── composer.json
├── composer.lock
├── composer.phar
├── config/
│   ├── bundles.php
│   ├── jwt/
│   │   ├── private.pem
│   │   └── public.pem
│   ├── packages/
│   │   ├── asset_mapper.yaml
│   │   ├── cache.yaml
│   │   ├── csrf.yaml
│   │   ├── debug.yaml
│   │   ├── doctrine.yaml
│   │   ├── doctrine_migrations.yaml
│   │   ├── framework.yaml
│   │   ├── gesdinet_jwt_refresh_token.yaml
│   │   ├── lexik_jwt_authentication.yaml
│   │   ├── mailer.yaml
│   │   ├── mercure.yaml
│   │   ├── messenger.yaml
│   │   ├── monolog.yaml
│   │   ├── nelmio_cors.yaml
│   │   ├── notifier.yaml
│   │   ├── property_info.yaml
│   │   ├── routing.yaml
│   │   ├── security.yaml
│   │   ├── translation.yaml
│   │   ├── twig.yaml
│   │   ├── ux_turbo.yaml
│   │   ├── validator.yaml
│   │   └── web_profiler.yaml
│   ├── preload.php
│   ├── reference.php
│   ├── routes/
│   │   ├── framework.yaml
│   │   ├── security.yaml
│   │   └── web_profiler.yaml
│   ├── routes.yaml
│   └── services.yaml
├── importmap.php
├── mercure_app/
├── migrations/
│   ├── Version20260430104847.php
│   ├── Version20260506122040.php
│   ├── Version20260506124431.php
│   ├── Version20260506154437.php
│   └── Version20260507105000.php
├── phpunit.dist.xml
├── public/
│   ├── index.php
│   ├── tv-player.html
│   └── uploads/
│       └── media/
│           ├── 233K-views-3-7K-reactions-Every-API-Type-Explained-in-4-Minutes-69fc8859f1067.mp4
│           ├── 3-Minute-Timer-with-Music-Eternal-Calm-Music-69fc7f3fa67d9.mp4
│           ├── ChatGPT-Image-Apr-30-2026-10-23-58-PM-69fc7f477bfbb.png
│           ├── Deadline-69e8faf9af433.jpg
│           ├── Four-Minute-Meditation-Music-Video-69fb523ee9528.mp4
│           ├── GoodReads-69e8f27989112.jpg
│           ├── IMG-20231103-WA0032-69e9ebd3019b8.jpg
│           ├── Main-1280-720-69fc85864733a.png
│           ├── Main-2000-1900-69fc8589ce6f4.png
│           ├── Music1-69e9edd4dddda.mp4
│           ├── Music2-69e9ee634a58e.mp4
│           ├── Sobrus-69fc889564946.png
│           └── image-1-69eb2da4d683e.png
├── src/
│   ├── Controller/
│   │   ├── .gitignore
│   │   ├── DeviceController.php
│   │   ├── MediaController.php
│   │   ├── MediaFolderController.php
│   │   ├── PlayerController.php
│   │   ├── PlaylistController.php
│   │   └── RegistrationController.php
│   ├── Entity/
│   │   ├── .gitignore
│   │   ├── Device.php
│   │   ├── Media.php
│   │   ├── MediaFolder.php
│   │   ├── Playlist.php
│   │   ├── PlaylistMedia.php
│   │   ├── RefreshToken.php
│   │   ├── User.php
│   │   ├── Workspace.php
│   │   └── Zone.php
│   ├── EventListener/
│   │   └── DeviceStatusListener.php
│   ├── Kernel.php
│   └── Repository/
│       ├── .gitignore
│       └── UserRepository.php
├── symfony.lock
├── templates/
│   └── base.html.twig
├── tests/
│   └── bootstrap.php
└── translations/
    └── .gitignore
```

---

## Player Project — Plaisoram_Player

```
Plaisoram_Player/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── release.yml
├── .gitignore
├── Android/
│   ├── Architecture/
│   │   ├── Android App Composition.md
│   │   ├── Common architectural principles.md
│   │   ├── Data_Layer/
│   │   │   ├── Build an offline-first app/
│   │   │   │   ├── Conflict resolution.md
│   │   │   │   ├── Desing an offline-first app.md
│   │   │   │   ├── Read_Write.md
│   │   │   │   ├── Synchronization and conflict resolution.md
│   │   │   │   └── model data in an offline-first app.md
│   │   │   ├── Common Tasks.md
│   │   │   ├── Data Layer architecture.md
│   │   │   ├── Exposing API s and Naming Conventions.md
│   │   │   ├── Multiple Levels of repositories and Naming Conventions.md
│   │   │   ├── Represent business models_Types of data operations_Expose errors.md
│   │   │   └── SOT _ Threading and Lifecycle.md
│   │   ├── Domain_Layer/
│   │   │   ├── Domain layer.md
│   │   │   └── Lifecycle Threading and Common Tasks.md
│   │   ├── Manage dependencies between components And General best practices.md
│   │   ├── Recommendations for Android architecture.md
│   │   ├── Recommended app architecture.md
│   │   └── UI_Layer/
│   │       ├── Consume UI State.md
│   │       ├── Expose UI State.md
│   │       ├── Other concerns in UI_Layer.md
│   │       ├── UI State Definition.md
│   │       ├── UI layer architecture.md
│   │       └── Unidirectional Data Flow.md
│   └── Fundamentals/
│       ├── Activate components.md
│       ├── Android App Components.md
│       ├── Android Apps Fundamentals.md
│       ├── AndroidManifest.xml.md
│       └── App Startup.md
├── README.md
├── app/
│   ├── .gitignore
│   ├── build.gradle.kts
│   ├── proguard-rules.pro
│   └── src/
│       └── main/
│           ├── AndroidManifest.xml
│           ├── java/
│           │   └── com/
│           │       └── sobrus/
│           │           └── plaisoramplayer/
│           │               ├── BootReceiver.kt
│           │               ├── MainActivity.kt
│           │               ├── PlaisoramPlayerApp.kt
│           │               ├── common/
│           │               │   └── util/
│           │               │       └── Resource.kt
│           │               ├── data/
│           │               │   ├── local/
│           │               │   │   ├── AppDatabase.kt
│           │               │   │   ├── Converters.kt
│           │               │   │   ├── dao/
│           │               │   │   │   ├── DeviceConfigDao.kt
│           │               │   │   │   ├── NewsDao.kt
│           │               │   │   │   ├── PlaylistItemDao.kt
│           │               │   │   │   └── WeatherDao.kt
│           │               │   │   └── entity/
│           │               │   │       ├── DeviceConfigEntity.kt
│           │               │   │       ├── NewsEntity.kt
│           │               │   │       ├── PlaylistItemEntity.kt
│           │               │   │       └── WeatherEntity.kt
│           │               │   ├── remote/
│           │               │   │   ├── NewsApi.kt
│           │               │   │   ├── PlaisoramApi.kt
│           │               │   │   ├── WeatherApi.kt
│           │               │   │   └── dto/
│           │               │   │       ├── InitDeviceRequestDto.kt
│           │               │   │       ├── InitDeviceResponseDto.kt
│           │               │   │       ├── PairingResponseDto.kt
│           │               │   │       ├── PlaylistItemDto.kt
│           │               │   │       ├── PlaylistLayoutDto.kt
│           │               │   │       └── WeatherDto.kt
│           │               │   ├── repository/
│           │               │   │   ├── DeviceRepositoryImpl.kt
│           │               │   │   ├── MediaRepositoryImpl.kt
│           │               │   │   ├── NewsRepositoryImpl.kt
│           │               │   │   ├── PlaylistRepositoryImpl.kt
│           │               │   │   └── WeatherRepositoryImpl.kt
│           │               │   ├── sync/
│           │               │   │   ├── SyncEngineImpl.kt
│           │               │   │   └── SyncWorker.kt
│           │               │   └── worker/
│           │               │       ├── DownloadWorker.kt
│           │               │       ├── NewsSyncWorker.kt
│           │               │       └── WeatherSyncWorker.kt
│           │               ├── di/
│           │               │   ├── DatabaseModule.kt
│           │               │   ├── NetworkModule.kt
│           │               │   ├── RepositoryModule.kt
│           │               │   └── SyncModule.kt
│           │               ├── domain/
│           │               │   ├── model/
│           │               │   │   ├── DeviceConfig.kt
│           │               │   │   ├── NewsArticle.kt
│           │               │   │   ├── PlaylistItem.kt
│           │               │   │   ├── SyncStatus.kt
│           │               │   │   └── WeatherData.kt
│           │               │   ├── repository/
│           │               │   │   ├── DeviceRepository.kt
│           │               │   │   ├── MediaRepository.kt
│           │               │   │   ├── NewsRepository.kt
│           │               │   │   ├── PlaylistRepository.kt
│           │               │   │   └── WeatherRepository.kt
│           │               │   ├── sync/
│           │               │   │   └── SyncEngine.kt
│           │               │   └── usecase/
│           │               │       ├── GetActivePlaylistUseCase.kt
│           │               │       ├── GetNewsUseCase.kt
│           │               │       ├── GetWeatherUseCase.kt
│           │               │       └── SyncPlaylistUseCase.kt
│           │               ├── presentation/
│           │               │   ├── news/
│           │               │   │   └── NewsViewModel.kt
│           │               │   ├── pairing/
│           │               │   │   └── PairingViewModel.kt
│           │               │   ├── player/
│           │               │   │   └── PlayerViewModel.kt
│           │               │   └── weather/
│           │               │       └── WeatherViewModel.kt
│           │               └── ui/
│           │                   ├── components/
│           │                   │   ├── ImagePlayer.kt
│           │                   │   ├── LayoutCompositor.kt
│           │                   │   ├── MultiZoneLayout.kt
│           │                   │   ├── NewsWidget.kt
│           │                   │   ├── VideoPlayer.kt
│           │                   │   ├── WeatherWidget.kt
│           │                   │   └── ZonedLayoutRenderer.kt
│           │                   ├── pairing/
│           │                   │   └── PairingScreen.kt
│           │                   ├── player/
│           │                   │   └── PlayerScreen.kt
│           │                   └── theme/
│           │                       ├── Color.kt
│           │                       ├── Theme.kt
│           │                       └── Type.kt
│           └── res/
│               ├── drawable/
│               │   ├── ic_launcher_background.xml
│               │   └── ic_launcher_foreground.xml
│               ├── mipmap-anydpi-v26/
│               │   ├── ic_launcher.xml
│               │   └── ic_launcher_round.xml
│               ├── mipmap-hdpi/
│               │   ├── ic_launcher.webp
│               │   └── ic_launcher_round.webp
│               ├── mipmap-mdpi/
│               │   ├── ic_launcher.webp
│               │   └── ic_launcher_round.webp
│               ├── mipmap-xhdpi/
│               │   ├── ic_launcher.webp
│               │   └── ic_launcher_round.webp
│               ├── mipmap-xxhdpi/
│               │   ├── ic_launcher.webp
│               │   └── ic_launcher_round.webp
│               ├── mipmap-xxxhdpi/
│               │   ├── ic_launcher.webp
│               │   └── ic_launcher_round.webp
│               ├── values/
│               │   ├── colors.xml
│               │   ├── strings.xml
│               │   └── themes.xml
│               └── xml/
│                   ├── backup_rules.xml
│                   └── data_extraction_rules.xml
├── build.gradle.kts
├── gradle/
│   ├── gradle-daemon-jvm.properties
│   ├── libs.versions.toml
│   └── wrapper/
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradle.properties
├── gradlew
├── gradlew.bat
├── local.properties
└── settings.gradle.kts
```

---

