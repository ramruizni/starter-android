# Convention Plugins Implementation Guide

This guide provides complete implementation details for all convention plugins used in the Android project initialization system, based on analysis of the Dazzle project architecture.

## Overview

Convention plugins standardize build configurations across modules, reducing duplication and ensuring consistency. Each plugin encapsulates specific functionality and can be composed with others.

## Plugin Architecture

### Core Principles
- **Single Responsibility**: Each plugin handles one specific concern
- **Composability**: Plugins can be combined for complex configurations
- **Company Agnostic**: No dependencies on proprietary libraries
- **Reusability**: Works across different projects and teams

## 1. Project Configuration Classes

### Create `ProjectConfig.kt`
```kotlin
// build-logic/convention/src/main/java/com/myproject/convention/ProjectConfig.kt
package com.myproject.convention

object ProjectConfig {
    const val MIN_SDK = 25
    const val COMPILE_SDK = 35
    const val TARGET_SDK = 35
}
```

### Create `ProjectExtensions.kt`
```kotlin
// build-logic/convention/src/main/java/com/myproject/convention/ProjectExtensions.kt
package com.myproject.convention

import org.gradle.api.Project
import org.gradle.api.artifacts.VersionCatalog
import org.gradle.api.artifacts.VersionCatalogsExtension
import org.gradle.kotlin.dsl.getByType

internal val Project.libs
    get(): VersionCatalog = extensions.getByType<VersionCatalogsExtension>().named("libs")
```

## 2. Core Configuration Functions

### Create `Kotlin.kt`
```kotlin
// build-logic/convention/src/main/java/com/myproject/convention/Kotlin.kt
package com.myproject.convention

import com.android.build.api.dsl.CommonExtension
import org.gradle.api.JavaVersion
import org.gradle.api.Project
import org.gradle.api.plugins.JavaPluginExtension
import org.gradle.api.tasks.testing.Test
import org.gradle.kotlin.dsl.configure
import org.gradle.kotlin.dsl.dependencies
import org.gradle.kotlin.dsl.provideDelegate
import org.gradle.kotlin.dsl.withType
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

/**
 * Configure base Kotlin with Android options
 */
internal fun Project.configureKotlinAndroid(
    commonExtension: CommonExtension<*, *, *, *, *, *>,
) {
    commonExtension.apply {
        compileSdk = ProjectConfig.COMPILE_SDK

        defaultConfig {
            minSdk = ProjectConfig.MIN_SDK
        }

        compileOptions {
            // Up to Java 11 APIs are available through desugaring
            sourceCompatibility = JavaVersion.VERSION_11
            targetCompatibility = JavaVersion.VERSION_11
            isCoreLibraryDesugaringEnabled = true
        }
    }

    configureKotlin()

    dependencies {
        add("coreLibraryDesugaring", libs.findLibrary("android.desugar.jdk.libs").get())
    }
}

/**
 * Configure base Kotlin options for JVM (non-Android)
 */
internal fun Project.configureKotlinJvm() {
    extensions.configure<JavaPluginExtension> {
        sourceCompatibility = JavaVersion.VERSION_11
        targetCompatibility = JavaVersion.VERSION_11
    }

    configureKotlin()
}

/**
 * Configure base Kotlin options
 */
private fun Project.configureKotlin() {
    configureSerializer()
    configureCoroutines()

    tasks.withType<KotlinCompile>().configureEach {
        kotlinOptions {
            jvmTarget = JavaVersion.VERSION_11.toString()
            
            val warningsAsErrors: String? by project
            allWarningsAsErrors = warningsAsErrors.toBoolean()
            
            freeCompilerArgs = freeCompilerArgs + listOf(
                "-opt-in=kotlin.RequiresOptIn",
                "-opt-in=kotlinx.coroutines.ExperimentalCoroutinesApi",
                "-opt-in=kotlinx.coroutines.FlowPreview",
            )
        }
    }

    tasks.withType<Test> {
        useJUnitPlatform()
    }
}
```

