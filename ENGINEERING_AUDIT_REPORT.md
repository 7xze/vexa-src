# Vexa — Engineering Audit Report

> **Source:** `element-hq/element-x-android` (Element X Android)
> **Date:** 2026-07-18
> **Scope:** Full codebase analysis for Vexa transformation
> **Status:** READ-ONLY — No modifications made

---

## Table of Contents

1. [Project Structure](#1-project-structure)
2. [Technologies & Libraries](#2-technologies--libraries)
3. [Architecture](#3-architecture)
4. [Data Flow](#4-data-flow)
5. [Login Flow](#5-login-flow)
6. [Messaging System](#6-messaging-system)
7. [Call System (VoIP)](#7-call-system-voip)
8. [Synapse Connection](#8-synapse-connection)
9. [Matrix SDK Usage](#9-matrix-sdk-usage)
10. [State Management](#10-state-management)
11. [Dependency Injection](#11-dependency-injection)
12. [Navigation](#12-navigation)
13. [Local Database](#13-local-database)
14. [Local Storage](#14-local-storage)
15. [Encryption](#15-encryption)
16. [Notifications](#16-notifications)
17. [Files & Responsibilities](#17-files--responsibilities)
18. [Dead Code & Unused Files](#18-dead-code--unused-files)
19. [Architectural Issues](#19-architectural-issues)
20. [Performance Issues](#20-performance-issues)
21. [Security Issues](#21-security-issues)
22. [Vexa Transformation Blockers](#22-vexa-transformation-blockers)

---

## 1. Project Structure

### 1.1 File Statistics

| File Type | Count |
|-----------|-------|
| Kotlin (.kt) | ~4,137 |
| XML (.xml) | ~2,318 |
| Java (.java) | 0 |
| Gradle Script (.kts) | ~273 |
| PNG (.png) | ~4,306 |

### 1.2 Architecture File Distribution

| Pattern | Count |
|---------|-------|
| `*Node.kt` | 123 |
| `*FlowNode.kt` | 27 |
| `*Presenter.kt` | 138 |
| `*View.kt` | 190 |
| `*State.kt` | 144 |
| `*StateProvider.kt` | 130 |
| `*Event.kt` | 65 |
| `*EntryPoint*.kt` | 145 |

### 1.3 Gradle Module Map (~250+ modules)

#### Root-Level Modules

| Module | Description |
|--------|-------------|
| `:app` | Main application module |
| `:appnav` | App navigation wiring |
| `:appconfig` | App configuration (branding, URLs) |
| `:appicon:element` | Element brand app icon |
| `:appicon:enterprise` | Enterprise brand app icon |
| `:annotations` | Custom annotations (`@ContributesNode`) |
| `:codegen` | Code generation utilities |

#### Feature Modules (45 features)

Each follows the **3-module pattern** (`api`/`impl`/`test`) with **6-file pattern** per screen (`Node`, `Presenter`, `View`, `State`, `Event`, `StateProvider`).

**Auth & Onboarding:**
| Feature | Sub-modules |
|---------|-------------|
| `login` | api, impl, test |
| `ftue` | api, impl, test |
| `lockscreen` | api, impl, test |
| `logout` | api, impl, test |
| `deactivation` | api, impl, test |
| `linknewdevice` | api, impl, test |
| `verifysession` | api, impl, test |

**Messaging & Rooms:**
| Feature | Sub-modules |
|---------|-------------|
| `messages` | api, impl, test (largest module) |
| `createroom` | api, impl, test |
| `startchat` | api, impl, test |
| `invite` | api, impl, test |
| `invitepeople` | api, impl |
| `forward` | api, impl, test |
| `share` | api, impl, test |
| `reportroom` | api, impl, test |
| `knockrequests` | api, impl, test |
| `poll` | api, impl, test |

**Rooms & Spaces:**
| Feature | Sub-modules |
|---------|-------------|
| `roomdetails` | api, impl |
| `roomdetailsedit` | api, impl, test |
| `roomdirectory` | api, impl |
| `roomaliasresolver` | api, impl |
| `roommembermoderation` | api, impl |
| `space` | api, impl |
| `viewfolder` | api, impl, test |
| `joinroom` | api, impl |
| `leaveroom` | api, impl |

**Calls & Location:**
| Feature | Sub-modules |
|---------|-------------|
| `call` | api, impl, test |
| `roomcall` | api, impl |
| `location` | api, impl, test |

**Security & Privacy:**
| Feature | Sub-modules |
|---------|-------------|
| `securebackup` | api, impl |
| `securityandprivacy` | api, impl, test |
| `rolesandpermissions` | api, impl, test |

**Settings & Preferences:**
| Feature | Sub-modules |
|---------|-------------|
| `preferences` | api, impl, test |
| `analytics` | api, impl |

**Support & Diagnostics:**
| Feature | Sub-modules |
|---------|-------------|
| `rageshake` | api, impl, test |
| `troubleshoot` | api, impl |
| `licenses` | api, impl, test |
| `announcement` | api, impl, test |

**Enterprise:**
| Feature | Sub-modules |
|---------|-------------|
| `enterprise` | api, impl-foss, test |
| `contentscanner` | api, impl |
| `networkmonitor` | api, impl, test |

#### Library Modules (50 modules)

**Matrix & SDK:**
| Module | Description |
|--------|-------------|
| `rustsdk` | Pre-built matrix-rust-sdk AAR |
| `rustls-tls` | Rust-based TLS verifier |
| `matrix:api` | Matrix client interfaces |
| `matrix:impl` | Rust SDK wrapper (50+ `Rust*.kt` files) |
| `matrix:test` | Test fakes for Matrix layer |
| `matrixmedia:api/impl/test` | Matrix media handling |

**UI & Design:**
| Module | Description |
|--------|-------------|
| `compound` | Element Design System (tokens, components) |
| `designsystem` | Design system integration |
| `ui-common` | Common UI utilities |
| `ui-strings` | String resources |
| `ui-utils` | UI utilities |
| `textcomposer:lib/impl` | Rich text message composer |
| `indicator` | Call/typing indicators |
| `previewutils` | Preview utilities |

**Core Infrastructure:**
| Module | Description |
|--------|-------------|
| `architecture` | Base classes (`Presenter`, `AsyncData`, `BaseFlowNode`) |
| `di` | Metro DI setup & scopes |
| `core` | Core utilities |
| `network` | OkHttp/Retrofit networking |
| `androidutils` | Android utilities |

**Data & Storage:**
| Module | Description |
|--------|-------------|
| `cachestore:api/impl/test` | Key-value cache |
| `session-storage:api/impl/test` | Session persistence |
| `encrypted-db` | SQLCipher encrypted database |
| `preferences:api/impl/test` | DataStore preferences |
| `cachecleaner` | Cache cleanup service |

**Media:**
| Module | Description |
|--------|-------------|
| `mediapickers:api/impl/test` | Media picker UI |
| `mediaplayer:api/impl/test` | Media playback |
| `mediaupload:api/impl/test` | Media upload |
| `mediaviewer:api/impl/test` | Media viewer |
| `audio:api/impl/test` | Audio playback |
| `voiceplayer:api/impl` | Voice message player |
| `voicerecorder:api/impl/test` | Voice message recorder |

**Security:**
| Module | Description |
|--------|-------------|
| `cryptography:api/impl/test` | AES encryption |
| `push:api/impl/test` | Push notification abstraction |
| `pushproviders:firebase` | Firebase Cloud Messaging |
| `pushproviders:unifiedpush` | UnifiedPush connector |
| `pushstore:api/impl/test` | Push token storage |

**Other:**
| Module | Description |
|--------|-------------|
| `dateformatter:api/impl/test` | Date formatting |
| `deeplink:api/impl` | Deep link parsing |
| `eventformatter:api/impl/test` | Event message formatting |
| `featureflag:api/impl/test/ui` | Feature flags |
| `fullscreenintent:api/impl` | Full-screen call intents |
| `oauth:api/impl/test` | OAuth implementation |
| `permissions:api/impl/noop/test` | Permission handling |
| `qrcode` | QR code generation/scanning |
| `recentemojis:api/impl` | Recent emojis |
| `roomselect:api/impl/test` | Room selection UI |
| `slashcommands:api/impl/test` | Slash commands |
| `usersearch:api/impl/test` | User search |
| `wellknown:api/impl/test` | .well-known handling |
| `workmanager:api/impl/test` | WorkManager integration |

#### Service Modules

| Module | Description |
|--------|-------------|
| `analytics:api/impl/compose/noop/test` | Analytics abstraction |
| `analyticsproviders:posthog` | PostHog provider |
| `analyticsproviders:sentry` | Sentry provider |
| `apperror:api/impl/test` | Error reporting |
| `appnavstate:api/impl/test` | App navigation state |
| `toolbox:api/impl/test` | Developer toolbox |

#### Test Modules

| Module | Description |
|--------|-------------|
| `tests:detekt-rules` | Custom Detekt rules |
| `tests:konsist` | Architecture compliance tests |
| `tests:uitests` | Screenshot/UI tests |
| `tests:testutils` | Shared test utilities |

### 1.4 Build Flavors

| Dimension | Flavors | Notes |
|-----------|---------|-------|
| `store` | `gplay`, `fdroid` | Google Play vs F-Droid |
| Build Type | `debug`, `release`, `nightly` | Standard Android |
| Brand | FOSS vs Enterprise | `isEnterpriseBuild` flag |

---

## 2. Technologies & Libraries

### 2.1 Build System

| Component | Version |
|-----------|---------|
| Gradle | 9.5.1 |
| AGP | 9.2.1 |
| Kotlin | 2.4.0 |
| KSP | 2.3.10 |
| Java Toolchain | 21 |

### 2.2 SDK Versions

| Property | Value |
|----------|-------|
| compileSdk | 37 |
| targetSdk | 37 |
| minSdk (FOSS) | 24 |
| minSdk (Enterprise) | 33 |
| buildToolsVersion | 37.0.0 |
| Version Name | 26.07.1 (CalVer) |

### 2.3 Core Libraries

| Category | Library | Version |
|----------|---------|---------|
| **UI** | Jetpack Compose BOM | 2026.06.01 |
| **UI** | Material 3 | 1.5.0-alpha22 |
| **UI** | Material 3 Adaptive | 1.0.0-alpha06 |
| **Navigation** | Appyx | 1.7.1 |
| **Presentation** | Molecule | 2.2.0 |
| **State** | FlowRedux | 1.2.2 |
| **DI** | Metro | 1.3.0 |
| **Networking** | OkHttp | 5.4.0 (BOM) |
| **Networking** | Retrofit | 3.0.0 (BOM) |
| **Database** | SQLDelight | 2.3.2 |
| **Encryption DB** | SQLCipher | 4.17.0 |
| **Storage** | DataStore Preferences | 1.2.1 |
| **Images** | Coil 3 | 3.5.0 |
| **Serialization** | KotlinX Serialization JSON | 1.11.0 |
| **Rich Text** | WYSIWYG | 2.42.0 |
| **Calls** | Element Call Embedded | 0.20.3 |
| **Media** | Media3 (ExoPlayer) | 1.10.1 |
| **Camera** | CameraX | 1.6.1 |
| **Maps** | MapLibre GL | 13.3.1 |
| **Push** | Firebase BOM | 34.15.0 |
| **Push** | UnifiedPush | 3.3.3 |
| **Analytics** | PostHog | 3.51.1 |
| **Analytics** | Sentry | 8.48.0 |
| **Crypto** | Google Tink | 1.23.0 |
| **Permissions** | Accompanist | 0.37.3 |
| **Blur** | Haze | 1.7.2 |
| **Zoom** | Telephoto | 0.19.0 |
| **Code Gen** | KotlinPoet | 2.3.0 |

### 2.4 Matrix SDK

| Component | Version/Source |
|-----------|---------------|
| matrix-rust-sdk | `org.matrix.rustcomponents:sdk-android:26.07.15` |
| JNA | 5.19.1 (FFI bridge) |
| NDK ABIs | armeabi-v7a, arm64-v8a, x86, x86_64 |

### 2.5 Testing

| Tool | Purpose |
|------|---------|
| JUnit 4 | Unit testing |
| MockK | Mocking |
| Turbine | Flow testing |
| Robolectric | Android unit tests |
| Roborazzi | Screenshot tests |
| Paparazzi | UI screenshot tests |
| Detekt | Static analysis |
| Konsist | Architecture compliance |
| Kover | Code coverage (70%/85%/90%) |

### 2.6 Custom Convention Plugins

| Plugin | Purpose |
|--------|---------|
| `io.element.android-compose-application` | App module with Compose |
| `io.element.android-compose-library` | Library with Compose |
| `io.element.android-library` | Library without Compose |
| `io.element.jvm-library` | Pure JVM library |

---

## 3. Architecture

### 3.1 High-Level Pattern

Single-activity, full **Jetpack Compose** application using **Matrix Rust SDK** via FFI bindings. Three pillars:

| Pillar | Library | Role |
|--------|---------|------|
| Navigation | **Appyx** | Model-driven, composable navigation |
| State | **Molecule** + Compose Runtime | Reactive Presenters as `@Composable` functions |
| DI | **Metro** | Compile-time dependency injection |

### 3.2 Core Architecture Types

#### `Presenter<State>` Interface

```kotlin
fun interface Presenter<State> {
    @Composable
    fun present(): State
}
```

- Single `@Composable` function returning a `State` data class
- NOT a ViewModel — leverages Compose runtime reactivity directly
- Uses `remember`, `LaunchedEffect`, `collectAsState` for state management

#### `AsyncData<T>` — Loading State Wrapper

```kotlin
sealed interface AsyncData<out T> {
    data object Uninitialized : AsyncData<Nothing>
    data class Loading<out T>(val prevData: T? = null) : AsyncData<T>
    data class Success<out T>(val data: T) : AsyncData<T>
    data class Failure<out T>(val error: Throwable, val prevData: T? = null) : AsyncData<T>
}
```

Includes `prevData` on `Loading`/`Failure` to avoid data loss during refresh.

#### `AsyncAction<T>` — User-Triggered Action Wrapper

```kotlin
sealed interface AsyncAction<out T> {
    data object Uninitialized : AsyncAction<Nothing>
    interface Confirming : AsyncAction<Nothing>
    data object Loading : AsyncAction<Nothing>
    data class Failure(val error: Throwable) : AsyncAction<Nothing>
    data class Success<out T>(val data: T) : AsyncAction<T>
}
```

Resets to `Uninitialized` after completion (unlike `AsyncData` which retains last value).

#### `BaseFlowNode<NavTarget>` — Multi-Screen Navigation

```kotlin
abstract class BaseFlowNode<NavTarget : Any>(
    val backstack: BackStack<NavTarget>,
    buildContext: BuildContext,
    plugins: List<Plugin>,
    val overlay: Overlay<NavTarget> = Overlay(null),
    val permanentNavModel: PermanentNavModel<NavTarget> = PermanentNavModel(emptySet(), null),
) : ParentNode<NavTarget>(
    navModel = overlay + backstack + permanentNavModel,
    buildContext = buildContext,
    plugins = plugins,
)
```

### 3.3 The 6-File Screen Pattern

| File | Role |
|------|------|
| `FooNode.kt` | Appyx Node — wires Presenter to View, handles navigation |
| `FooPresenter.kt` | `@Composable` function producing `FooState` from `FooEvent`s |
| `FooView.kt` | Stateless Composable rendering UI from `FooState` |
| `FooState.kt` | Immutable data class with `eventSink: (FooEvent) -> Unit` lambda |
| `FooEvent.kt` | Sealed interface for UI actions |
| `FooStateProvider.kt` | Preview parameter provider for screenshots |

### 3.4 The 3-Module Feature Pattern

```
features/foo/
  api/     — Public interfaces, data classes, EntryPoints
  impl/    — Internal implementation (Node, Presenter, View, State, Event)
  test/    — Fake implementations for testing
```

**Dependency inversion:** feature modules never depend on each other's implementations. Navigation glue lives in `appnav/`.

---

## 4. Data Flow

### 4.1 Unidirectional Data Flow (UDF)

```
Presenter → State (with eventSink lambda) → Node → View → User Action → Event → Presenter
```

**Flow:**
1. **Presenter** produces a `State` data class via `@Composable present()`
2. **State** contains an `eventSink` lambda: `(Event) -> Unit`
3. **Node** connects Presenter output to View input
4. **View** renders state and calls `state.eventSink(SomeEvent)` on user interaction
5. **Presenter** receives event via the captured `handleEvent` function

### 4.2 Communication Rules

- `Presenter` and `View` **never communicate directly**
- Communication is ONLY through `State` and `Event`
- Navigation callbacks (`onBackClick`, `onNavigate`) are **separate parameters** from events
- Zero ViewModel usage across the entire codebase

### 4.3 Example: Simple Feature (Licenses)

```kotlin
// State
data class DependencyLicensesListState(
    val licenses: AsyncData<ImmutableList<DependencyLicenseItem>>,
    val filter: String,
    val eventSink: (DependencyLicensesListEvent) -> Unit,
)

// Event
sealed interface DependencyLicensesListEvent {
    data class SetFilter(val filter: String) : DependencyLicensesListEvent
}

// Presenter
@Composable override fun present(): DependencyLicensesListState {
    var filter by remember { mutableStateOf("") }
    fun handleEvent(event: DependencyLicensesListEvent) {
        when (event) {
            is DependencyLicensesListEvent.SetFilter -> filter = event.filter
        }
    }
    return DependencyLicensesListState(
        licenses = filteredLicenses,
        filter = filter,
        eventSink = ::handleEvent,
    )
}

// View
@Composable fun DependencyLicensesListView(state: ...) {
    TextField(
        value = state.filter,
        onValueChange = { state.eventSink(DependencyLicensesListEvent.SetFilter(it)) },
    )
}
```

### 4.4 Timeline Data Flow (Complex)

```
Rust SDK Timeline → TimelineController → TimelineItemsFactory → TimelinePresenter → TimelineView
                                                ↑
                                    TimelineItemEventContent mapping
                                    (24 message types)
```

---

## 5. Login Flow

### 5.1 Login Methods

| Method | Flow |
|--------|------|
| **Password** | OnBoarding → LoginPassword → `authenticationService.login(username, password)` |
| **OAuth/OIDC** | OnBoarding → Chrome Custom Tab → OAuth callback → `client.loginWithOauthCallback(url)` |
| **QR Code** | OnBoarding → Scan QR → `authenticationService.loginWithQrCode(data)` |
| **Element Classic Import** | ClassicFlowNode → binds to Element Classic → imports session |

### 5.2 LoginOrchestrator: `LoginModePresenter`

Central presenter shared across multiple login screens:
1. Calls `authenticationService.setHomeserver(url)` — Rust SDK builds Client, discovers OAuth support
2. Checks `MatrixHomeServerDetails` for supported modes
3. Routes to `LoginMode.PasswordLogin`, `LoginMode.OAuth(oAuthDetails)`, or `LoginMode.AccountCreation(url)`

### 5.3 Post-Login: FTUE

After successful login, `FtueFlowNode` handles:
- **Session Verification** (`ChooseSessionVerificationModeNode`)
- **Notifications Opt-In** (`NotificationsOptInNode`)

### 5.4 Session Restoration

```
App Start → SessionStore → MatrixClientProvider.getOrRestore(sessionId)
  → ClientBuilder(sessionId) → client.restoreSession()
  → RustMatrixClient wraps native Client
  → LoggedInFlowNode → HomeScreen
```

---

## 6. Messaging System

### 6.1 Core Components

| Component | File | Role |
|-----------|------|------|
| MessagesFlowNode | `features/messages/impl/` | Top-level navigation (media viewers, threads, polls, pinned messages) |
| MessagesNode | `features/messages/impl/` | Wires `MessagesPresenter` to `MessagesView` |
| TimelineController | `timeline/TimelineController.kt` | Manages live + detached timelines |
| TimelinePresenter | `timeline/TimelinePresenter.kt` | Produces `TimelineState` from timeline items |
| TimelineView | `timeline/TimelineView.kt` | `LazyColumn` (reverseLayout=true) rendering timeline |
| MessageComposerPresenter | `messagecomposer/` | Text input, slash commands, voice messages |
| TextComposer | `libraries/textcomposer/` | Reusable rich text composer UI |

### 6.2 Supported Message Types (24)

| Content Type | Category |
|--------------|----------|
| `TimelineItemTextContent` | Text |
| `TimelineItemNoticeContent` | Notice |
| `TimelineItemEmoteContent` | Emote |
| `TimelineItemImageContent` | Image |
| `TimelineItemVideoContent` | Video |
| `TimelineItemFileContent` | File |
| `TimelineItemAudioContent` | Audio |
| `TimelineItemVoiceContent` | Voice message |
| `TimelineItemStickerContent` | Sticker |
| `TimelineItemLocationContent` | Location (static + live) |
| `TimelineItemPollContent` | Poll |
| `TimelineItemGalleryContent` | Multi-image gallery |
| `TimelineItemAttachmentsContent` | Multiple files |
| `TimelineItemStateContent` | State event |
| `TimelineItemRoomMembershipContent` | Membership change |
| `TimelineItemProfileChangeContent` | Profile change |
| `TimelineItemEncryptedContent` | Unable to decrypt |
| `TimelineItemRedactedContent` | Redacted message |
| `TimelineItemRtcNotificationContent` | RTC call notification |
| `TimelineItemLegacyCallInviteContent` | Legacy call invite |
| `TimelineItemUnknownContent` | Unknown |

### 6.3 Message Sending Flow

1. Extract `Message` (markdown, html, mentions) from editor state
2. Check for **slash commands** via `SlashCommandService.parse()`
3. Reset composer immediately
4. Based on `MessageComposerMode`:
   - **Normal/Attachment** → `timeline.sendMessage(body, htmlBody, mentions)`
   - **Edit** → `timeline.editMessage(eventId, markdown, html, mentions)`
   - **EditCaption** → `timeline.editCaption(eventId, caption)`
   - **Reply** → `timeline.replyMessage(body, htmlBody, mentions, repliedToEventId)`
5. Notify `notificationConversationService.onSendMessage()`
6. Capture analytics event

### 6.4 Timeline Performance

- `LazyColumn` with `reverseLayout=true` (newest at bottom)
- `contentType` and `key` on every item for optimal recomposition
- `TimelinePrefetchingHelper` with debounce + conflate for pagination
- `@Immutable` on `TimelineItem` sealed interface
- `ImmutableList<TimelineItem>` in state

---

## 7. Call System (VoIP)

### 7.1 Architecture

Element Call is implemented via **WebView** embedding the `element-call` (matrix-js-sdk based) widget.

```
MessagesNode → CallData → elementCallEntryPoint.startCall()
  → ElementCallActivity → CallScreenPresenter
  → WebView (Element Call widget URL)
  → Bidirectional WidgetMessage communication
  → CallForegroundService (audio focus, notification)
```

### 7.2 Call Flow

**Outgoing:**
1. User taps call button in messages
2. `ElementCallEntryPoint.startCall()` launches `ElementCallActivity`
3. `CallScreenPresenter` creates widget URL with auth token, room ID, etc.
4. WebView loads Element Call widget
5. `CallForegroundService` starts for audio focus + notification
6. Bidirectional `WidgetMessage` protocol (Join/HangUp/Close/SendEvent)

**Incoming:**
1. Push notification → `ActiveCallManager.registerIncomingCall()`
2. Ringing notification (PRIORITY_MAX, CATEGORY_CALL, full-screen intent)
3. `IncomingCallActivity` (full-screen intent) → accept/reject
4. → `ElementCallActivity`

**Hangup:**
1. HangUp WidgetMessage → 2s delay → activity closes → service stops → audio focus released

### 7.3 Call Features

| Feature | Implementation |
|---------|---------------|
| Picture-in-Picture | WebView PiP API, 3:5 ratio, auto-enter on Android S+ |
| Permissions | Android runtime (RECORD_AUDIO, CAMERA) → WebKit permissions |
| Full-screen intents | `FullscreenIntentManager` for incoming calls |
| Enterprise control | `isElementCallAvailable()` hides call buttons when restricted |

---

## 8. Synapse Connection

### 8.1 Homeserver Discovery

```
User enters URL → HomeserverResolver
  → Direct URL check
  → Fallback: append .org/.com/.io
  → Rust SDK fetches .well-known/matrix/client
  → Extracts actual homeserver URL
  → Compatibility check via inMemoryStore()
  → Client created with resolved URL
```

### 8.2 Well-Known Delegation

| Source | Redirected To |
|--------|---------------|
| `matrix.org` | SDK fetches `.well-known/matrix/client` → `matrix-client.matrix.org` |
| Custom domain | Same pattern — .well-known delegation |

### 8.3 Element Well-Known Extensions

Beyond standard Matrix `.well-known`, the app also checks for Element-specific fields:

| Field | Purpose |
|-------|---------|
| `registration_helper_url` | Web-based registration |
| `rageshake_url` | Bug reports |
| `content_scanner_url` | Content scanning |
| `force_disable_e2ee` | Force disable encryption |

### 8.4 Server Compatibility

The `HomeServerLoginCompatibilityChecker` checks:
- Sliding Sync support (required)
- Login methods supported (password, OAuth, QR)
- Account creation availability
- Enterprise requirements (Element Pro)

### 8.5 Sliding Sync Configuration

| Phase | Strategy |
|-------|----------|
| Login | `SlidingSyncDiscoverMethod.DISCOVER_NATIVE` |
| Restored session | `RESTORED` or `NATIVE` |

Client builder config: SQLite store, 30s timeout, 3 retries, cross-process lock, auto cross-signing.

---

## 9. Matrix SDK Usage

### 9.1 Architecture Layers

```
Rust SDK (matrix-rust-sdk AAR)          ← Native Rust, JNI/FFI
       ↓
libraries/rustsdk/                       ← Pre-built AAR wrapper
       ↓
libraries/matrix/impl                    ← RustMatrixClient (50+ Rust*.kt files)
       ↓
libraries/matrix/api                     ← MatrixClient interface (pure Kotlin)
       ↓
features/                                ← Login, Rooms, Messages, etc.
```

### 9.2 Key SDK Interfaces

**`MatrixClient`** — Core client interface:
- Properties: `sessionId`, `deviceId`, `homeserverUrl`, `userProfile`
- Services: `roomListService`, `spaceService`, `syncService`, `sessionVerificationService`, `pushersService`, `notificationService`, `encryptionService`, `roomDirectoryService`
- Operations: room management, user management, media upload, account management

**`MatrixAuthenticationService`** — Login/auth:
- `setHomeserver(url)` → creates Client, returns capabilities
- `login(username, password)` → password login
- `getOAuthUrl(prompt, loginHint)` → OAuth initiation
- `loginWithOAuth(callbackUrl)` → OAuth completion
- `loginWithQrCode(qrCodeData, progress)` → QR login
- `importCreatedSession(externalSession)` → Element Classic import
- `restoreSession(sessionId)` → session restoration

### 9.3 Rust SDK Configuration

```kotlin
ClientBuilder(sessionId)
    .sqliteDbPath(sqliteDbPath)
    .sessionDelegate(sessionDelegate)
    .enableCrossSigningAutoDiscovery(true)
    .slidingSyncProxy(discoverMethod)
    .withUtdHook(utdHook)
    .serverVersionsPolicy(...)
    .buildWithSession(sessionData.toSession())
```

Timeout: 30s, Retries: 3, Cross-process lock enabled.

### 9.4 Rust*.kt Wrapper Pattern

50+ files in `libraries/matrix/impl/` map Rust SDK types to Kotlin domain types:
- `RustMatrixClient.kt` (921 lines) — wraps native `Client`
- `RustMatrixAuthenticationService.kt` — wraps auth operations
- `RustMatrixRoomListService.kt` — wraps room list
- `RustEncryptionService.kt` — wraps encryption
- etc.

---

## 10. State Management

### 10.1 Core Pattern

- **No ViewModel** anywhere in the codebase
- All state lives in **Compose runtime** via `remember` / `mutableStateOf`
- **Molecule** bridges Compose runtime with Presenters
- **FlowRedux** used in specific state machines (e.g., `SecureBackupSetupStateMachine`)

### 10.2 State Data Class Pattern

```kotlin
data class FooState(
    val data: AsyncData<SomeData>,
    val isLoading: Boolean,
    val eventSink: (FooEvent) -> Unit,  // Event channel
)
```

### 10.3 Reactive Data Sources

| Source | Usage |
|--------|-------|
| `remember` + `mutableStateOf` | Local presenter state |
| `LaunchedEffect` | Async operations |
| `collectAsState()` | Flow collection |
| `snapshotFlow` | Reactive triggers |
| `combine()` | Multiple flow merging |

### 10.4 Stability Annotations

- Automated **Konsist test** enforces `@Immutable` or `@Stable` on all sealed interfaces used as Composable parameters
- Widespread use across 100+ state/model types
- `ImmutableList` from `kotlinx.collections.immutable` for collection stability

---

## 11. Dependency Injection

### 11.1 Framework: Metro

`dev.zacsweers.metro` — compile-time DI (replaced Dagger/Anvil).

### 11.2 Scope Hierarchy

| Scope | Lifetime |
|-------|----------|
| `AppScope` | Application-wide singleton |
| `SessionScope` | Per logged-in user session |
| `RoomScope` | Per active chat room |

### 11.3 Key Annotations

| Annotation | Usage |
|------------|-------|
| `@Inject` | Constructor injection |
| `@AssistedInject` + `@AssistedFactory` | Runtime arguments (Navigators, IDs) |
| `@ContributesNode(Scope)` | Auto-register Appyx Nodes in DI |
| `@ContributesBinding(Scope)` | Bind interface implementations |
| `@ContributesIntoSet` | Multi-binding (e.g., PushProviders) |
| `@SingleIn(Scope)` | Singleton within scope |

### 11.4 Qualifier Annotations

| Qualifier | Type |
|-----------|------|
| `@AppCoroutineScope` | CoroutineScope |
| `@SessionCoroutineScope` | CoroutineScope |
| `@RoomCoroutineScope` | CoroutineScope |
| `@ApplicationContext` | Context |
| `@CacheDirectory` | File |
| `@BaseDirectory` | File |
| `@SentryDsn` / `@SentrySdkDsn` | Inline value classes |

### 11.5 Session Graph Pattern

```kotlin
interface SessionGraphFactory {
    fun create(client: MatrixClient): Any
}
```

`LoggedInAppScopeFlowNode` creates the session graph, allowing child nodes to receive `SessionScope` dependencies.

### 11.6 Example: Node Injection

```kotlin
@ContributesNode(SessionScope::class)
@AssistedInject
class LoggedInFlowNode(
    @Assisted buildContext: BuildContext,
    @Assisted plugins: List<Plugin>,
    private val homeEntryPoint: HomeEntryPoint,
    // ... other injected dependencies
)
```

---

## 12. Navigation

### 12.1 Framework: Bumble Appyx

Model-driven, composable navigation using Node trees.

### 12.2 Root Structure

```
RootFlowNode (AppScope)
  ├── SplashScreen
  └── NotLoggedInFlow (SessionScope)
        └── LoginFlowNode
              ├── ClassicFlowNode (Element Classic import)
              ├── OnBoardingNode
              ├── LoginPasswordNode
              ├── QrCodeLoginFlowNode
              └── ... (account provider selection)
  └── LoggedInAppScopeFlowNode (SessionScope)
        └── LoggedInFlowNode
              ├── HomeScreen (permanent)
              ├── RoomFlowNode
              │     ├── TimelineNode
              │     ├── RoomDetailsNode
              │     └── ...
              ├── SettingsFlowNode
              └── ...
```

### 12.3 Navigation Operations

| Operation | Description |
|-----------|-------------|
| `push(target)` | Add to back stack |
| `pop()` | Remove top |
| `replace(target)` | Replace top |
| `newRoot(target)` | Clear stack, set new root |
| `singleTop(target)` | Push only if not already on top |
| `safeRoot(target)` | Clear stack only if different |

### 12.4 Deep Links

| Pattern | Example |
|---------|---------|
| App scheme | `element://open/{sessionId}/{roomId}/{threadId}/{eventId}` |
| Matrix permalinks | `https://matrix.to/#/{roomId}` |
| Login deep links | `mobile.element.io` with `account_provider` and `login_hint` |
| OAuth callback | `io.element.android.debug:/callback` |

Intent resolution priority: Deep links → OAuth → Mobile config → Matrix permalinks → Incoming shares.

### 12.5 Room Navigation

`AttachRoomOperation` limits room nodes to 5. On revisit, removes and recreates the node for fresh state.

---

## 13. Local Database

### 13.1 SQLDelight

Used for type-safe SQL with compile-time verification.

**`SessionStore`** — Session persistence:
- Fields: userId, tokens, homeserver, loginType, etc.
- Coroutine-based Flow for reactive updates
- Mutex for thread-safe writes

**`CacheStore`** — Key-value cache:
- General-purpose key-value storage
- Same pattern as SessionStore

### 13.2 SQLCipher Encrypted Database

- **SQLCipher 4.17.0** with `SupportOpenHelperFactory`
- Key generated by `RandomDatabaseSecretProvider`: 32-byte `SecureRandom`
- Key stored in `EncryptedFile` (encrypted with Android Keystore)
- Pattern: DB key encrypted at rest with hardware-backed Keystore

### 13.3 Database Architecture

```
App Start
  → EncryptedFile reads DB key from Keystore-encrypted file
  → SQLCipherDriverFactory creates SQLCipher driver with key
  → SQLDelight generates type-safe queries
  → SessionStore / CacheStore provide reactive access
```

---

## 14. Local Storage

### 14.1 Storage Layers

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Session persistence | SQLDelight + SQLCipher | Encrypted session data |
| Key-value cache | SQLDelight + SQLCipher | General caching |
| Preferences | DataStore Preferences | App + session settings |
| File encryption | AES-128-GCM (Keystore) | Encrypted file I/O |
| In-memory cache | ConcurrentHashMap | Session client cache |

### 14.2 Preferences Stores

**`AppPreferencesStore`** — App-wide settings:
- Developer mode
- Theme
- Call ringtone
- Weak biometrics enabled

**`SessionPreferencesStore`** — Per-session settings:
- Share presence
- Read receipts
- Typing notifications
- Media compression

### 14.3 File Encryption

```
KeyStoreSecretKeyRepository
  → Retrieves/generates AES key from Android Keystore
  → KeyGenParameterSpec (hardware-backed)
  → AESEncryptionDecryptionService
    → AES-128-GCM encrypt/decrypt
    → Random IV per encryption
    → EncryptedFile for file I/O
```

---

## 15. Encryption

### 15.1 End-to-End Encryption (E2EE)

Handled entirely by **matrix-rust-sdk** (Rust FFI layer):
- Olm/Megolm protocol implementation
- Key verification via `SessionVerificationService`
- Cross-signing auto-discovery
- `EncryptionService` wraps recovery/backup operations

### 15.2 App-Level Encryption

| Component | Implementation |
|-----------|---------------|
| AES encryption | AES-128-GCM via Android Keystore |
| Key storage | `KeyStoreSecretKeyRepository` (hardware-backed) |
| DB key | 32-byte SecureRandom → EncryptedFile → Keystore |
| PIN code | AES-128-GCM encrypted, stored in DataStore |

### 15.3 Secure Backup

- `SecureBackupSetupStateMachine` (FlowRedux state machine)
- Recovery key management
- Cross-signing key backup
- 4S (Secure Secret Storage and Sharing) integration

### 15.4 Biometric Authentication

- **Strong biometrics** (fingerprint)
- **Weak biometrics** (face/iris) — configurable
- **Device credentials** (PIN/pattern)
- Crypto-backed (not just gate-keeping)
- Rate limiting with configurable max attempts before logout

---

## 16. Notifications

### 16.1 Push Providers

| Provider | Use Case |
|----------|----------|
| Firebase Cloud Messaging | Google Play builds |
| UnifiedPush | F-Droid / self-hosted |

Modular via `PushProvider` interface with `@ContributesIntoSet` multi-binding.

### 16.2 Notification Processing

```
Push received → DefaultPushHandler
  → Parse notification payload
  → Route to appropriate conversation/service
  → Create Android notification channels
  → Display notification
```

### 16.3 Background Services

| Service | Purpose |
|---------|---------|
| `FetchPendingNotificationsWorker` | WorkManager-based notification polling |
| `FetchPushForegroundService` | Foreground service with WakeLock for active notifications |
| `NotificationConversationService` | Manages Android shortcuts |
| `CallForegroundService` | Active call audio focus + notification |

### 16.4 Notification Channels

Configurable per notification type with `runBlocking` for channel creation (identified issue).

---

## 17. Files & Responsibilities

### 17.1 Key Configuration Files

| File | Purpose |
|------|---------|
| `plugins/src/main/kotlin/Versions.kt` | SDK versions, CalVer versioning |
| `plugins/src/main/kotlin/config/BuildTimeConfig.kt` | Build identity, Firebase IDs |
| `plugins/src/main/kotlin/config/ModulesConfig.kt` | Feature flags, push config |
| `gradle/libs.versions.toml` | Dependency versions |
| `app/build.gradle.kts` | App module config, flavors, signing |
| `settings.gradle.kts` | Module includes |
| `app/src/main/kotlin/.../ApplicationConfig.kt` | Runtime app config |

### 17.2 Key Source Files

| File | Lines | Purpose |
|------|-------|---------|
| `RustMatrixClient.kt` | 921 | Core Rust SDK wrapper |
| `RootFlowNode.kt` | ~300 | Root navigation |
| `LoggedInFlowNode.kt` | ~200 | Main navigation |
| `MessagesFlowNode.kt` | ~150 | Messages navigation |
| `TimelinePresenter.kt` | ~200 | Timeline state |
| `MessageComposerPresenter.kt` | ~300 | Message composition |
| `LoginModePresenter.kt` | ~200 | Login orchestration |
| `CallScreenPresenter.kt` | ~200 | Call state |

---

## 18. Dead Code & Unused Files

### 18.1 Identified Issues

| Issue | Location |
|-------|----------|
| `TODO()` at runtime | `ConfigureRoomPresenter.kt:151` — will crash app |
| 81+ TODO/FIXME/HACK comments | Throughout codebase — incomplete features |
| Enterprise stub implementations | `features/enterprise/impl-foss/` — no-op stubs |
| `enterprise/` directory exists but is empty | No build.gradle.kts files |
| Element Classic import code | `features/login/impl/classic/` — legacy feature |

### 18.2 Enterprise vs FOSS

- Enterprise module exists but is empty (no build files)
- `impl-foss` provides stub implementations
- `isEnterpriseBuild` flag switches behavior at compile time

---

## 19. Architectural Issues

### 19.1 `TODO()` at Runtime — CRITICAL

**Location:** `ConfigureRoomPresenter.kt:151`

```kotlin
// Will crash the app if reached
TODO("Not yet implemented")
```

This is a runtime crash waiting to happen.

### 19.2 `runBlocking` in Production Code — HIGH

8+ locations using `runBlocking` on potentially main thread:

| File | Line | Context |
|------|------|---------|
| `RustMatrixClient.kt` | 168, 182 | Client initialization (spaceService, notificationClient) |
| `PlatformInitializer.kt` | 34-40 | App initialization |
| `RustClientSessionDelegate.kt` | 70, 93 | Session store operations |
| `HomeFlowNode.kt` | 275 | Navigation node creation |
| `NotificationChannels.kt` | 124 | Notification channel config |
| `DynamicHttpLoggingInterceptor.kt` | 33 | OkHttp interceptor |

Most have TODO comments acknowledging the issue.

### 19.3 Tight Appyx Coupling

The entire navigation system is deeply coupled to Appyx. Any migration away from Appyx would require rewriting all 123+ Node files and 27+ FlowNode files.

### 19.4 Showkase Code Generation Fragility

The Showkase code generation (for Compose previews) is sensitive to system locale — Arabic-Indic numerals corrupt generated code on Arabic-locale Windows systems.

---

## 20. Performance Issues

### 20.1 Compose Recomposition — WELL OPTIMIZED

- Automated Konsist test enforces `@Immutable`/`@Stable` on all Composable parameter types
- `ImmutableList` from kotlinx.collections.immutable
- LazyColumn with `contentType` and `key` on every item
- Timeline uses granular presenter factories per item type

### 20.2 Timeline Rendering

| Optimization | Status |
|--------------|--------|
| reverseLayout=true | Implemented |
| contentType + key | Implemented |
| Prefetching with debounce | Implemented (100ms delay + conflate) |
| Granular recomposition | Implemented (per-item presenter factories) |

### 20.3 Potential Issues

| Issue | Severity | Location |
|-------|----------|----------|
| `runBlocking` on main thread | HIGH | 8+ locations |
| `AnimatedVisibility(visible=true)` wrapping entire timeline | LOW | `TimelineView.kt:186` |
| Function references in `TimelineItem.Event` create new objects | LOW | `TimelineItem.kt:108` |
| JSON logging up to 100K chars | LOW | Debug only |

---

## 21. Security Issues

### 21.1 OkHttp Full Body Logging — HIGH

**Location:** `NetworkModule.kt:46`, `DynamicHttpLoggingInterceptor.kt:34`

- Debug builds log **full HTTP request/response bodies**
- Includes access tokens, message content, encryption keys
- AGP doc stripping mitigates for release builds
- Debug APKs are fully exposed

### 21.2 Release Signing Uses Debug Keystore — HIGH

**Location:** `app/build.gradle.kts:125`

```kotlin
getByName("release") {
    signingConfig = signingConfigs.getByName("debug")
}
```

Production release builds ship with the debug keystore. Intentional for FOSS builds but a risk.

### 21.3 Hardcoded API Keys — MEDIUM

| Key | Location |
|-----|----------|
| PostHog API key (release) | `PosthogEndpointConfigProvider.kt:36` |
| PostHog API key (debug) | `PosthogEndpointConfigProvider.kt:41` |
| Firebase App IDs | `app/build.gradle.kts:182-185` |

PostHog keys are client-safe, but still hardcoded.

### 21.4 FOSS Builds Disable Obfuscation — MEDIUM

**Location:** `default-proguard-rules.pro`

```proguard
-dontobfuscate
```

Entire app is readable via decompilation in FOSS builds.

### 21.5 Positive Security Patterns

| Pattern | Implementation |
|---------|---------------|
| E2EE | Rust SDK (Olm/Megolm) — solid |
| DB encryption | SQLCipher + Keystore-backed key — solid |
| Biometric auth | Crypto-backed (not gate-keeping) — solid |
| AES encryption | GCM mode with random IV — solid |
| PIN rate limiting | Configurable max attempts before logout — solid |

---

## 22. Vexa Transformation Blockers

### 22.1 Critical Blockers

| # | Blocker | Location | Effort |
|---|---------|----------|--------|
| 1 | **`TODO()` runtime crash** | `ConfigureRoomPresenter.kt:151` | 10 min |
| 2 | **Hardcoded Element URLs** | Multiple files | 2-4 hours |
| 3 | **Firebase/Google dependencies** | Multiple files | 4-8 hours |
| 4 | **Element branding** | `ApplicationConfig.kt`, strings | 2-4 hours |
| 5 | **PostHog/Sentry analytics** | Analytics providers | 2-4 hours |

### 22.2 URLs to Replace

| URL | Location | Purpose |
|-----|----------|---------|
| `element.io/help#encryption` | `LearnMoreConfig.kt` (5 URLs) | Help links |
| `posthog.element.io` | `PosthogEndpointConfigProvider.kt:35` | Analytics |
| `posthog.element.dev` | `PosthogEndpointConfigProvider.kt:40` | Analytics (debug) |
| `call.element.io` | `DefaultCallWidgetSettingsProvider` | Voice/video calls |
| `mobile.element.io` | `DefaultLoginIntentResolver.kt:21` | Login deep links |
| `app.element.io` | `DefaultMatrixToConverter.kt:38` | Permalink conversion |
| `matrix.to` | `MatrixConfiguration.kt:12` | Permalinks |

### 22.3 Firebase Dependencies

| Dependency | Modular? |
|------------|----------|
| Firebase App Distribution | Yes — CI/CD only |
| Firebase Cloud Messaging | Yes — via `PushProvidersConfig` |
| Google Services plugin | Yes — already commented out |
| Google Tink | Partial — dependency substitution |

**Mitigation:** `PushProvidersConfig` allows disabling Firebase. UnifiedPush is already an alternative. `gplay`/`fdroid` flavors separate Google Play builds.

### 22.4 What Needs to Change for Custom Backend

| # | Change | Files |
|---|--------|-------|
| 1 | Build identity | `BuildTimeConfig.kt` — app ID, name |
| 2 | Login flow | `DefaultLoginIntentResolver.kt` — hardcoded host |
| 3 | Deep links | `element://` scheme in manifest + converters |
| 4 | OAuth redirect | `oAuthRedirectSchemeBase` in build.gradle.kts |
| 5 | Analytics | Replace or remove PostHog/Sentry |
| 6 | Help URLs | `LearnMoreConfig.kt` — 5 URLs |
| 7 | App distribution | Firebase App Distribution removal |
| 8 | Push | Disable Firebase, use UnifiedPush or custom |
| 9 | Branding | `ApplicationConfig.kt` — names |
| 10 | Call widget | `DefaultCallWidgetSettingsProvider` — Element Call URL |

### 22.5 Architectural Strengths for Transformation

| Strength | Benefit |
|----------|---------|
| Clean api/impl/test separation | Feature replacement is straightforward |
| Metro DI with @ContributesBinding | Swap implementations without touching existing code |
| Enterprise build pattern | Already demonstrates behavior forking |
| `appconfig` module | Centralizes ~10 constants for full rebrand |
| matrix-rust-sdk is homeserver-agnostic | Works with any compliant Matrix server |
| Modular push providers | Firebase ↔ UnifiedPush ↔ Custom |
| FOSS/Enterprise flavors | Already separated |

---

## Summary of Critical Findings

| Priority | Finding | Location | Impact |
|----------|---------|----------|--------|
| **CRITICAL** | `TODO()` at runtime can crash the app | `ConfigureRoomPresenter.kt:151` | Crash |
| **HIGH** | OkHttp logs full request/response bodies in debug | `NetworkModule.kt:46` | Data leak |
| **HIGH** | 8+ `runBlocking` calls in production code | `RustMatrixClient.kt`, `PlatformInitializer.kt`, etc. | ANR risk |
| **HIGH** | Release signing uses debug keystore | `app/build.gradle.kts:125` | Integrity |
| **MEDIUM** | Hardcoded PostHog API keys in source | `PosthogEndpointConfigProvider.kt:36,41` | Security |
| **MEDIUM** | 81 TODO/FIXME/HACK comments | Throughout codebase | Stability |
| **MEDIUM** | FOSS builds disable all obfuscation | `default-proguard-rules.pro` | Security |
| **MEDIUM** | Element-specific URLs hardcoded | Multiple files | Transformation |
| **LOW** | Showkase codegen locale sensitivity | Windows Arabic locale | Build |
| **LOW** | `AnimatedVisibility(visible=true)` overhead | `TimelineView.kt:186` | Performance |

---

> **Report generated by Vexa Engineering Intelligence Core**
> **Analysis scope:** Full Element X Android codebase
> **No files were modified, refactored, or deleted during this analysis**
