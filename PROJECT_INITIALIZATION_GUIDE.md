# Android Project Initialization Guide

This guide provides a complete setup for creating new Android projects with standardized convention plugins, gradle configuration, and modular architecture based on the Dazzle project analysis.

## Overview

This initialization system provides:
- **Convention Plugins**: Standardized build configurations for different module types
- **Version Catalog**: Centralized dependency management
- **Modular Architecture**: Scalable project structure
- **Company Agnostic**: No dependency on specific shared libraries
- **Module Templates**: Ready-to-use module creation patterns

## 1. Initial Project Structure

### Root Project Setup
```
my-project/
├── app/                              # Main application module
├── build-logic/                      # Convention plugins
│   └── convention/
│       ├── build.gradle.kts
│       ├── settings.gradle.kts
│       └── src/main/java/
├── features/                         # Feature modules
├── core/                            # Core modules
├── domain/                          # Domain layer modules
├── data/                            # Data layer modules
├── ui/                              # UI modules
├── build.gradle.kts                 # Root build file
├── settings.gradle.kts              # Project settings
├── gradle.properties               # Gradle properties
└── gradle/
    └── libs.versions.toml           # Version catalog
```

## 2. Version Catalog Setup

### Create `gradle/libs.versions.toml`
```toml
[versions]
# Android
agp = "8.9.3"
compileSdk = "35"
minSdk = "25"
targetSdk = "35"

# Kotlin
kotlin = "2.0.20"
kotlinxSerializationPlugin = "2.1.21"
kotlinxCoroutines = "1.9.0"
kotlinxSerializationJson = "1.8.1"
kotlinxCollectionsImmutable = "0.3.6"
ksp = "2.1.21-2.0.1"

# AndroidX
coreKtx = "1.16.0"
lifecycleRuntimeKtx = "2.9.0"
lifecycleViewmodelKtx = "2.9.0"
activityCompose = "1.10.1"
androidxNavigation = "2.9.0"
androidxHiltNavigationCompose = "1.2.0"

# Compose
composeBom = "2025.05.01"

# UI
material3 = "1.3.1"
constraintlayoutComposeAndroid = "1.1.1"
coil = "2.6.0"

# Testing
junit5 = "5.10.1"
junitVersion = "1.2.1"
espressoCore = "3.6.1"

# Dependency Injection
hilt = "2.56.2"

# Database
room = "2.7.1"

# Network
okhttp = "4.12.0"
gson = "2.10.1"

# Other
androidDesugarJdkLibs = "2.1.5"
coreSplashscreen = "1.0.1"

[libraries]
# Android Core
androidx-core-ktx = { group = "androidx.core", name = "core-ktx", version.ref = "coreKtx" }
androidx-lifecycle-runtime-ktx = { group = "androidx.lifecycle", name = "lifecycle-runtime-ktx", version.ref = "lifecycleRuntimeKtx" }
androidx-lifecycle-viewmodel-ktx = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-ktx", version.ref = "lifecycleViewmodelKtx" }
androidx-lifecycle-viewmodel-compose = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-compose" }
androidx-activity-compose = { group = "androidx.activity", name = "activity-compose", version.ref = "activityCompose" }
androidx-core-splashscreen = { module = "androidx.core:core-splashscreen", version.ref = "coreSplashscreen" }

# Compose BOM
androidx-compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "composeBom" }
androidx-ui = { group = "androidx.compose.ui", name = "ui" }
androidx-ui-graphics = { group = "androidx.compose.ui", name = "ui-graphics" }
androidx-ui-tooling = { group = "androidx.compose.ui", name = "ui-tooling" }
androidx-ui-tooling-preview = { group = "androidx.compose.ui", name = "ui-tooling-preview" }
androidx-ui-test-manifest = { group = "androidx.compose.ui", name = "ui-test-manifest" }
androidx-ui-test-junit4 = { group = "androidx.compose.ui", name = "ui-test-junit4" }
androidx-material3 = { group = "androidx.compose.material3", name = "material3" }
androidx-constraintlayout-compose-android = { group = "androidx.constraintlayout", name = "constraintlayout-compose-android", version.ref = "constraintlayoutComposeAndroid" }

# Navigation
androidx-navigation-compose = { group = "androidx.navigation", name = "navigation-compose", version.ref = "androidxNavigation" }
androidx-hilt-navigation-compose = { group = "androidx.hilt", name = "hilt-navigation-compose", version.ref = "androidxHiltNavigationCompose" }

# Dependency Injection
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-android-testing = { group = "com.google.dagger", name = "hilt-android-testing", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-android-compiler", version.ref = "hilt" }

# Database
room-common = { group = "androidx.room", name = "room-common", version.ref = "room" }
room-compiler = { group = "androidx.room", name = "room-compiler", version.ref = "room" }
room-gradle-plugin = { group = "androidx.room", name = "room-gradle-plugin", version.ref = "room" }
room-ktx = { group = "androidx.room", name = "room-ktx", version.ref = "room" }
room-runtime = { group = "androidx.room", name = "room-runtime", version.ref = "room" }

# Kotlin
kotlinx-coroutines = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-core", version.ref = "kotlinxCoroutines" }
kotlinx-coroutines-test = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-test", version.ref = "kotlinxCoroutines" }
kotlinx-serialization-json = { module = "org.jetbrains.kotlinx:kotlinx-serialization-json", version.ref = "kotlinxSerializationJson" }
kotlinx-collections-immutable = { module = "org.jetbrains.kotlinx:kotlinx-collections-immutable", version.ref = "kotlinxCollectionsImmutable" }

# Network
okhttp-core = { group = "com.squareup.okhttp3", name = "okhttp", version.ref = "okhttp" }
gson = { group = "com.google.code.gson", name = "gson", version.ref = "gson" }

# Image Loading
coil = { group = "io.coil-kt", name = "coil-compose", version.ref = "coil" }

# Testing
junit5-api = { group = "org.junit.jupiter", name = "junit-jupiter-api", version.ref = "junit5" }
junit5-engine = { group = "org.junit.jupiter", name = "junit-jupiter-engine", version.ref = "junit5" }
androidx-junit = { group = "androidx.test.ext", name = "junit", version.ref = "junitVersion" }
androidx-espresso-core = { group = "androidx.test.espresso", name = "espresso-core", version.ref = "espressoCore" }

# Other
android-desugar-jdk-libs = { group = "com.android.tools", name = "desugar_jdk_libs", version.ref = "androidDesugarJdkLibs" }

# Build plugins
android-gradle-plugin = { group = "com.android.tools.build", name = "gradle", version.ref = "agp" }
kotlin-gradle-plugin = { group = "org.jetbrains.kotlin", name = "kotlin-gradle-plugin", version.ref = "kotlin" }
ksp-gradle-plugin = { group = "com.google.devtools.ksp", name = "com.google.devtools.ksp.gradle.plugin", version.ref = "ksp" }

[plugins]
# Android
android-application = { id = "com.android.application", version.ref = "agp" }
android-library = { id = "com.android.library", version.ref = "agp" }

# Kotlin
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kotlin-compose = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
kotlin-jvm = { id = "org.jetbrains.kotlin.jvm", version.ref = "kotlin" }
kotlinx-serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlinxSerializationPlugin" }

# Tools
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
room = { id = "androidx.room", version.ref = "room" }

# Convention Plugins
project-android-application = { id = "project.android.application", version = "unspecified" }
project-android-application-compose = { id = "project.android.application.compose", version = "unspecified" }
project-android-library = { id = "project.android.library", version = "unspecified" }
project-android-library-compose = { id = "project.android.library.compose", version = "unspecified" }
project-android-hilt = { id = "project.android.hilt", version = "unspecified" }
project-android-room = { id = "project.android.room", version = "unspecified" }
project-jvm-library = { id = "project.jvm.library", version = "unspecified" }

# Architecture plugins
project-arch-view = { id = "project.arch.view", version = "unspecified" }
project-arch-viewmodel = { id = "project.arch.viewmodel", version = "unspecified" }

[bundles]
# Android Core UI
android-ui = [
    "androidx-core-ktx",
    "androidx-lifecycle-runtime-ktx",
    "androidx-activity-compose",
    "androidx-material3",
    "androidx-ui-tooling-preview",
]

# Compose
androidx-compose = [
    "androidx-ui",
    "androidx-ui-graphics",
    "androidx-navigation-compose",
    "androidx-hilt-navigation-compose",
    "androidx-lifecycle-viewmodel-compose"
]

# Room Database
room-datasource = [
    "room-ktx",
    "room-runtime"
]

# Testing
junit5 = [
    "junit5-api",
    "junit5-engine"
]
```

