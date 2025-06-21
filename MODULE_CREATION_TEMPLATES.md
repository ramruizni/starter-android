# Module Creation Templates Guide

This guide provides ready-to-use templates for creating different types of modules in your Android project, ensuring consistency and following established architecture patterns.

## Overview

Module templates provide:
- **Consistent Structure**: Standardized directory layouts
- **Build Configuration**: Pre-configured `build.gradle.kts` files
- **Code Templates**: Basic implementations following architecture patterns
- **Dependency Management**: Proper module relationships
- **Testing Setup**: Complete test configurations

## Module Types

### 1. Feature View Module
### 2. Feature ViewModel Module  
### 3. Domain Models Module
### 4. Domain Use Cases Module
### 5. Data Layer Module
### 6. UI Component Module
### 7. Core Library Module

## 1. Feature View Module Template

### Directory Structure
```
features/[feature-name]/view/
├── src/
│   ├── main/
│   │   ├── java/com/myproject/features/[feature]/view/
│   │   │   ├── [Feature]Screen.kt
│   │   │   ├── I[Feature]Navigator.kt
│   │   │   └── components/
│   │   │       └── [Feature]Content.kt
│   │   └── AndroidManifest.xml
│   └── test/
│       └── java/com/myproject/features/[feature]/view/
│           └── [Feature]ScreenTest.kt
└── build.gradle.kts
```

### Build Configuration
```kotlin
// features/[feature-name]/view/build.gradle.kts
plugins {
    alias(libs.plugins.project.arch.view)
}

dependencies {
    implementation(project(":features:[feature-name]:viewmodel"))
    implementation(project(":domain:[feature-name]:models"))
    implementation(project(":core:navigation"))
    
    // Add feature-specific dependencies here
    // implementation(libs.coil) // for image loading
    // implementation(libs.androidx.constraintlayout.compose.android) // for complex layouts
}
```

### Screen Template
```kotlin
// features/[feature-name]/view/src/main/java/com/myproject/features/[feature]/view/[Feature]Screen.kt
package com.myproject.features.[feature].view

import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Scaffold
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.getValue
import androidx.compose.ui.Modifier
import androidx.compose.ui.platform.LocalContext
import androidx.hilt.navigation.compose.hiltViewModel
import androidx.lifecycle.compose.collectAsStateWithLifecycle
import com.myproject.features.[feature].view.components.[Feature]Content
import com.myproject.features.[feature].viewmodel.[Feature]ViewModel

@Composable
fun [Feature]Screen(
    navigator: I[Feature]Navigator,
    viewModel: [Feature]ViewModel = hiltViewModel(),
) {
    val context = LocalContext.current
    val state by viewModel.state.collectAsStateWithLifecycle()

    // Event handling
    LaunchedEffect(context) {
        viewModel.events.collect { event ->
            when (event) {
                is [Feature]ViewModel.Event.NavigateBack -> {
                    navigator.navigateUp()
                }
                is [Feature]ViewModel.Event.NavigateToDetail -> {
                    navigator.navigateToDetail(event.itemId)
                }
                is [Feature]ViewModel.Event.ShowError -> {
                    // Handle error display
                }
            }
        }
    }

    Scaffold(
        modifier = Modifier.fillMaxSize()
    ) { innerPadding ->
        [Feature]Content(
            modifier = Modifier.padding(innerPadding),
            state = state,
            onAction = viewModel::handleAction
        )
    }
}
```

### Navigator Interface Template
```kotlin
// features/[feature-name]/view/src/main/java/com/myproject/features/[feature]/view/I[Feature]Navigator.kt
package com.myproject.features.[feature].view

interface I[Feature]Navigator {
    fun navigateUp()
    fun navigateToDetail(itemId: String)
    fun navigateToSettings()
}
```

### Content Component Template
```kotlin
// features/[feature-name]/view/src/main/java/com/myproject/features/[feature]/view/components/[Feature]Content.kt
package com.myproject.features.[feature].view.components

import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import com.myproject.features.[feature].viewmodel.[Feature]ViewModel

@Composable
fun [Feature]Content(
    state: [Feature]ViewModel.State,
    onAction: ([Feature]ViewModel.Action) -> Unit,
    modifier: Modifier = Modifier,
) {
    Column(
        modifier = modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        Text(
            text = "[Feature] Screen",
            style = MaterialTheme.typography.headlineMedium
        )
        
        // Add your UI components here based on state
        when {
            state.isLoading -> {
                // Loading UI
            }
            state.hasError -> {
                // Error UI
            }
            else -> {
                // Content UI
            }
        }
    }
}
```

## 2. Feature ViewModel Module Template

