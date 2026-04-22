## as-monaco-basket-flutter

> Use these templates to ensure consistent structure across all features.

# Feature Templates

## When Creating New Features
Use these templates to ensure consistent structure across all features.

## Feature Creation Checklist
1. Create feature folder: `lib/features/[feature_name]/`
2. Create layer subfolders
3. Generate template files
4. Update dependency injection
5. Add tests
6. Update routing

## Complete Feature Template Structure
```
lib/features/[feature_name]/
├── presentation/
│   ├── pages/
│   │   └── [feature_name]_page.dart
│   ├── widgets/
│   │   ├── [feature_name]_card.dart
│   │   └── [feature_name]_list_item.dart
│   └── bloc/
│       ├── [feature_name]_bloc.dart
│       ├── [feature_name]_event.dart
│       └── [feature_name]_state.dart
├── domain/
│   ├── entities/
│   │   └── [feature_name]_entity.dart
│   ├── repositories/
│   │   └── [feature_name]_repository.dart
│   └── usecases/
│       ├── get_[feature_name]_usecase.dart
│       ├── create_[feature_name]_usecase.dart
│       ├── update_[feature_name]_usecase.dart
│       └── delete_[feature_name]_usecase.dart
└── data/
    ├── models/
    │   └── [feature_name]_model.dart
    ├── repositories/
    │   └── [feature_name]_repository_impl.dart
    └── datasources/
        ├── [feature_name]_remote_datasource.dart
        └── [feature_name]_local_datasource.dart
```

## Basketball-Specific Feature Examples

### Players Feature
```
lib/features/players/
├── presentation/
│   ├── pages/
│   │   ├── players_page.dart
│   │   └── player_detail_page.dart
│   ├── widgets/
│   │   ├── player_card.dart
│   │   ├── player_stats_widget.dart
│   │   └── players_grid.dart
│   └── bloc/
│       ├── players_bloc.dart
│       ├── players_event.dart
│       └── players_state.dart
├── domain/
│   ├── entities/
│   │   ├── player_entity.dart
│   │   └── player_stats_entity.dart
│   ├── repositories/
│   │   └── players_repository.dart
│   └── usecases/
│       ├── get_players_usecase.dart
│       ├── get_player_details_usecase.dart
│       └── get_player_stats_usecase.dart
└── data/
    ├── models/
    │   ├── player_model.dart
    │   └── player_stats_model.dart
    ├── repositories/
    │   └── players_repository_impl.dart
    └── datasources/
        ├── players_remote_datasource.dart
        └── players_local_datasource.dart
```

### Games Feature
```
lib/features/games/
├── presentation/
│   ├── pages/
│   │   ├── games_page.dart
│   │   └── game_detail_page.dart
│   ├── widgets/
│   │   ├── game_card.dart
│   │   ├── score_widget.dart
│   │   └── game_timeline.dart
│   └── bloc/
│       ├── games_bloc.dart
│       ├── games_event.dart
│       └── games_state.dart
├── domain/
│   ├── entities/
│   │   ├── game_entity.dart
│   │   └── score_entity.dart
│   ├── repositories/
│   │   └── games_repository.dart
│   └── usecases/
│       ├── get_games_usecase.dart
│       ├── get_game_details_usecase.dart
│       └── get_live_score_usecase.dart
└── data/
    ├── models/
    │   ├── game_model.dart
    │   └── score_model.dart
    ├── repositories/
    │   └── games_repository_impl.dart
    └── datasources/
        ├── games_remote_datasource.dart
        └── games_local_datasource.dart
```

## Quick Commands for Feature Creation

### 1. Entity Template
```dart
// lib/features/[feature]/domain/entities/[feature]_entity.dart
import 'package:equatable/equatable.dart';

class FeatureNameEntity extends Equatable {
  final int id;
  final String name;
  final DateTime createdAt;
  final DateTime updatedAt;

  const FeatureNameEntity({
    required this.id,
    required this.name,
    required this.createdAt,
    required this.updatedAt,
  });

  @override
  List<Object?> get props => [id, name, createdAt, updatedAt];
}
```

### 2. Repository Interface Template
```dart
// lib/features/[feature]/domain/repositories/[feature]_repository.dart
import '../../core/resources/data_state.dart';
import '../entities/[feature]_entity.dart';

abstract class FeatureNameRepository {
  Future<DataState<List<FeatureNameEntity>>> getFeatures();
  Future<DataState<FeatureNameEntity>> getFeatureById(int id);
  Future<DataState<void>> createFeature(FeatureNameEntity feature);
  Future<DataState<void>> updateFeature(FeatureNameEntity feature);
  Future<DataState<void>> deleteFeature(int id);
}
```

### 3. Use Case Template
```dart
// lib/features/[feature]/domain/usecases/get_[feature]_usecase.dart
import '../../core/resources/data_state.dart';
import '../entities/[feature]_entity.dart';
import '../repositories/[feature]_repository.dart';

class GetFeatureNameUseCase {
  final FeatureNameRepository repository;

  const GetFeatureNameUseCase(this.repository);

  Future<DataState<List<FeatureNameEntity>>> call() async {
    return await repository.getFeatures();
  }
}
```

### 4. BLoC Template
```dart
// lib/features/[feature]/presentation/bloc/[feature]_bloc.dart
import 'package:bloc/bloc.dart';
import 'package:equatable/equatable.dart';
import '../../domain/entities/[feature]_entity.dart';
import '../../domain/usecases/get_[feature]_usecase.dart';

part '[feature]_event.dart';
part '[feature]_state.dart';

class FeatureNameBloc extends Bloc<FeatureNameEvent, FeatureNameState> {
  final GetFeatureNameUseCase getFeatureNameUseCase;

  FeatureNameBloc(this.getFeatureNameUseCase) : super(FeatureNameInitial()) {
    on<GetFeatureNameEvent>(_onGetFeatureName);
  }

  Future<void> _onGetFeatureName(
    GetFeatureNameEvent event,
    Emitter<FeatureNameState> emit,
  ) async {
    emit(FeatureNameLoading());
    final result = await getFeatureNameUseCase();

    if (result.isSuccess) {
      emit(FeatureNameSuccess(result.data!));
    } else {
      emit(FeatureNameError(result.error!));
    }
  }
}
```

## File Generation Commands
Use these patterns to quickly generate files:
- Replace `[feature]` with actual feature name
- Replace `FeatureName` with PascalCase version
- Replace `feature_name` with snake_case version
- Update imports accordingly
- Add to dependency injection container

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pabStudi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:gemini_md:2026-04-09 -->
