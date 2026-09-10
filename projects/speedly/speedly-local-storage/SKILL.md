---
name: speedly-local-storage
description: >-
  Manage persistent client storage, cached user sessions, tokens, and app settings in Speedly
  using GetStorage and the LocalStorage service. Use when persisting user data, retrieving stored preferences,
  or managing cached responses.
---

# Speedly Local Storage Skill

This skill explains how local storage and session persistence are managed in **Speedly** using `GetStorage` through the central `LocalStorage` utility class.

## Storage Containers

- **Default Storage Container**: `GetStorage()` initialized in `main.dart` for general user session, tokens, locale, and cached dashboards.
- **Permissions Storage Container**: `GetStorage("permissions")` for granular feature permissions.
- **Secure Container**: `GetStorage("app_secure")` for sensitive credentials or security flags.

---

## Central `LocalStorage` Class

Located at: `lib/src/core/data/data_source/local/local_storage.dart`

### Standard Keys & Access Patterns

```dart
// User Token
await LocalStorage.setToken("jwt_token_string");
String? token = await LocalStorage.getToken();

// User Profile Object
await LocalStorage.setUser(apiUser);
ApiUser? user = await LocalStorage.getUser();

// App Locale ("ar" or "en")
await LocalStorage.setLocale("ar");
String locale = LocalStorage.getLocale();

// FCM Token
await LocalStorage.setFCMToken(token);
String? fcmToken = await LocalStorage.getFCMToken();

// Notification Toggle State
await LocalStorage.setNotificationState("1");
bool isEnabled = await LocalStorage.getNotificationState();

// Clear Session / Sign Out
await LocalStorage.clearUserData();
```

---

## Adding New Cached Fields

When persisting a new setting or cached state, add typed helper methods to `LocalStorage`:

```dart
// lib/src/core/data/data_source/local/local_storage.dart
class LocalStorage {
  // ...
  static final _myCustomKey = "my_custom_key";

  static Future<void> setCustomData(String value) async {
    await globalStorage.write(_myCustomKey, value);
  }

  static String? getCustomData() {
    return globalStorage.read(_myCustomKey);
  }
}
```

---

## Best Practices
1. **Never write raw keys directly via `GetStorage()` inside UI widgets.** Always encapsulate keys in `LocalStorage` methods to maintain single-source-of-truth.
2. **Handle Null Values Safely**: Always provide default values (e.g. `?? "ar"` or `?? false`) when reading keys that might not yet be initialized.
3. **Session Invalidation**: When logging out or handling a `401 Unauthorized` response, always clear session data via `LocalStorage.clearUserData()`.