### Directory Structure
```
features/[feature-name]/viewmodel/
├── src/
│   ├── main/
│   │   └── java/com/myproject/features/[feature]/viewmodel/
│   │       └── [Feature]ViewModel.kt
│   └── test/
│       └── java/com/myproject/features/[feature]/viewmodel/
│           └── [Feature]ViewModelTest.kt
└── build.gradle.kts
```

### Build Configuration
```kotlin
// features/[feature-name]/viewmodel/build.gradle.kts
plugins {
    alias(libs.plugins.project.arch.viewmodel)
}

dependencies {
    implementation(project(":domain:[feature-name]:usecases"))
    implementation(project(":domain:[feature-name]:models"))
    
    // Testing
    testImplementation(libs.kotlinx.coroutines.test)
    testImplementation(libs.junit5.api)
}
```

### ViewModel Template
```kotlin
// features/[feature-name]/viewmodel/src/main/java/com/myproject/features/[feature]/viewmodel/[Feature]ViewModel.kt
package com.myproject.features.[feature].viewmodel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.flow.MutableSharedFlow
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.asSharedFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.update
import kotlinx.coroutines.launch
import javax.inject.Inject

@HiltViewModel
class [Feature]ViewModel @Inject constructor(
    private val get[Feature]DataUseCase: Get[Feature]DataUseCase,
    private val update[Feature]UseCase: Update[Feature]UseCase,
) : ViewModel() {

    private val _state = MutableStateFlow(State())
    val state = _state.asStateFlow()

    private val _events = MutableSharedFlow<Event>()
    val events = _events.asSharedFlow()

    init {
        loadData()
    }

    fun handleAction(action: Action) {
        when (action) {
            is Action.LoadData -> loadData()
            is Action.UpdateItem -> updateItem(action.itemId, action.data)
            is Action.NavigateToDetail -> navigateToDetail(action.itemId)
            is Action.Refresh -> refresh()
        }
    }

    private fun loadData() {
        viewModelScope.launch(Dispatchers.IO) {
            _state.update { it.copy(isLoading = true) }
            
            try {
                val data = get[Feature]DataUseCase()
                _state.update { 
                    it.copy(
                        data = data,
                        isLoading = false,
                        hasError = false
                    )
                }
            } catch (error: Exception) {
                handleError(error)
            }
        }
    }

    private fun updateItem(itemId: String, data: String) {
        viewModelScope.launch(Dispatchers.IO) {
            try {
                update[Feature]UseCase(itemId, data)
                loadData() // Refresh data
            } catch (error: Exception) {
                handleError(error)
            }
        }
    }

    private fun navigateToDetail(itemId: String) {
        viewModelScope.launch {
            _events.emit(Event.NavigateToDetail(itemId))
        }
    }

    private fun refresh() {
        loadData()
    }

    private fun handleError(error: Exception) {
        _state.update { 
            it.copy(
                isLoading = false,
                hasError = true,
                errorMessage = error.message
            ) 
        }
        
        viewModelScope.launch {
            _events.emit(Event.ShowError(error.message ?: "Unknown error"))
        }
    }

    data class State(
        val data: List<[Feature]Item> = emptyList(),
        val isLoading: Boolean = false,
        val hasError: Boolean = false,
        val errorMessage: String? = null,
    )

    sealed interface Action {
        data object LoadData : Action
        data class UpdateItem(val itemId: String, val data: String) : Action
        data class NavigateToDetail(val itemId: String) : Action
        data object Refresh : Action
    }

    sealed interface Event {
        data class NavigateToDetail(val itemId: String) : Event
        data object NavigateBack : Event
        data class ShowError(val message: String) : Event
    }
}
```

## 3. Domain Models Module Template

### Directory Structure
```
domain/[feature-name]/models/
├── src/
│   ├── main/
│   │   └── java/com/myproject/domain/[feature]/models/
│   │       ├── [Feature]Item.kt
│   │       ├── [Feature]Request.kt
│   │       └── [Feature]Response.kt
│   └── test/
│       └── java/com/myproject/domain/[feature]/models/
│           └── [Feature]ItemTest.kt
└── build.gradle.kts
```

### Build Configuration
```kotlin
// domain/[feature-name]/models/build.gradle.kts
plugins {
    alias(libs.plugins.project.jvm.library)
}

dependencies {
    implementation(libs.kotlinx.serialization.json)
    implementation(libs.kotlinx.collections.immutable)
    
    // Testing
    testImplementation(libs.junit5.api)
}
```

