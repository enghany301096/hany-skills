---
name: speedly-easy-localization
description: >-
  Add, update, and manage multilingual localization keys in Speedly using easy_localization,
  assets/translations/ar.json, and assets/translations/en.json. Use when adding UI strings, error messages,
  or fixing missing translations in Arabic or English.
---

# Speedly Easy Localization Skill

This skill guides the localization workflow for **Speedly**, ensuring all UI text, validation messages, and server errors are fully translated into Arabic (`ar`) and English (`en`) using `easy_localization`.

## File Locations
- Arabic translations: `assets/translations/ar.json`
- English translations: `assets/translations/en.json`

---

## 1. Key Naming Standards
- Use **`snake_case`** for all keys (e.g., `user_profile_title`, `save_changes_btn`, `invalid_phone_error`).
- Use contextual prefixes for clarity:
  - Screens: `profile_...`, `chat_...`, `auth_...`
  - Actions/Buttons: `..._btn`, `..._action`
  - Errors/Validation: `..._error`, `invalid_...`
  - Dialogs: `dialog_...`, `confirm_...`

---

## 2. Adding New Translations

Always update **BOTH** `ar.json` and `en.json` simultaneously to avoid runtime fallback errors.

### Example in `assets/translations/en.json`:
```json
{
  "quick_reply_title": "Quick Replies",
  "quick_reply_description": "Set up automated instant replies for your customers.",
  "add_reply_btn": "Add Reply"
}
```

### Example in `assets/translations/ar.json`:
```json
{
  "quick_reply_title": "الردود السريعة",
  "quick_reply_description": "قم بإعداد الردود التلقائية الفورية لعملائك.",
  "add_reply_btn": "إضافة رد"
}
```

---

## 3. Consuming Translations in Code

### Static Strings
```dart
import 'package:easy_localization/easy_localization.dart';

Text("quick_reply_title".tr())
```

### Dynamic / Formatted Strings
In JSON:
```json
"welcome_user": "Welcome back, {}!",
"items_count": "You have {count} messages"
```
In Dart:
```dart
// Positional arguments
"welcome_user".tr(args: [userName])

// Named arguments
"items_count".tr(namedArgs: {"count": messages.length.toString()})
```

### In BLoCs and Data Layers
Use `.tr()` directly on error keys before emitting error states:
```dart
emit(MyFeatureError("something_went_wrong".tr()));
```

---

## Best Practices
1. **Never Hardcode Strings**: Every label, hint, button, error message, and dialog title MUST use `.tr()`.
2. **Synchronize Keys**: If a key is added to `en.json`, it must be present with the Arabic translation in `ar.json`.
3. **RTL / LTR Awareness**: Do not hardcode directional widgets (`EdgeInsets.only(left: 10)`). Prefer `EdgeInsetsDirectional.only(start: 10)` or `Directionality.of(context)` when building custom asymmetric layouts.