### Create `Serializer.kt`
```kotlin
// build-logic/convention/src/main/java/com/myproject/convention/Serializer.kt
package com.myproject.convention

import org.gradle.api.Project
import org.gradle.kotlin.dsl.dependencies

internal fun Project.configureSerializer() {
    with(pluginManager) {
        apply(libs.findPlugin("kotlinx-serialization").get().get().pluginId)
    }

    dependencies {
        add("implementation", libs.findLibrary("kotlinx.serialization.json").get())
    }
}
```

### Create `Coroutines.kt`
```kotlin
// build-logic/convention/src/main/java/com/myproject/convention/Coroutines.kt
package com.myproject.convention

import org.gradle.api.Project
import org.gradle.kotlin.dsl.dependencies

internal fun Project.configureCoroutines() {
    dependencies {
        add("implementation", libs.findLibrary("kotlinx.coroutines").get())
        add("testImplementation", libs.findLibrary("kotlinx.coroutines.test").get())
    }
}
```

### Create `AndroidCompose.kt`
```kotlin
// build-logic/convention/src/main/java/com/myproject/convention/AndroidCompose.kt
package com.myproject.convention

import com.android.build.api.dsl.CommonExtension
import org.gradle.api.Project
import org.gradle.kotlin.dsl.dependencies
import org.gradle.kotlin.dsl.withType
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

/**
 * Configure Compose-specific options
 */
internal fun Project.configureAndroidCompose(
    commonExtension: CommonExtension<*, *, *, *, *, *>,
) {
    commonExtension.apply {
        buildFeatures {
            compose = true
        }

        dependencies {
            val bom = libs.findLibrary("androidx.compose.bom").get()
            add("implementation", platform(bom))
            add("implementation", libs.findBundle("androidx.compose").get())

            add("androidTestImplementation", platform(bom))
        }
    }

    tasks.withType<KotlinCompile>().configureEach {
        kotlinOptions {
            freeCompilerArgs = freeCompilerArgs + buildComposeMetricsParameters()
        }
    }
}

private fun Project.buildComposeMetricsParameters(): List<String> {
    val metricParameters = mutableListOf<String>()
    val enableMetricsProvider = project.providers.gradleProperty("enableComposeCompilerMetrics")
    val relativePath = projectDir.relativeTo(rootDir)
    val buildDir = layout.buildDirectory.get().asFile
    val enableMetrics = (enableMetricsProvider.orNull == "true")
    
    if (enableMetrics) {
        val metricsFolder = buildDir.resolve("compose-metrics").resolve(relativePath)
        metricParameters.add("-P")
        metricParameters.add(
            "plugin:androidx.compose.compiler.plugins.kotlin:metricsDestination=" + metricsFolder.absolutePath
        )
    }

    val enableReportsProvider = project.providers.gradleProperty("enableComposeCompilerReports")
    val enableReports = (enableReportsProvider.orNull == "true")
    
    if (enableReports) {
        val reportsFolder = buildDir.resolve("compose-reports").resolve(relativePath)
        metricParameters.add("-P")
        metricParameters.add(
            "plugin:androidx.compose.compiler.plugins.kotlin:reportsDestination=" + reportsFolder.absolutePath
        )
    }
    
    return metricParameters.toList()
}
```

### Create `BuildTypes.kt`
```kotlin
// build-logic/convention/src/main/java/com/myproject/convention/BuildTypes.kt
package com.myproject.convention

import com.android.build.api.dsl.ApplicationExtension
import com.android.build.api.dsl.BuildType
import com.android.build.api.dsl.CommonExtension
import com.android.build.api.dsl.LibraryExtension
import org.gradle.api.Project
import org.gradle.kotlin.dsl.configure

/**
 * Configure build types for applications
 */
internal fun Project.configureBuildTypes(
    commonExtension: CommonExtension<*, *, *, *, *, *>,
) {
    commonExtension.apply {
        buildTypes {
            getByName("debug") {
                configureDebugBuildType()
            }
            getByName("release") {
                configureReleaseBuildType(commonExtension)
            }
        }
    }
}

private fun BuildType.configureDebugBuildType() {
    isDebuggable = true
    applicationIdSuffix = ".debug"
    versionNameSuffix = "-debug"
}

private fun BuildType.configureReleaseBuildType(
    commonExtension: CommonExtension<*, *, *, *, *, *>
) {
    isMinifyEnabled = true
    
    when (commonExtension) {
        is ApplicationExtension -> {
            isShrinkResources = true
            proguardFiles(
                commonExtension.getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
        is LibraryExtension -> {
            consumerProguardFiles("consumer-rules.pro")
        }
    }
}
```