### Model Templates
```kotlin
// domain/[feature-name]/models/src/main/java/com/myproject/domain/[feature]/models/[Feature]Item.kt
package com.myproject.domain.[feature].models

import kotlinx.serialization.Serializable

@Serializable
data class [Feature]Item(
    val id: String,
    val title: String,
    val description: String,
    val createdAt: Long,
    val updatedAt: Long,
    val isActive: Boolean = true,
)

@Serializable
data class [Feature]Request(
    val title: String,
    val description: String,
)

@Serializable
data class [Feature]Response(
    val items: List<[Feature]Item>,
    val totalCount: Int,
    val hasMore: Boolean,
)
```

## 4. Domain Use Cases Module Template

### Directory Structure
```
domain/[feature-name]/usecases/
├── src/
│   ├── main/
│   │   └── java/com/myproject/domain/[feature]/usecases/
│   │       ├── Get[Feature]DataUseCase.kt
│   │       ├── Update[Feature]UseCase.kt
│   │       └── Delete[Feature]UseCase.kt
│   └── test/
│       └── java/com/myproject/domain/[feature]/usecases/
│           └── Get[Feature]DataUseCaseTest.kt
└── build.gradle.kts
```

### Build Configuration
```kotlin
// domain/[feature-name]/usecases/build.gradle.kts
plugins {
    alias(libs.plugins.project.jvm.library)
    alias(libs.plugins.project.android.hilt)
}

dependencies {
    implementation(project(":domain:[feature-name]:models"))
    implementation(project(":data:[feature-name]"))
    
    implementation(libs.kotlinx.coroutines)
    
    // Testing
    testImplementation(libs.kotlinx.coroutines.test)
    testImplementation(libs.junit5.api)
}
```

### Use Case Templates
```kotlin
// domain/[feature-name]/usecases/src/main/java/com/myproject/domain/[feature]/usecases/Get[Feature]DataUseCase.kt
package com.myproject.domain.[feature].usecases

import com.myproject.domain.[feature].models.[Feature]Item
import com.myproject.data.[feature].[Feature]Repository
import javax.inject.Inject

class Get[Feature]DataUseCase @Inject constructor(
    private val repository: [Feature]Repository
) {
    suspend operator fun invoke(
        userId: String? = null,
        limit: Int = 20,
        offset: Int = 0
    ): List<[Feature]Item> {
        return repository.get[Feature]Items(
            userId = userId,
            limit = limit,
            offset = offset
        )
    }
}

class Update[Feature]UseCase @Inject constructor(
    private val repository: [Feature]Repository
) {
    suspend operator fun invoke(itemId: String, title: String, description: String): [Feature]Item {
        return repository.update[Feature]Item(
            itemId = itemId,
            title = title,
            description = description
        )
    }
}

class Delete[Feature]UseCase @Inject constructor(
    private val repository: [Feature]Repository
) {
    suspend operator fun invoke(itemId: String) {
        repository.delete[Feature]Item(itemId)
    }
}
```

## 5. Data Layer Module Template

### Directory Structure
```
data/[feature-name]/
├── src/
│   ├── main/
│   │   └── java/com/myproject/data/[feature]/
│   │       ├── [Feature]Repository.kt
│   │       ├── [Feature]DataSource.kt
│   │       ├── local/
│   │       │   ├── [Feature]Dao.kt
│   │       │   └── [Feature]Entity.kt
│   │       ├── remote/
│   │       │   ├── [Feature]ApiService.kt
│   │       │   └── [Feature]Dto.kt
│   │       └── di/
│   │           └── [Feature]DataModule.kt
│   └── test/
│       └── java/com/myproject/data/[feature]/
│           └── [Feature]RepositoryTest.kt
└── build.gradle.kts
```

### Build Configuration
```kotlin
// data/[feature-name]/build.gradle.kts
plugins {
    alias(libs.plugins.project.android.library)
    alias(libs.plugins.project.android.hilt)
    alias(libs.plugins.project.android.room)
}

dependencies {
    implementation(project(":domain:[feature-name]:models"))
    implementation(project(":core:database"))
    implementation(project(":core:network"))
    
    implementation(libs.okhttp.core)
    implementation(libs.gson)
    
    // Testing
    testImplementation(libs.kotlinx.coroutines.test)
    testImplementation(libs.junit5.api)
}
```

