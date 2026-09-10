---
name: speedly-firebase-notifications
description: >-
  Configure and handle Firebase Cloud Messaging (FCM), push notifications, local notifications,
  and Firebase Analytics in Speedly. Use when implementing notification listeners, background message handlers,
  or analytics events.
---

# Speedly Firebase & Notifications Skill

This skill outlines how Firebase services, push notifications (FCM), and foreground notifications (`flutter_local_notifications`) are structured and handled in **Speedly**.

## Architecture Overview

- **`FirebaseServices`**: `lib/src/core/services/firebase_services.dart`
  - FCM token retrieval and registration (`addFcmToken()`).
  - Topic subscription (e.g. `system-custom`).
  - Analytics event logging (`logSignInEvent`, `logSignUpEvent`).
- **`LocalNotificationService`**: `lib/src/core/services/local_notificaton_service.dart`
  - Displays local notifications when the app is in foreground.
  - Handles notification tap actions and routing.
- **Initialization**: Configured in `main.dart` with `DefaultFirebaseOptions.currentPlatform`.

---

## 1. Handling Incoming FCM Messages

### Foreground Notifications
When a message arrives while the app is active, display a local notification via `LocalNotificationService`:

```dart
FirebaseMessaging.onMessage.listen((RemoteMessage message) {
  if (message.notification != null) {
    LocalNotificationService.showNotification(
      title: message.notification?.title ?? '',
      body: message.notification?.body ?? '',
      payload: message.data.toString(),
    );
  }
});
```

### Background & Terminated Notifications
Handled by the top-level background handler in `main.dart`:

```dart
@pragma('vm:entry-point')
Future<void> _firebaseMessagingBackgroundHandler(RemoteMessage message) async {
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  // Handle background data synchronization if needed
}
```

---

## 2. FCM Token Lifecycle

1. Request notification permissions on app startup:
   ```dart
   await FirebaseMessaging.instance.requestPermission(
     alert: true,
     badge: true,
     sound: true,
   );
   ```
2. Retrieve and persist the token:
   ```dart
   await FirebaseServices.addFcmToken();
   ```
3. Send updated token to the backend using `ApiRoutes.addFCM` if the user is authenticated.

---

## 3. Analytics Tracking

Use `FirebaseServices.analytics` to track critical user conversions:

```dart
await FirebaseServices.analytics.logEvent(
  name: 'feature_action_performed',
  parameters: {
    'feature_name': 'quick_reply',
    'action_type': 'created',
  },
);
```

---

## Best Practices
1. **Always use `@pragma('vm:entry-point')`** for top-level background notification handlers to prevent Dart tree-shaking in release builds.
2. **Handle token refresh**: Listen to `FirebaseMessaging.instance.onTokenRefresh` to keep the backend synchronization up-to-date.
3. **Respect User Preferences**: Check `user.notificationsEnabled` before subscribing to broadcast topics or sending push tokens.