## 3. Base Convention Plugins

### Create `AndroidApplicationConventionPlugin.kt`
```kotlin
// build-logic/convention/src/main/java/AndroidApplicationConventionPlugin.kt
import com.android.build.api.dsl.ApplicationExtension
import com.myproject.convention.ProjectConfig
import com.myproject.convention.configureBuildTypes
import com.myproject.convention.configureKotlinAndroid
import com.myproject.convention.libs
import org.gradle.api.Plugin
import org.gradle.api.Project
import org.gradle.kotlin.dsl.configure
import org.gradle.kotlin.dsl.dependencies

class AndroidApplicationConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            with(pluginManager) {
                apply(libs.findPlugin("android.application").get().get().pluginId)
                apply(libs.findPlugin("kotlin.android").get().get().pluginId)
            }

            extensions.configure<ApplicationExtension> {
                configureKotlinAndroid(this)

                defaultConfig {
                    targetSdk = ProjectConfig.TARGET_SDK
                    testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
                    vectorDrawables {
                        useSupportLibrary = true
                    }
                }

                configureBuildTypes(this)

                packaging {
                    resources {
                        excludes += "/META-INF/{AL2.0,LGPL2.1}"
                    }
                }
            }

            dependencies {
                add("implementation", libs.findBundle("android.ui").get())
                add("debugImplementation", libs.findLibrary("androidx.ui.tooling").get())
                add("debugImplementation", libs.findLibrary("androidx.ui.test.manifest").get())
            }
        }
    }
}
```

### Create `AndroidLibraryConventionPlugin.kt`
```kotlin
// build-logic/convention/src/main/java/AndroidLibraryConventionPlugin.kt
import com.android.build.gradle.LibraryExtension
import com.myproject.convention.ProjectConfig
import com.myproject.convention.configureKotlinAndroid
import com.myproject.convention.libs
import org.gradle.api.Plugin
import org.gradle.api.Project
import org.gradle.kotlin.dsl.configure
import org.gradle.kotlin.dsl.dependencies

class AndroidLibraryConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            with(pluginManager) {
                apply(libs.findPlugin("android.library").get().get().pluginId)
                apply(libs.findPlugin("kotlin.android").get().get().pluginId)
            }

            extensions.configure<LibraryExtension> {
                configureKotlinAndroid(this)
                defaultConfig.targetSdk = ProjectConfig.TARGET_SDK
            }

            dependencies {
                add("implementation", libs.findLibrary("androidx.core.ktx").get())
            }
        }
    }
}
```

### Create `JvmLibraryConventionPlugin.kt`
```kotlin
// build-logic/convention/src/main/java/JvmLibraryConventionPlugin.kt
import com.myproject.convention.configureKotlinJvm
import com.myproject.convention.libs
import org.gradle.api.Plugin
import org.gradle.api.Project

class JvmLibraryConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            with(pluginManager) {
                apply(libs.findPlugin("kotlin.jvm").get().get().pluginId)
            }
            
            configureKotlinJvm()
        }
    }
}
```

## 4. Compose Convention Plugins

### Create `AndroidApplicationComposeConventionPlugin.kt`
```kotlin
// build-logic/convention/src/main/java/AndroidApplicationComposeConventionPlugin.kt
import com.android.build.api.dsl.ApplicationExtension
import com.myproject.convention.configureAndroidCompose
import com.myproject.convention.libs
import org.gradle.api.Plugin
import org.gradle.api.Project
import org.gradle.kotlin.dsl.getByType

class AndroidApplicationComposeConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            pluginManager.apply(libs.findPlugin("kotlin.compose").get().get().pluginId)
            
            val extension = extensions.getByType<ApplicationExtension>()
            configureAndroidCompose(extension)
        }
    }
}
```