## 3. Root Build Configuration

### Create `settings.gradle.kts`
```kotlin
pluginManagement {
    includeBuild("build-logic")
    repositories {
        google {
            content {
                includeGroupByRegex("com\\.android.*")
                includeGroupByRegex("com\\.google.*")
                includeGroupByRegex("androidx.*")
            }
        }
        mavenCentral()
        gradlePluginPortal()
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}

rootProject.name = "MyProject"

// Core modules
include(":app")

// Example modules (add as needed)
include(":core:domain")
include(":core:data")
include(":core:ui")
include(":core:navigation")

include(":features:home:view")
include(":features:home:viewmodel")
include(":features:profile:view")
include(":features:profile:viewmodel")

include(":ui:design-system")
include(":ui:components")
```

### Create root `build.gradle.kts`
```kotlin
// Top-level build file where you can add configuration options common to all sub-projects/modules.
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.android.library) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.compose) apply false
    alias(libs.plugins.kotlin.jvm) apply false
    alias(libs.plugins.kotlinx.serialization) apply false
    alias(libs.plugins.ksp) apply false
    alias(libs.plugins.hilt) apply false
    alias(libs.plugins.room) apply false
}
```

### Create `gradle.properties`
```properties
# Gradle
org.gradle.jvmargs=-Xmx2048m -Dfile.encoding=UTF-8
org.gradle.caching=true
org.gradle.parallel=true

# Kotlin
kotlin.code.style=official

# Android
android.useAndroidX=true
android.nonTransitiveRClass=true

# Compose
enableComposeCompilerMetrics=false
enableComposeCompilerReports=false
```

