# Kotlin + Android Project Plan (Week 3-4)

## Goal

Build comfort reading and writing Kotlin, understand the Android build ecosystem, and learn how Android-side native bridging differs from iOS in React Native.

---

## Project 1: Jetpack Compose List App (Days 1-7)

A minimal app that fetches JSON from a public API and renders it in a list — same concept as the iOS project, different platform.

### Tech Stack

- Kotlin / Jetpack Compose
- `kotlinx.coroutines` for async work
- `kotlinx.serialization` or `Gson` for JSON parsing
- `HttpURLConnection` or `OkHttp` for networking
- Android Studio project with Gradle

### Steps

1. **Set up the Android Studio project** ✅
   - [x] Create a new Empty Compose Activity project (created at repo root; package `com.davecade.composelistapp`, `compileSdk=34`, `minSdk=24`, `targetSdk=34`)
   - [x] Walk through the project structure: `build.gradle.kts` (project + module level), `AndroidManifest.xml`, `src/main/`, `res/`
   - [x] Run the default app on the Android emulator
   - [x] Note the parallels and differences with Xcode's project structure _(substituted RN/JS analogies instead, since iOS is unfamiliar)_

2. **Build the data layer**
   - [x] Pick a public API to fetch from — chose JSONPlaceholder `https://jsonplaceholder.typicode.com/posts` (fields: `userId`, `id`, `title`, `body`)
   - [x] Decide on libraries — `kotlinx.serialization` for JSON parsing + `HttpURLConnection` (JDK built-in) for networking + `kotlinx-coroutines-android` for `suspend`/dispatchers
   - [x] Wire up Gradle dependencies
     - [x] Add `coroutines` and `kotlinxSerialization` version entries to `gradle/libs.versions.toml` `[versions]` block
     - [x] Add `kotlinx-coroutines-android` and `kotlinx-serialization-json` library aliases to `[libraries]` block
     - [x] Add `kotlin-serialization` plugin alias to `[plugins]` block (version locked to Kotlin compiler version)
     - [x] Declare `kotlin-serialization` plugin in root `build.gradle.kts` with `apply false`
     - [x] Apply `kotlin-serialization` plugin in `app/build.gradle.kts`
     - [x] Add `implementation(libs.kotlinx.coroutines.android)` and `implementation(libs.kotlinx.serialization.json)` to `app/build.gradle.kts` `dependencies` block
   - [x] Add `INTERNET` permission to `AndroidManifest.xml` (as `<uses-permission>` sibling of `<application>`)
   - [ ] Create `data/` sub-package under `com.davecade.composelistapp` ⬅️ **RESUME HERE**
   - [ ] Define a Kotlin `data class Post` for the response model (annotated with `@Serializable`, fields: `userId: Int`, `id: Int`, `title: String`, `body: String`) in `data/Post.kt`
   - [ ] Write a `suspend fun fetchPosts(): List<Post>` in `data/PostsApi.kt` using `HttpURLConnection` + `Json.decodeFromString`, running on `Dispatchers.IO` via `withContext`

3. **Build the UI**
   - Create a `LazyColumn` composable that displays each item
   - Use `remember` and `mutableStateOf` for state management (note the similarity to React's `useState`)
   - Trigger the fetch from `LaunchedEffect` (similar to SwiftUI's `.task {}`)

4. **Handle edge cases**
   - Loading state (`CircularProgressIndicator`)
   - Error state
   - Empty state

5. **Polish and review**
   - Read through every file — understand what Gradle is doing, what the manifest controls, how resources work
   - Try building a signed APK (forces you through Android's keystore signing — different mental model than iOS)
   - Intentionally break the Gradle config and see what the errors look like

### Key Concepts to Internalize

- Kotlin syntax that maps to TypeScript: null safety (`?`, `!!`, `?.`, `?:`), lambdas, extension functions, `data class`
- Jetpack Compose's declarative model — nearly identical mental model to SwiftUI and React
- Gradle build system: how dependencies are declared, build variants, product flavors
- Android's permission model and manifest configuration
- The difference between `compileSdk`, `targetSdk`, and `minSdk`

---

## Project 2: Native Module Bridge to React Native (Days 8-14)

Write a Kotlin native module and expose it to a throwaway RN project. Compare the experience with iOS bridging.

### Steps

1. **Create a throwaway React Native project** (or reuse the iOS one)
   - Confirm it builds and runs on the Android emulator
   - Look at the `android/` folder structure — `app/build.gradle`, `MainApplication`, `MainActivity`

2. **Write a Kotlin native module**
   - Create a Kotlin class that extends `ReactContextBaseJavaModule`
   - Implement the same functionality as your iOS module (battery level, string reversal, etc.) for direct comparison
   - Override `getName()` to set the module name
   - Annotate methods with `@ReactMethod`

3. **Register the module**
   - Create a `ReactPackage` implementation that returns your module
   - Register the package in `MainApplication.kt` (or `MainApplication.java` if the template still uses Java)
   - On the JS side, import via `NativeModules` and call the function

4. **Compare iOS vs Android bridging**
   - Write down the differences: annotations vs macros, package registration vs bridging header, Kotlin vs Swift patterns
   - This comparison is the actual deliverable — it's what you'll reference when debugging platform-specific bridge issues at work

5. **Experiment with the bridge**
   - Pass different data types (strings, numbers, maps/dictionaries, arrays)
   - Try callbacks and Promises (`@ReactMethod` with `Promise` parameter)
   - Intentionally cause errors and read the stack traces — Android's are more verbose than iOS but also more informative

6. **Stretch: Try the New Architecture (TurboModules)**
   - If time allows, convert to a TurboModule on Android
   - Compare the Android TurboModule setup with the iOS one from the previous weeks

### Key Concepts to Internalize

- How Android's native module registration differs from iOS (package-based vs bridging header)
- Reading Android build errors from Gradle and logcat
- How `@ReactMethod` annotation works vs iOS's `RCT_EXTERN_METHOD`
- The role of `MainApplication` as the entry point for native module registration
- How to use `adb logcat` to debug native-side issues

---

## What to Skip

- Java deep dives — same reasoning as Objective-C, read it when you see it
- XML layouts (the pre-Compose way) — Compose is the modern path
- Complex Android architecture (Dagger/Hilt, Room, Navigation Component) — overkill for this exercise
- Firebase or Google Play Services integration — not relevant to the goal

## Success Criteria

- [ ] Can read a Kotlin file and understand what it does without constant reference lookups
- [ ] Can explain how Gradle builds an Android app
- [ ] Have built and run an app on the Android emulator
- [ ] Have successfully bridged a native module into React Native on Android
- [ ] Can articulate the key differences between iOS and Android native bridging in RN
- [ ] Have a written comparison of the two platforms' bridging patterns to reference on the job