### Create `AndroidLibraryComposeConventionPlugin.kt`
```kotlin
// build-logic/convention/src/main/java/AndroidLibraryComposeConventionPlugin.kt
import com.android.build.gradle.LibraryExtension
import com.myproject.convention.configureAndroidCompose
import com.myproject.convention.libs
import org.gradle.api.Plugin
import org.gradle.api.Project
import org.gradle.kotlin.dsl.getByType

class AndroidLibraryComposeConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            pluginManager.apply(libs.findPlugin("kotlin.compose").get().get().pluginId)
            
            val extension = extensions.getByType<LibraryExtension>()
            configureAndroidCompose(extension)
        }
    }
}
```

## 5. Dependency Injection Plugins

### Create `AndroidHiltConventionPlugin.kt`
```kotlin
// build-logic/convention/src/main/java/AndroidHiltConventionPlugin.kt
import com.myproject.convention.libs
import org.gradle.api.Plugin
import org.gradle.api.Project
import org.gradle.kotlin.dsl.dependencies

class AndroidHiltConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            with(pluginManager) {
                apply(libs.findPlugin("ksp").get().get().pluginId)
                apply(libs.findPlugin("hilt").get().get().pluginId)
            }

            dependencies {
                add("implementation", libs.findLibrary("hilt.android").get())
                add("ksp", libs.findLibrary("hilt.compiler").get())
                
                add("kspTest", libs.findLibrary("hilt.compiler").get())
                add("testImplementation", libs.findLibrary("hilt.android.testing").get())
            }
        }
    }
}
```

## 6. Database Plugins

### Create `AndroidRoomConventionPlugin.kt`
```kotlin
// build-logic/convention/src/main/java/AndroidRoomConventionPlugin.kt
import com.myproject.convention.libs
import org.gradle.api.Plugin
import org.gradle.api.Project
import org.gradle.kotlin.dsl.dependencies

class AndroidRoomConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            with(pluginManager) {
                apply(libs.findPlugin("room").get().get().pluginId)
                apply(libs.findPlugin("ksp").get().get().pluginId)
            }

            dependencies {
                add("implementation", libs.findBundle("room.datasource").get())
                add("ksp", libs.findLibrary("room.compiler").get())
            }
        }
    }
}
```

## 7. Architecture Convention Plugins

### Create `ArchViewConventionPlugin.kt`
```kotlin
// build-logic/convention/src/main/java/ArchViewConventionPlugin.kt
import com.myproject.convention.libs
import org.gradle.api.Plugin
import org.gradle.api.Project
import org.gradle.kotlin.dsl.dependencies
import org.gradle.kotlin.dsl.project

class ArchViewConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            pluginManager.apply {
                apply("project.android.library")
                apply("project.android.library.compose")
                apply("project.android.hilt")
            }

            dependencies {
                add("implementation", libs.findBundle("android.ui").get())
                
                add("debugImplementation", libs.findLibrary("androidx.ui.tooling").get())
                add("debugImplementation", libs.findLibrary("androidx.ui.test.manifest").get())

                // Common UI modules
                add("implementation", project(":ui:components"))
                add("implementation", project(":ui:design-system"))
                
                // Navigation
                add("implementation", project(":core:navigation"))
            }
        }
    }
}
```

### Create `ArchViewModelConventionPlugin.kt`
```kotlin
// build-logic/convention/src/main/java/ArchViewModelConventionPlugin.kt
import com.myproject.convention.libs
import org.gradle.api.Plugin
import org.gradle.api.Project
import org.gradle.kotlin.dsl.dependencies
import org.gradle.kotlin.dsl.project

class ArchViewModelConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            pluginManager.apply {
                apply("project.android.library")
                apply("project.android.hilt")
            }

            dependencies {
                add("implementation", libs.findLibrary("androidx.lifecycle.viewmodel.ktx").get())
                add("implementation", libs.findLibrary("kotlinx.coroutines").get())
                
                // Core domain
                add("implementation", project(":core:domain"))
            }
        }
    }
}
```

