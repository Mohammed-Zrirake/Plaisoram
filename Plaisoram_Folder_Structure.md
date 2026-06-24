# Plaisoram — Complete Folder Structure

> **Generated:** 2026-06-24 16:36:20
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
├── AGENTS.md
├── CHANGELOG.md
├── CLAUDE.md
├── README.md
├── biome.json
├── components.json
├── global.css
├── messages/
│   ├── en.json
│   └── fr.json
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
│   ├── logo.svg
│   ├── next.svg
│   ├── vercel.svg
│   └── window.svg
├── src/
│   ├── actions/
│   │   ├── auth.ts
│   │   ├── locale.ts
│   │   └── profile.ts
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── layout.tsx
│   │   │   ├── login/
│   │   │   │   └── page.tsx
│   │   │   └── signup/
│   │   │       └── page.tsx
│   │   ├── (dashboard)/
│   │   │   ├── apps/
│   │   │   │   └── page.tsx
│   │   │   ├── canvas/
│   │   │   │   └── page.tsx
│   │   │   ├── devices/
│   │   │   │   ├── add/
│   │   │   │   │   └── page.tsx
│   │   │   │   ├── components/
│   │   │   │   └── page.tsx
│   │   │   ├── layout.tsx
│   │   │   ├── loading.tsx
│   │   │   ├── media/
│   │   │   │   ├── components/
│   │   │   │   │   ├── FolderModal.tsx
│   │   │   │   │   └── PublishMediaModal.tsx
│   │   │   │   └── page.tsx
│   │   │   ├── page.tsx
│   │   │   ├── playlists/
│   │   │   │   ├── editLayout/
│   │   │   │   │   ├── components/
│   │   │   │   │   │   ├── DevicePickerModal.tsx
│   │   │   │   │   │   ├── MediaPickerModal.tsx
│   │   │   │   │   │   └── TVCanvas.tsx
│   │   │   │   │   └── page.tsx
│   │   │   │   └── page.tsx
│   │   │   ├── schedules/
│   │   │   │   └── page.tsx
│   │   │   └── settings/
│   │   │       ├── about/
│   │   │       │   └── page.tsx
│   │   │       ├── billing/
│   │   │       │   └── page.tsx
│   │   │       ├── companyinformations/
│   │   │       │   └── page.tsx
│   │   │       ├── layout.tsx
│   │   │       ├── logo/
│   │   │       │   └── page.tsx
│   │   │       ├── page.tsx
│   │   │       ├── profile/
│   │   │       │   └── page.tsx
│   │   │       └── support/
│   │   │           └── page.tsx
│   │   ├── api/
│   │   │   ├── [...slug]/
│   │   │   │   └── route.ts
│   │   │   └── logout/
│   │   │       └── route.ts
│   │   └── layout.tsx
│   ├── components/
│   │   ├── Dashboard/
│   │   │   ├── ActionCard.tsx
│   │   │   ├── GettingStartedWidget.tsx
│   │   │   ├── OnboardingTour.tsx
│   │   │   ├── PublishScheduleModal.tsx
│   │   │   └── UsageStatsWidget.tsx
│   │   ├── Layout/
│   │   │   └── Dashboard/
│   │   │       ├── Footer.tsx
│   │   │       ├── Sidebar.tsx
│   │   │       └── TopHeader.tsx
│   │   ├── Providers.tsx
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
│   │       ├── confirm-dialog.tsx
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
│   │   ├── use-mobile.ts
│   │   ├── useDashboard.ts
│   │   ├── useDebounce.ts
│   │   ├── useDevices.ts
│   │   ├── useMedia.ts
│   │   └── usePlaylists.ts
│   ├── i18n/
│   │   ├── request.ts
│   │   └── routing.ts
│   ├── lib/
│   │   ├── api.ts
│   │   ├── fetchClient.ts
│   │   ├── layouts.ts
│   │   └── utils.ts
│   ├── proxy.ts
│   ├── styles/
│   │   ├── favicon.ico
│   │   └── globals.css
│   └── utils/
├── tools.md
├── tsconfig.json
└── tsconfig.tsbuildinfo
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
├── bin/
│   ├── console
│   ├── phpunit
│   └── test_delete.php
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
│   │   ├── cache.yaml
│   │   ├── csrf.yaml
│   │   ├── debug.yaml
│   │   ├── doctrine.yaml
│   │   ├── doctrine_migrations.yaml
│   │   ├── flysystem.yaml
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
├── migrations/
│   ├── Version20260513163802.php
│   ├── Version20260519171003.php
│   ├── Version20260520134632.php
│   ├── Version20260520160213.php
│   ├── Version20260525140800.php
│   ├── Version20260525144200.php
│   └── Version20260526113722.php
├── phpunit.dist.xml
├── public/
│   ├── .htaccess
│   └── index.php
├── run-worker.bat
├── src/
│   ├── Kernel.php
│   ├── Modules/
│   │   ├── Device/
│   │   │   ├── Controller/
│   │   │   │   ├── DeviceController.php
│   │   │   │   └── PlayerController.php
│   │   │   ├── Entity/
│   │   │   │   └── Device.php
│   │   │   ├── EventListener/
│   │   │   │   └── DeviceStatusListener.php
│   │   │   └── Service/
│   │   │       └── DeviceMapper.php
│   │   ├── Media/
│   │   │   ├── Controller/
│   │   │   │   ├── MediaController.php
│   │   │   │   └── MediaFolderController.php
│   │   │   └── Entity/
│   │   │       ├── Media.php
│   │   │       └── MediaFolder.php
│   │   ├── Playlist/
│   │   │   ├── Command/
│   │   │   │   └── CleanupPlaylistsCommand.php
│   │   │   ├── Controller/
│   │   │   │   └── PlaylistController.php
│   │   │   ├── Entity/
│   │   │   │   ├── Playlist.php
│   │   │   │   ├── PlaylistMedia.php
│   │   │   │   ├── PlaylistSection.php
│   │   │   │   └── Zone.php
│   │   │   ├── Message/
│   │   │   │   └── PublishPlaylistMessage.php
│   │   │   ├── MessageHandler/
│   │   │   │   └── PublishPlaylistMessageHandler.php
│   │   │   └── Service/
│   │   │       └── PlaylistManager.php
│   │   ├── Schedule/
│   │   │   ├── Controller/
│   │   │   │   └── ScheduleController.php
│   │   │   └── Entity/
│   │   │       └── PublishSchedule.php
│   │   └── User/
│   │       ├── Controller/
│   │       │   ├── ProfileController.php
│   │       │   └── RegistrationController.php
│   │       ├── Entity/
│   │       │   ├── RefreshToken.php
│   │       │   ├── User.php
│   │       │   └── Workspace.php
│   │       └── Repository/
│   │           └── UserRepository.php
│   └── Shared/
│       ├── Domain/
│       │   └── .gitkeep
│       ├── Exception/
│       │   ├── .gitkeep
│       │   ├── DomainException.php
│       │   └── Playlist/
│       │       ├── CannotDeleteDefaultPlaylistException.php
│       │       ├── DuplicatePlaylistNameException.php
│       │       └── ReservedNameException.php
│       └── Infrastructure/
│           └── .gitkeep
├── symfony.lock
├── tests/
│   └── bootstrap.php
└── translations/
    ├── .gitignore
    └── messages.fr.yaml