## 4. Convention Plugins Setup

### Create `build-logic/settings.gradle.kts`
```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
    }
    versionCatalogs {
        create("libs") {
            from(files("../gradle/libs.versions.toml"))
        }
    }
}

rootProject.name = "build-logic"
include(":convention")
```

### Create `build-logic/convention/build.gradle.kts`
```kotlin
import org.jetbrains.kotlin.gradle.dsl.JvmTarget

plugins {
    `kotlin-dsl`
}

group = "com.myproject.convention"

// Configure the build-logic plugins to target JDK 17
java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
}

tasks.withType<org.jetbrains.kotlin.gradle.tasks.KotlinCompile>().configureEach {
    compilerOptions {
        jvmTarget.set(JvmTarget.JVM_17)
    }
}

dependencies {
    compileOnly(libs.android.gradle.plugin)
    compileOnly(libs.kotlin.gradle.plugin)
    compileOnly(libs.room.gradle.plugin)
}

gradlePlugin {
    plugins {
        register("android-application") {
            id = "project.android.application"
            implementationClass = "AndroidApplicationConventionPlugin"
        }
        
        register("android-application-compose") {
            id = "project.android.application.compose"
            implementationClass = "AndroidApplicationComposeConventionPlugin"
        }
        
        register("android-library") {
            id = "project.android.library"
            implementationClass = "AndroidLibraryConventionPlugin"
        }
        
        register("android-library-compose") {
            id = "project.android.library.compose"
            implementationClass = "AndroidLibraryComposeConventionPlugin"
        }
        
        register("android-hilt") {
            id = "project.android.hilt"
            implementationClass = "AndroidHiltConventionPlugin"
        }
        
        register("android-room") {
            id = "project.android.room"
            implementationClass = "AndroidRoomConventionPlugin"
        }
        
        register("jvm-library") {
            id = "project.jvm.library"
            implementationClass = "JvmLibraryConventionPlugin"
        }
        
        // Architecture plugins
        register("arch-view") {
            id = "project.arch.view"
            implementationClass = "ArchViewConventionPlugin"
        }
        
        register("arch-viewmodel") {
            id = "project.arch.viewmodel"
            implementationClass = "ArchViewModelConventionPlugin"
        }
    }
}
```

## 5. Creating a New Project

### Quick Start Script
Create a script `create-project.sh` for rapid project initialization:

