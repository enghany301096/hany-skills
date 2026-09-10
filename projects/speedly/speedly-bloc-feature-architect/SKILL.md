---
name: speedly-bloc-feature-architect
description: >-
  Architect and implement new features or refactor existing ones in Speedly using Clean Architecture
  (Data, Domain, Presentation) and Flutter BLoC with flutter_screenutil, toastification, and dialogs.
  Use when creating screens, blocs, usecases, or repositories in Speedly.
---

# Speedly BLoC Feature Architecture Skill

This skill guides the design and implementation of features within the **Speedly** Flutter application, ensuring strict adherence to the project's Clean Architecture standards, state management with `flutter_bloc`, responsive styling via `flutter_screenutil`, and common UI components.

## Feature Structure

Every new feature in `lib/src/features/<feature_name>/` or subfeature within it must follow this 3-layer structure:

```text
lib/src/features/<feature_name>/
├── data/
│   ├── data_source/
│   │   ├── local/
│   │   └── remote/
│   ├── models/
│   └── repositories/
├── domain/
│   ├── entities/
│   ├── repositories/
│   └── usecase/
└── presentation/
    ├── blocs/
    │   └── <feature_name>/
    │       ├── <feature_name>_bloc.dart
    │       ├── <feature_name>_event.dart
    │       └── <feature_name>_state.dart
    ├── screens/
    └── widgets/
```

---

## Step-by-Step Implementation Workflow

### 1. Domain Layer (Entities & Use Cases)
- **Entities**: Plain Dart objects with data contracts.
- **Repository Interface**: Define abstract methods returning `Future<BaseResponse>` or domain models.
- **Use Case**: Encapsulate single business actions.

```dart
// domain/repositories/my_feature_repository.dart
abstract class MyFeatureRepository {
  Future<BaseResponse> fetchItems();
  Future<BaseResponse> submitItem({required String name});
}

// domain/usecase/my_feature_usecase.dart
class MyFeatureUsecase {
  final MyFeatureRepository repository;
  MyFeatureUsecase({MyFeatureRepository? repository})
      : repository = repository ?? MyFeatureRepositoryImpl();

  Future<BaseResponse> getItems() => repository.fetchItems();
  Future<BaseResponse> createItem(String name) => repository.submitItem(name: name);
}
```

### 2. Data Layer (Models, Data Sources & Repository Implementation)
- **Model**: Extends/maps domain entity, implements `fromJson` and `toJson`.
- **Remote Data Source**: Calls `DioClient` with endpoints from `ApiRoutes`.
- **Repository Implementation**: Connects remote/local data sources and returns responses.

```dart
// data/data_source/remote/my_feature_api.dart
class MyFeatureApi {
  Future<BaseResponse> getItems() async {
    return await DioClient.get(url: ApiRoutes.myItems);
  }
}
```

### 3. Presentation Layer (BLoC, States & Events)
- Define explicit events using `abstract class <Feature>Event`.
- Define states representing UI state transitions (`Initial`, `Loading`, `Success`, `Error`).
- Use `AppLogger.debug(...)` or `AppLogger.error(...)` for diagnostic logs.
- Trigger global loading with `showCustomLoad()` and `closeCustomLoad()` when appropriate.

```dart
// presentation/blocs/my_feature/my_feature_event.dart
abstract class MyFeatureEvent {}
class LoadMyFeatureItems extends MyFeatureEvent {}

// presentation/blocs/my_feature/my_feature_state.dart
abstract class MyFeatureState {}
class MyFeatureInitial extends MyFeatureState {}
class MyFeatureLoading extends MyFeatureState {}
class MyFeatureLoaded extends MyFeatureState {
  final List<ItemModel> items;
  MyFeatureLoaded(this.items);
}
class MyFeatureError extends MyFeatureState {
  final String message;
  MyFeatureError(this.message);
}

// presentation/blocs/my_feature/my_feature_bloc.dart
class MyFeatureBloc extends Bloc<MyFeatureEvent, MyFeatureState> {
  final MyFeatureUsecase _usecase = MyFeatureUsecase();

  MyFeatureBloc() : super(MyFeatureInitial()) {
    on<LoadMyFeatureItems>((event, emit) async {
      emit(MyFeatureLoading());
      final res = await _usecase.getItems();
      if (res is SuccessReponse) {
        final items = (res.object['data'] as List)
            .map((e) => ItemModel.fromJson(e))
            .toList();
        emit(MyFeatureLoaded(items));
      } else {
        emit(MyFeatureError(res.message ?? "general_error".tr()));
      }
    });
  }
}
```

### 4. UI Layer (Screens & Widgets)
- Use `ScreenUtil` responsive extensions: `.w`, `.h`, `.sp`, `.r`, `.horizontalSpace`, `.verticalSpace`.
- Use `easy_localization` `.tr()` for all user-facing strings.
- Provide `BlocProvider` or inject at screen level.

```dart
// presentation/screens/my_feature_screen.dart
class MyFeatureScreen extends StatelessWidget {
  const MyFeatureScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => MyFeatureBloc()..add(LoadMyFeatureItems()),
      child: Scaffold(
        appBar: AppBar(title: Text("my_feature_title".tr())),
        body: BlocBuilder<MyFeatureBloc, MyFeatureState>(
          builder: (context, state) {
            if (state is MyFeatureLoading) {
              return const Center(child: CircularProgressIndicator());
            } else if (state is MyFeatureLoaded) {
              return ListView.separated(
                padding: EdgeInsets.all(16.r),
                itemCount: state.items.length,
                separatorBuilder: (_, __) => 12.verticalSpace,
                itemBuilder: (context, index) {
                  final item = state.items[index];
                  return ListTile(
                    title: Text(item.name, style: TextStyle(fontSize: 14.sp)),
                  );
                },
              );
            } else if (state is MyFeatureError) {
              return Center(child: Text(state.message));
            }
            return const SizedBox.shrink();
          },
        ),
      ),
    );
  }
}
```

---

## Best Practices & Guidelines
1. **Never make API calls or business calculations inside UI Widgets.** Always delegate to BLoC and UseCases.
2. **Always close loading dialogs** (`closeCustomLoad()`) in both success and catch/error branches if `showCustomLoad()` was opened.
3. **Use ScreenUtil for all dimensions** to maintain consistent cross-device UI scaling.
4. **Localization is mandatory**: Never hardcode English or Arabic strings in UI files.