### Repository Template
```kotlin
// data/[feature-name]/src/main/java/com/myproject/data/[feature]/[Feature]Repository.kt
package com.myproject.data.[feature]

import com.myproject.data.[feature].local.[Feature]Dao
import com.myproject.data.[feature].remote.[Feature]ApiService
import com.myproject.domain.[feature].models.[Feature]Item
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class [Feature]Repository @Inject constructor(
    private val apiService: [Feature]ApiService,
    private val dao: [Feature]Dao,
) {
    
    suspend fun get[Feature]Items(
        userId: String? = null,
        limit: Int = 20,
        offset: Int = 0
    ): List<[Feature]Item> {
        return try {
            // Try remote first
            val remoteItems = apiService.get[Feature]Items(userId, limit, offset)
            
            // Cache locally
            dao.insertAll(remoteItems.map { it.toEntity() })
            
            remoteItems.map { it.toDomain() }
        } catch (e: Exception) {
            // Fallback to local cache
            dao.getAll(limit, offset).map { it.toDomain() }
        }
    }
    
    fun observe[Feature]Items(): Flow<List<[Feature]Item>> {
        return dao.observeAll().map { entities ->
            entities.map { it.toDomain() }
        }
    }
    
    suspend fun update[Feature]Item(
        itemId: String,
        title: String,
        description: String
    ): [Feature]Item {
        val updatedItem = apiService.update[Feature]Item(itemId, title, description)
        dao.insert(updatedItem.toEntity())
        return updatedItem.toDomain()
    }
    
    suspend fun delete[Feature]Item(itemId: String) {
        apiService.delete[Feature]Item(itemId)
        dao.deleteById(itemId)
    }
}
```

## 6. UI Component Module Template

### Directory Structure
```
ui/[component-name]/
├── src/
│   ├── main/
│   │   └── java/com/myproject/ui/[component]/
│   │       ├── [Component].kt
│   │       └── [Component]Preview.kt
│   └── test/
│       └── java/com/myproject/ui/[component]/
│           └── [Component]Test.kt
└── build.gradle.kts
```

### Build Configuration
```kotlin
// ui/[component-name]/build.gradle.kts
plugins {
    alias(libs.plugins.project.android.library)
    alias(libs.plugins.project.android.library.compose)
}

dependencies {
    implementation(project(":ui:design-system"))
    
    implementation(libs.androidx.ui.tooling.preview)
    debugImplementation(libs.androidx.ui.tooling)
}
```

## 7. Quick Module Creation Scripts

### Create Feature Module Script
```bash
#!/bin/bash
# create-feature.sh
FEATURE_NAME=$1
PACKAGE_PREFIX="com.myproject"

if [ -z "$FEATURE_NAME" ]; then
    echo "Usage: ./create-feature.sh <feature-name>"
    exit 1
fi

# Create directory structure
mkdir -p "features/$FEATURE_NAME"/{view,viewmodel}
mkdir -p "domain/$FEATURE_NAME"/{models,usecases}
mkdir -p "data/$FEATURE_NAME"

# Create view module
mkdir -p "features/$FEATURE_NAME/view/src/main/java/${PACKAGE_PREFIX//.//}/features/$FEATURE_NAME/view"
mkdir -p "features/$FEATURE_NAME/view/src/test/java/${PACKAGE_PREFIX//.//}/features/$FEATURE_NAME/view"

# Create build files from templates
cp templates/feature-view-build.gradle.kts "features/$FEATURE_NAME/view/build.gradle.kts"
cp templates/feature-viewmodel-build.gradle.kts "features/$FEATURE_NAME/viewmodel/build.gradle.kts"
cp templates/domain-models-build.gradle.kts "domain/$FEATURE_NAME/models/build.gradle.kts"
cp templates/domain-usecases-build.gradle.kts "domain/$FEATURE_NAME/usecases/build.gradle.kts"
cp templates/data-build.gradle.kts "data/$FEATURE_NAME/build.gradle.kts"

# Replace placeholders
find "features/$FEATURE_NAME" "domain/$FEATURE_NAME" "data/$FEATURE_NAME" -name "*.kts" -exec sed -i "s/\[feature-name\]/$FEATURE_NAME/g" {} \;

# Update settings.gradle.kts
echo "include(\":features:$FEATURE_NAME:view\")" >> settings.gradle.kts
echo "include(\":features:$FEATURE_NAME:viewmodel\")" >> settings.gradle.kts
echo "include(\":domain:$FEATURE_NAME:models\")" >> settings.gradle.kts
echo "include(\":domain:$FEATURE_NAME:usecases\")" >> settings.gradle.kts
echo "include(\":data:$FEATURE_NAME\")" >> settings.gradle.kts

echo "Feature $FEATURE_NAME created successfully!"
```

## 8. Module Template Checklist

When creating new modules, ensure:

### ✅ Structure
- [ ] Correct directory structure
- [ ] Proper package naming
- [ ] Manifest files where needed

### ✅ Build Configuration
- [ ] Convention plugins applied
- [ ] Dependencies configured
- [ ] Testing setup included

### ✅ Code Templates
- [ ] Basic implementations provided
- [ ] Architecture patterns followed
- [ ] Error handling included

### ✅ Integration
- [ ] Added to settings.gradle.kts
- [ ] Dependencies properly linked
- [ ] Navigation updated if needed

This comprehensive template system ensures consistent module creation while following established architecture patterns and build configurations.