```bash
#!/bin/bash

# Create new Android project
PROJECT_NAME=$1
PACKAGE_NAME=$2

if [ -z "$PROJECT_NAME" ] || [ -z "$PACKAGE_NAME" ]; then
    echo "Usage: ./create-project.sh <project-name> <package-name>"
    echo "Example: ./create-project.sh MyApp com.mycompany.myapp"
    exit 1
fi

# Create directory structure
mkdir -p "$PROJECT_NAME"/{app,build-logic/convention,core,features,ui,data,domain}
mkdir -p "$PROJECT_NAME"/gradle

# Copy template files
cp -r template/gradle/libs.versions.toml "$PROJECT_NAME"/gradle/
cp template/settings.gradle.kts "$PROJECT_NAME"/
cp template/build.gradle.kts "$PROJECT_NAME"/
cp template/gradle.properties "$PROJECT_NAME"/

# Copy build-logic
cp -r template/build-logic/* "$PROJECT_NAME"/build-logic/

# Update package names
find "$PROJECT_NAME" -name "*.kt" -o -name "*.kts" -o -name "*.xml" | xargs sed -i "s/com\.myproject/$PACKAGE_NAME/g"
find "$PROJECT_NAME" -name "*.kt" -o -name "*.kts" -o -name "*.xml" | xargs sed -i "s/MyProject/$PROJECT_NAME/g"

echo "Project $PROJECT_NAME created successfully!"
echo "Next steps:"
echo "1. cd $PROJECT_NAME"
echo "2. ./gradlew build"
echo "3. Open in Android Studio"
```

## 6. Adding New Modules

### Feature Module Creation
```bash
# Create feature module structure
mkdir -p features/newfeature/{view,viewmodel}
mkdir -p features/newfeature/view/src/main/java/com/myproject/features/newfeature/view
mkdir -p features/newfeature/viewmodel/src/main/java/com/myproject/features/newfeature/viewmodel

# Add to settings.gradle.kts
echo 'include(":features:newfeature:view")' >> settings.gradle.kts
echo 'include(":features:newfeature:viewmodel")' >> settings.gradle.kts
```

### Domain Module Creation
```bash
# Create domain module
mkdir -p domain/newdomain/{models,usecases}
mkdir -p domain/newdomain/models/src/main/java/com/myproject/domain/newdomain/models
mkdir -p domain/newdomain/usecases/src/main/java/com/myproject/domain/newdomain/usecases

# Add to settings.gradle.kts
echo 'include(":domain:newdomain:models")' >> settings.gradle.kts
echo 'include(":domain:newdomain:usecases")' >> settings.gradle.kts
```

## 7. Module Templates

### Feature View Module `build.gradle.kts`
```kotlin
plugins {
    alias(libs.plugins.project.arch.view)
}

dependencies {
    implementation(project(":features:featurename:viewmodel"))
    implementation(project(":core:navigation"))
    implementation(project(":ui:design-system"))
}
```

### Feature ViewModel Module `build.gradle.kts`
```kotlin
plugins {
    alias(libs.plugins.project.arch.viewmodel)
}

dependencies {
    implementation(project(":domain:featurename:usecases"))
    implementation(project(":core:domain"))
}
```

### Domain Models Module `build.gradle.kts`
```kotlin
plugins {
    alias(libs.plugins.project.jvm.library)
}

dependencies {
    implementation(libs.kotlinx.serialization.json)
}
```

### Domain Use Cases Module `build.gradle.kts`
```kotlin
plugins {
    alias(libs.plugins.project.jvm.library)
    alias(libs.plugins.project.android.hilt)
}

dependencies {
    implementation(project(":domain:featurename:models"))
    implementation(project(":data:featurename"))
    implementation(libs.kotlinx.coroutines)
}
```

## 8. Best Practices

### Project Organization
- Use feature-based modules for scalability
- Separate view and viewmodel layers
- Keep domain models in separate modules
- Use consistent naming conventions

### Build Configuration
- Leverage convention plugins for consistency
- Use version catalogs for dependency management
- Enable parallel builds and caching
- Configure proper JVM settings

### Development Workflow
1. Create modules using templates
2. Update dependencies as needed
3. Follow established architecture patterns
4. Use convention plugins consistently
5. Maintain version catalog regularly

This initialization guide provides a complete foundation for creating scalable, maintainable Android projects with standardized build configurations and modular architecture.