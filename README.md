# Starlight ✨
> **An offline Instagram-style Android app where you are your own celebrity.**

Starlight is a 100% offline, privacy-first Android app built with modern Kotlin and Jetpack Compose (Material 3). Capture photos and videos, craft captions, assemble multi-shot collages, and watch your engagement (likes, comments, reshares, followers) simulate real celebrity growth—all without a single byte leaving your phone.

---

## 🔒 100% Offline & Private
- **Zero Internet Permission**: The app manifest does **not** declare `android.permission.INTERNET`.
- **Zero Network Libraries**: No cloud SDKs, no trackers, no external APIs.
- **App-Private Storage**: All captured images, videos, collages, and profile avatars are stored strictly in `context.filesDir`. Room stores only internal file paths and metadata.

---

## 📱 Features

### 1. Onboarding
- Profile configuration on first launch: username, bio, and optional profile picture.
- Strict username validation (not empty, no whitespace, max 30 characters).
- Persistent profile management using Jetpack DataStore Preferences.

### 2. Stories Row & Full-Screen Viewer
- **Stories Row**:
  - Horizontal carousel on Home screen.
  - First item is the user's avatar labelled *"Your story"* with a `+` badge.
  - Posts created within the last 24 hours automatically count as your story.
  - Active gradient ring (orange-to-green in light mode, purple-to-pink in dark mode) indicates active unviewed stories.
  - Neutral grey ring indicates all stories viewed or no active stories.
- **Full-Screen Story Viewer**:
  - Segmented progress bars across the top.
  - Auto-advance: 5 seconds for photos/collages, video duration for videos.
  - Tap right/left to navigate; press and hold to pause; swipe down to dismiss.
  - Automatically marks stories as seen when completed.

### 3. Vertical Feed & Media Playback
- Reverse-chronological post feed.
- Post card shows user avatar, `@username`, media, engagement icons, caption, and exact timestamp (e.g., `14 Oct 2026, 3:42 PM`).
- Inline video player (muted by default with tap-to-play/pause).
- Display-only engagement icons (hearts fill red when likes > 0).
- Post dropdown menu to **Edit caption** or **Delete post** (safely cleans up the local media file).
- Friendly empty state when no posts exist yet.

### 4. In-App Camera & Collage Mode
- **CameraX** integration: Photo mode, Video mode (with recording timer), Flip camera, Flash toggle.
- **Collage Mode**:
  - Progress indicator (`2 of 4`).
  - Supports 3 or 4 photos.
  - Single photo retake strip before finalising.
  - 4 photos compose a 2x2 grid; 3 photos compose 1 large image on top and 2 below.
  - Generated at 1080x1080 resolution with EXIF orientation correction and center-crop fit.
- **System Photo Picker**: Import existing photos/videos without requiring storage permissions.
- **Post Preview**: Review media, write a caption, and post.

### 5. Core Engagement Simulation
- **Background-to-Foreground Detection**: Powered by `ProcessLifecycleOwner` (does not fire on screen rotation or composable recomposition).
- **Constant Increment**: Increases likes, comments, and reshares of **every** post by exactly **11** (`EngagementConfig.ENGAGEMENT_INCREMENT_AMOUNT = 11L`).
- Persisted to Room and reflects immediately when the feed loads.
- Generates an Activity entry on app open (e.g. `mango.vibes and 10 others liked your post`).

### 6. Notifications, Followers & Activity Tab
- **WorkManager**: Periodic local follower notifications scheduled randomly between 15 and 60 minutes.
- **Random Fan Generator**: Produces celebrity-style usernames (e.g. `pineapple12`, `mango.vibes`, `tiger_88`).
- **Follower Growth**: Each notification increments the follower count by `others + 1` (where `others` is random from 1 to 9).
- **Activity Tab**: Real-time list of follower and like events with relative timestamps.
- **Unread Badge**: Red badge on the Activity navigation icon (`9+` when over 9), clearing automatically upon visiting the tab.