```

---

## Player Project — Plaisoram_Player

```
Plaisoram_Player/
├── .gitignore
├── README.md
├── app/
│   ├── .gitignore
│   ├── build.gradle.kts
│   ├── proguard-rules.pro
│   └── src/
│       └── main/
│           ├── AndroidManifest.xml
│           ├── assets/
│           │   └── logo.svg
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
│           │               │   │   │   └── PlaylistItemDao.kt
│           │               │   │   └── entity/
│           │               │   │       ├── DeviceConfigEntity.kt
│           │               │   │       └── PlaylistItemEntity.kt
│           │               │   ├── remote/
│           │               │   │   ├── PlaisoramApi.kt
│           │               │   │   └── dto/
│           │               │   │       ├── InitDeviceRequestDto.kt
│           │               │   │       ├── InitDeviceResponseDto.kt
│           │               │   │       ├── PairingResponseDto.kt
│           │               │   │       ├── PlaylistItemDto.kt
│           │               │   │       └── PlaylistLayoutDto.kt
│           │               │   ├── repository/
│           │               │   │   ├── DeviceRepositoryImpl.kt
│           │               │   │   ├── MediaRepositoryImpl.kt
│           │               │   │   └── PlaylistRepositoryImpl.kt
│           │               │   ├── sync/
│           │               │   │   ├── SyncEngineImpl.kt
│           │               │   │   └── SyncWorker.kt
│           │               │   └── worker/
│           │               │       └── DownloadWorker.kt
│           │               ├── di/
│           │               │   ├── DatabaseModule.kt
│           │               │   ├── NetworkModule.kt
│           │               │   ├── RepositoryModule.kt
│           │               │   └── SyncModule.kt
│           │               ├── domain/
│           │               │   ├── model/
│           │               │   │   ├── DeviceConfig.kt
│           │               │   │   ├── PlaylistItem.kt
│           │               │   │   └── SyncStatus.kt
│           │               │   ├── repository/
│           │               │   │   ├── DeviceRepository.kt
│           │               │   │   ├── MediaRepository.kt
│           │               │   │   └── PlaylistRepository.kt
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

