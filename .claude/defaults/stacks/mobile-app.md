# Mobile Application Stack

iOS and Android mobile applications. Use this for native or cross-platform mobile apps.

## AI Version Baseline

> **AI Training Cutoff**: May 2025
>
> See @.claude/defaults/ai-known-versions.md for detailed version confidence levels.

### React Native / Expo

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| React Native | 0.73 | Moderate |
| Expo | SDK 50 | Moderate if SDK 52+ |
| React | 18.x | Moderate if 19+ |
| TypeScript | 5.3 | Minor |

### Flutter

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| Flutter | 3.16 | Moderate if 4+ |
| Dart | 3.2 | Minor |

### Native iOS

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| Swift | 5.9 | Minor |
| SwiftUI | iOS 17 | Minor |
| Xcode | 15.x | Minor |

### Native Android

| Technology | AI Confident Version | Gap Risk |
|------------|---------------------|----------|
| Kotlin | 1.9 | Minor |
| Jetpack Compose | 1.5 | Minor |
| Android Studio | Hedgehog | Minor |

**Recommendation**: React Native/Expo for teams with web experience. Flutter for maximum code sharing. Native for platform-specific features and best performance.

## Default Stack by Framework

### React Native with Expo (Default for cross-platform)

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Framework** | Expo | Managed React Native |
| **Language** | TypeScript | Type safety |
| **Navigation** | Expo Router | File-based routing |
| **State** | Zustand / Jotai | State management |
| **Styling** | NativeWind | Tailwind for RN |
| **API** | TanStack Query | Data fetching |
| **Backend** | Supabase | BaaS with mobile SDKs |
| **Testing** | Jest + Detox | Unit + E2E |
| **Distribution** | EAS Build | Cloud builds |
| **Source Control** | GitHub | Repository hosting |

### Flutter

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Framework** | Flutter | Cross-platform UI |
| **Language** | Dart | Flutter's language |
| **State** | Riverpod / Bloc | State management |
| **Navigation** | go_router | Declarative routing |
| **API** | Dio | HTTP client |
| **Backend** | Firebase / Supabase | BaaS |
| **Testing** | flutter_test + integration_test | Testing |
| **Distribution** | Fastlane | Automated releases |
| **Source Control** | GitHub | Repository hosting |

### Native iOS (SwiftUI)

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Framework** | SwiftUI | Declarative UI |
| **Language** | Swift | Apple's language |
| **Architecture** | MVVM | Pattern |
| **Networking** | URLSession / Alamofire | HTTP |
| **Persistence** | SwiftData / Core Data | Local storage |
| **Backend** | Firebase / Supabase | BaaS |
| **Testing** | XCTest | Unit + UI testing |
| **Distribution** | App Store Connect | Distribution |
| **Source Control** | GitHub | Repository hosting |

### Native Android (Jetpack Compose)

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Framework** | Jetpack Compose | Declarative UI |
| **Language** | Kotlin | Android's language |
| **Architecture** | MVVM + Hilt | Pattern + DI |
| **Networking** | Retrofit / Ktor | HTTP |
| **Persistence** | Room | Local database |
| **Backend** | Firebase / Supabase | BaaS |
| **Testing** | JUnit + Espresso | Unit + UI testing |
| **Distribution** | Google Play Console | Distribution |
| **Source Control** | GitHub | Repository hosting |

## Testing Stack

### React Native / Expo
| Tool | Purpose |
|------|---------|
| Jest | Unit testing |
| React Native Testing Library | Component testing |
| Detox | E2E testing |

### Flutter
| Tool | Purpose |
|------|---------|
| flutter_test | Unit testing |
| integration_test | E2E testing |

### iOS
| Tool | Purpose |
|------|---------|
| XCTest | Unit + UI testing |

### Android
| Tool | Purpose |
|------|---------|
| JUnit | Unit testing |
| Espresso | UI testing |

## Logging Stack

| Framework | Logger | Error Tracking |
|-----------|--------|----------------|
| React Native | react-native-logs | Sentry |
| Flutter | logger | Sentry / Crashlytics |
| iOS | os.log | Sentry / Crashlytics |
| Android | Timber | Sentry / Crashlytics |

## When to Choose Each Framework

| Requirement | Recommended |
|-------------|-------------|
| Web team, fast iteration | React Native / Expo |
| Maximum code sharing | Flutter |
| Best iOS experience | Native SwiftUI |
| Best Android experience | Native Compose |
| Complex animations | Flutter or Native |
| AR/camera features | Native |
| Quick prototype | Expo |

## Project Structure

### Expo (React Native)
```
{project}/
├── app/               # Expo Router screens
│   ├── (tabs)/
│   ├── _layout.tsx
│   └── index.tsx
├── components/
├── hooks/
├── lib/
├── assets/
├── app.json
├── package.json
└── README.md
```

### Flutter
```
{project}/
├── lib/
│   ├── main.dart
│   ├── app/
│   ├── features/
│   └── core/
├── test/
├── android/
├── ios/
├── pubspec.yaml
└── README.md
```

### iOS (SwiftUI)
```
{project}/
├── {Project}/
│   ├── App.swift
│   ├── Views/
│   ├── Models/
│   ├── ViewModels/
│   └── Services/
├── {Project}Tests/
├── {Project}.xcodeproj
└── README.md
```

### Android (Compose)
```
{project}/
├── app/
│   └── src/
│       └── main/
│           ├── java/com/example/
│           │   ├── ui/
│           │   ├── data/
│           │   └── domain/
│           └── res/
├── build.gradle.kts
└── README.md
```

## Backend Options

| Service | Best For | Notes |
|---------|----------|-------|
| Supabase | Default choice | PostgreSQL, auth, real-time |
| Firebase | Google ecosystem | NoSQL, easy auth |
| AWS Amplify | AWS ecosystem | Full AWS integration |
| Custom API | Full control | See backend-api.md |

## Distribution

| Platform | Store | Notes |
|----------|-------|-------|
| iOS | App Store | Apple review required |
| Android | Google Play | Faster review |
| Both | TestFlight / Internal Testing | Beta distribution |

## For Non-Technical Users

When the user is non-technical, don't ask about stack choices. Simply state:

> "I'll build this mobile app using React Native with Expo - it works on both iPhone and Android from a single codebase, and is easy to update. You'll be able to put it on both app stores."

## For Technical Users

Ask about preferences:

> "For mobile apps, I recommend Expo (React Native), Flutter, or native development. What's your preference?"

Then dive into:
- Target platforms? (iOS, Android, both)
- Team background? (web, mobile, both)
- Performance requirements?
- Platform-specific features needed?
- App store distribution timeline?