## 8. Testing Convention Plugins

### Create `AndroidTestConventionPlugin.kt`
```kotlin
// build-logic/convention/src/main/java/AndroidTestConventionPlugin.kt
import com.myproject.convention.libs
import org.gradle.api.Plugin
import org.gradle.api.Project
import org.gradle.kotlin.dsl.dependencies

class AndroidTestConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            dependencies {
                add("testImplementation", libs.findLibrary("androidx.junit").get())
                add("androidTestImplementation", libs.findLibrary("androidx.junit").get())
                add("androidTestImplementation", libs.findLibrary("androidx.espresso.core").get())
                add("androidTestImplementation", libs.findLibrary("androidx.ui.test.junit4").get())
            }
        }
    }
}
```

### Create `Junit5TestConventionPlugin.kt`
```kotlin
// build-logic/convention/src/main/java/Junit5TestConventionPlugin.kt
import com.myproject.convention.libs
import org.gradle.api.Plugin
import org.gradle.api.Project
import org.gradle.kotlin.dsl.dependencies

class Junit5TestConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            dependencies {
                add("testImplementation", libs.findBundle("junit5").get())
                add("testRuntimeOnly", libs.findLibrary("junit5.engine").get())
            }
        }
    }
}
```

## 9. Plugin Usage Examples

### Application Module
```kotlin
// app/build.gradle.kts
plugins {
    alias(libs.plugins.project.android.application)
    alias(libs.plugins.project.android.application.compose)
    alias(libs.plugins.project.android.hilt)
}

dependencies {
    implementation(project(":core:ui"))
    implementation(project(":core:navigation"))
    implementation(project(":features:home:view"))
    implementation(project(":features:profile:view"))
}
```

### Feature View Module
```kotlin
// features/home/view/build.gradle.kts
plugins {
    alias(libs.plugins.project.arch.view)
}

dependencies {
    implementation(project(":features:home:viewmodel"))
    implementation(project(":domain:home:models"))
}
```

### Feature ViewModel Module
```kotlin
// features/home/viewmodel/build.gradle.kts
plugins {
    alias(libs.plugins.project.arch.viewmodel)
}

dependencies {
    implementation(project(":domain:home:usecases"))
    implementation(project(":domain:home:models"))
}
```

### Domain Models Module
```kotlin
// domain/home/models/build.gradle.kts
plugins {
    alias(libs.plugins.project.jvm.library)
}

dependencies {
    implementation(libs.kotlinx.serialization.json)
}
```

### Data Layer Module
```kotlin
// data/home/build.gradle.kts
plugins {
    alias(libs.plugins.project.android.library)
    alias(libs.plugins.project.android.hilt)
    alias(libs.plugins.project.android.room)
}

dependencies {
    implementation(project(":domain:home:models"))
    implementation(libs.okhttp.core)
    implementation(libs.gson)
}
```

## 10. Plugin Composition Patterns

### Complex Feature Module
```kotlin
// Complex module combining multiple plugins
plugins {
    alias(libs.plugins.project.android.library)
    alias(libs.plugins.project.android.library.compose)  
    alias(libs.plugins.project.android.hilt)
    alias(libs.plugins.project.android.room)
}
```

### Minimal JVM Module
```kotlin
// Simple domain module
plugins {
    alias(libs.plugins.project.jvm.library)
}
```

## 11. Best Practices

### Plugin Design
- Keep plugins focused on single responsibilities
- Use internal functions for shared configuration
- Leverage version catalogs for dependency management
- Make plugins composable and reusable

### Configuration
- Extract common configurations to utility functions
- Use project properties for feature flags
- Maintain consistent naming conventions
- Document plugin purposes and usage

### Testing
- Test plugin applications in isolation
- Verify dependency configurations
- Check build script generation
- Validate plugin composition scenarios

This convention plugin system provides a scalable, maintainable approach to build configuration management across Android projects, ensuring consistency while remaining flexible and company-agnostic.