# Group-7mobile-car-repair-hub
Mobile Car Repair Hub — Build Setup Guide
Project Title: Mobile Car Repair Hub
Problem: Lack of a modern tool that connects car owners to nearby garages in cases of breakdown.
Solution: A mobile Android app that connects car owners to nearby garages in cases of emergency breakdown.
MobileGarage is a native Android app (Kotlin + Jetpack Compose) with a Supabase backend. This guide covers everything needed to install the required tools and build the APK from source.
1. Requirements
Tool	Version required	Notes
JDK	17 or newer	Bundled with Android Studio — a separate install usually isn't needed
Android Studio	Latest stable (2024.x / "Ladybug" or newer)	Includes the Android SDK Manager, Gradle integration, and emulator
Android SDK	compileSdk 36, minSdk 26, targetSdk 36	Installed via Android Studio's SDK Manager
Gradle	9.3.1	Auto-downloaded by the included Gradle Wrapper — no manual install needed
Kotlin	2.2.10	Managed by Gradle, no manual install needed
Internet connection	—	Required for the first Gradle sync (downloads dependencies) and for the app to reach Supabase at runtime
2. Install Android Studio
Download Android Studio from the official site.
Run the installer and accept the default components (Android SDK, Android SDK Platform, Android Virtual Device).
On first launch, open Settings/Preferences → Languages & Frameworks → Android SDK and make sure the following are installed:
SDK Platforms tab: Android 16 (API 36) — matches `compileSdk`/`targetSdk`
SDK Tools tab: Android SDK Build-Tools, Android SDK Platform-Tools, Android SDK Command-line Tools, Android Emulator (if you want to test on a virtual device)
3. Get the project onto your machine
Unzip the project archive (or clone the repo) to a folder of your choice.
Open Android Studio → File → Open → select the `Project` folder (the one containing `settings.gradle.kts`).
Let Android Studio detect the Gradle project and prompt a Gradle Sync — allow it to run. This downloads all dependencies listed in `gradle/libs.versions.toml` (AndroidX, Compose, Supabase, Ktor, Play Services Location, etc.) from Google's and Maven Central's repositories.
> No manual dependency installation is needed — Gradle handles it automatically as long as you have an internet connection during sync.
4. Configure local environment values
The build reads two secrets from a `local.properties` file at the project root (this file is git-ignored and never committed):
```properties
sdk.dir=<path to your Android SDK, auto-filled by Android Studio>

SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your-anon-key
```
`sdk.dir` is written automatically by Android Studio the first time you open the project — you normally don't need to touch it.
`SUPABASE_URL` and `SUPABASE_ANON_KEY` must be added manually if `local.properties` doesn't already contain them. You can find these values in your Supabase project dashboard under Project Settings → API.
These values are injected into the app at compile time via `BuildConfig.SUPABASE_URL` and `BuildConfig.SUPABASE_ANON_KEY` (see `app/build.gradle.kts`).
Without valid Supabase credentials, the app will build fine but authentication/database calls will fail at runtime.
5. Build the APK
Option A — Android Studio (recommended)
Wait for Gradle sync to finish (progress bar at the bottom of the window).
Go to Build → Build Bundle(s)/APK(s) → Build APK(s).
Once complete, click the "locate" link in the notification, or find the file at:
```
   app/build/outputs/apk/debug/app-debug.apk
   ```

6. Install the APK on a device or emulator
Physical device: Enable Developer Options → USB Debugging on the phone, connect via USB, then run the app from Android Studio (▶ Run button) or `./gradlew installDebug`.
Emulator: Create a virtual device via Tools → Device Manager in Android Studio (API 26+), then hit Run.
7. Runtime permissions to be aware of
The app declares the following permissions in `AndroidManifest.xml`, used for GPS-based features (e.g. driver/mechanic location tracking, SOS requests):
`INTERNET` — required for all Supabase network calls
`ACCESS_FINE_LOCATION`
`ACCESS_COARSE_LOCATION`
`ACCESS_BACKGROUND_LOCATION`
These are requested at runtime on Android 6.0+ — no extra setup is needed to build, but you'll need to grant them on-device to test location features.
8. Tech stack summary (for reference)
Language: Kotlin 2.2.10
UI: Jetpack Compose (BOM 2024.09.00), Material 3
Navigation: Navigation Compose 2.9.5
Backend: Supabase (Auth + Postgrest) 3.4.1, via Ktor client 3.1.3
Location: Google Play Services Location 21.2.0
Build system: Gradle 9.3.1 (via wrapper), Android Gradle Plugin 9.1.0
Min/Target/Compile SDK: 26 / 36 / 36
Troubleshooting
"SDK location not found" — Open the project in Android Studio once so it can auto-generate `local.properties` with the correct `sdk.dir`, or set it manually to your SDK path (e.g. `C:\Users\<you>\AppData\Local\Android\Sdk` on Windows, `~/Library/Android/sdk` on macOS, `~/Android/Sdk` on Linux).
Gradle sync fails / dependency download errors — Check your internet connection; corporate/campus networks sometimes block `dl.google.com` or `repo.maven.apache.org`.
Build succeeds but login/data fails at runtime — Double-check `SUPABASE_URL` and `SUPABASE_ANON_KEY` in `local.properties`.
