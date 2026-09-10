---
name: speedly-dio-networking
description: >-
  Implement and integrate REST API endpoints in Speedly using DioClient, ApiRoutes, BaseResponse,
  and LocalStorage token handling. Use when adding network calls, integrating backend services, or debugging HTTP requests in Speedly.
---

# Speedly Dio Networking Skill

This skill provides guidelines and patterns for making REST API calls in **Speedly** using the application's central `DioClient` and `ApiRoutes`.

## Key Network Components

- **`ApiRoutes`**: Defined in `lib/src/core/data/data_source/remote/api_routes.dart`. All endpoints and base headers (`x-api-key`, `content-Type`) are managed here.
- **`DioClient`**: Defined in `lib/src/core/data/data_source/remote/dio_client.dart`. Encapsulates `Dio` instance, connection check via `InternetCheck.connected()`, offline error routing, timeouts, and response abstraction.
- **`BaseResponse`**: Defined in `lib/src/core/data/models/response/base_reponse.dart`.
  - `SuccessReponse(dynamic object, int statusCode)`
  - `FailedReponse(String message, {int? code})`
  - `UnauthorizedResponse()`

---

## 1. Registering Endpoints in `ApiRoutes`

When adding a new API endpoint, declare it as a static property in `ApiRoutes`:

```dart
// lib/src/core/data/data_source/remote/api_routes.dart
class ApiRoutes {
  // ...
  static var myNewFeature = "$_apiUrl/feature/endpoint";
}
```

If authorization headers (Bearer Token) are required, obtain the token from `LocalStorage.getToken()`:

```dart
static Map<String, String> get authorizedHeader => {
  ...header,
  "Authorization": "Bearer ${LocalStorage.getToken()}",
};
```

---

## 2. Remote Data Source Implementation

Create a dedicated API client class for your feature inside `data/data_source/remote/`:

```dart
// lib/src/features/my_feature/data/data_source/remote/my_feature_api.dart
import 'package:speedly/src/core/data/data_source/remote/api_routes.dart';
import 'package:speedly/src/core/data/data_source/remote/dio_client.dart';
import 'package:speedly/src/core/data/models/response/base_reponse.dart';

class MyFeatureApi {
  Future<BaseResponse> fetchList({int page = 1}) async {
    return await DioClient.get(
      url: "${ApiRoutes.myNewFeature}?page=$page",
      headers: ApiRoutes.authorizedHeader,
    );
  }

  Future<BaseResponse> submitData(Map<String, dynamic> payload) async {
    return await DioClient.post(
      url: ApiRoutes.myNewFeature,
      data: payload,
      headers: ApiRoutes.authorizedHeader,
    );
  }
  
  Future<BaseResponse> uploadFile(FormData formData) async {
    return await DioClient.post(
      url: ApiRoutes.myNewFeature,
      data: formData,
      headers: ApiRoutes.authorizedHeader,
    );
  }
}
```

---

## 3. Handling Responses in Repositories & BLoCs

Handle `BaseResponse` types cleanly using pattern matching / type checking:

```dart
final response = await _api.fetchList();

if (response is SuccessReponse) {
  final rawData = response.object;
  // Parse response payload
  final items = (rawData['data'] as List).map((json) => ItemModel.fromJson(json)).toList();
  return items;
} else if (response is UnauthorizedResponse) {
  // Session expired - redirect to login if not already intercepted
  throw Exception("unauthorized");
} else if (response is FailedReponse) {
  // Show error message
  throw Exception(response.message);
}
```

---

## Best Practices
1. **Never use raw `http` or instantiate ad-hoc `Dio()`** instances. Always use `DioClient` to benefit from standard logging, offline checks, timeout configs, and language header injection (`Accept-Language`).
2. **Handle File Uploads with `FormData` and `MultipartFile`**:
   ```dart
   final formData = FormData.fromMap({
     "file": await MultipartFile.fromFile(filePath, filename: fileName),
     "title": title,
   });
   ```
3. **Cancel Tokens**: Use `DioClient.cancelToken` when operations require cancellation on screen dispose.