### 7. Daily Streak & Monthly Calendar View
- **Flame Hero**: Displays current streak in days with an adaptive flame icon.
- **Stats**: Longest streak, total posts, posts this week, and posts this month.
- **Interactive Calendar**: Monthly view with previous/next navigation; highlights days where at least one post was made.
- **Streak Preservation**: A streak stays alive until the end of the day after the last post.
- **Milestones**: Badges for 3, 7, 30, and 100 days that dynamically unlock when achieved.

### 8. Settings & Profile
- Edit profile (username, bio, profile photo).
- Settings screen:
  - Toggle simulated follower notifications.
  - Toggle *"Also save a copy to my phone gallery"* (MediaStore export).
  - Privacy *"About"* section explaining that all data remains local.

---

## 🎨 Dual Theme System (Material 3)

| Element | Light Theme | Dark Theme |
|---|---|---|
| **Background** | `#FFFFFF` | `#0F0B1A` |
| **Surface** | `#FAFAFA` | `#1A1426` |
| **Primary Accent** | Orange `#FF7A00` | Purple `#9B5DE5` |
| **Story Ring Gradient** | Orange `#FF7A00` → Green `#2ECC71` | Purple `#9B5DE5` → Pink `#E056FD` |
| **Seen Story Ring** | `#DBDBDB` | `#3A3050` |
| **Camera Button** | Orange-to-Green gradient | Purple-to-Pink gradient |
| **Notification Badge** | Red `#E53935` | Red `#FF5252` |
| **Heart Icon (Liked)** | Red `#E53935` | Red `#FF5252` |
| **Streak Flame** | Orange `#FF6D00` | Purple `#9B5DE5` |
| **Calendar Highlight** | Green `#2ECC71` | Purple `#9B5DE5` |
| **Unlocked Milestones** | Green `#2ECC71` | Purple `#9B5DE5` |
| **Follow Icon Accent** | Green `#2ECC71` | Lavender `#C9A7FF` |

---

## 🧠 Assumptions

1. **Story Duration**: Any post created within the last 24 hours (`createdAt >= now - 24h`) automatically qualifies for the story row. Viewing the stories marks them seen in Room.
2. **Follower Notification Interval**: Managed via chained `OneTimeWorkRequest` executions in `WorkManager` with a randomised initial delay between 15 and 60 minutes. This guarantees offline compatibility and Android Doze compliance.
3. **Streak Continuity**: As specified, the current streak stays alive until the end of the day after the last post. If you posted yesterday but haven't posted yet today, your streak is still active.
4. **Milestone Unlocking**: Milestone badges unlock when your highest achieved streak (longest streak) meets or exceeds the milestone threshold (3, 7, 30, or 100 days).
5. **Collage Aspect Ratio**: Collages render at a square 1:1 aspect ratio (1080x1080) with center-crop scaling and automatic EXIF orientation normalization.

---

## 🛠️ Tech Stack & Architecture

- **Language**: Kotlin 1.9.22
- **UI Toolkit**: Jetpack Compose with Material 3 & Navigation Compose
- **Architecture**: MVVM with unidirectional data flow (`StateFlow`)
- **Database**: Room 2.6.1 with Coroutines & Flow
- **Key-Value Store**: Jetpack DataStore Preferences
- **Camera**: CameraX 1.3.1 (ImageCapture, VideoCapture, CameraSelector)
- **Media Playback**: Media3 ExoPlayer 1.2.1
- **Image Loading**: Coil Compose 2.5.0
- **Background Tasks**: AndroidX WorkManager 2.9.0
- **Lifecycle**: `ProcessLifecycleOwner` from Lifecycle Process 2.7.0
- **Build System**: Gradle 8.5 with Kotlin DSL & Version Catalog (`libs.versions.toml`)

---

## 🚀 Building & Running

### Prerequisites
- JDK 17 (set via `JAVA_HOME`)
- Android SDK 34 (`compileSdk = 34`, `minSdk = 26`, `targetSdk = 34`)

### Compile Sources
```bash
./gradlew compileDebugSources
```

### Run Unit Tests
```bash
./gradlew testDebugUnitTest
```

### Build Debug APK
```bash
./gradlew assembleDebug
```
The output APK will be located at:
`app/build/outputs/apk/debug/app-debug.apk`
