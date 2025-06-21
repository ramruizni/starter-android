# Template Files Collection

This document contains all the actual template files needed to create a complete Android project from scratch. These files can be copied directly and modified with project-specific values.

## 1. Root Project Files

### `gradle/libs.versions.toml`
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
retrofit = "2.9.0"

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
okhttp-logging = { group = "com.squareup.okhttp3", name = "logging-interceptor", version.ref = "okhttp" }
retrofit-core = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
retrofit-gson = { group = "com.squareup.retrofit2", name = "converter-gson", version.ref = "retrofit" }
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

# Network
network = [
    "okhttp-core",
    "okhttp-logging",
    "retrofit-core",
    "retrofit-gson",
    "gson"
]

# Testing
junit5 = [
    "junit5-api",
    "junit5-engine"
]
```

### `settings.gradle.kts`
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

rootProject.name = "PROJECT_NAME"

// Core modules
include(":app")
include(":core:database")
include(":core:network")
include(":core:domain")
include(":core:navigation")

// UI modules  
include(":ui:design-system")
include(":ui:components")

// Example feature modules (can be removed/modified)
include(":features:home:view")
include(":features:home:viewmodel")
include(":domain:home:models")
include(":domain:home:usecases")
include(":data:home")
```

### `build.gradle.kts`
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

### `gradle.properties`
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

## 2. Build Logic Templates

### `build-logic/settings.gradle.kts`
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

### `build-logic/convention/build.gradle.kts`
```kotlin
import org.jetbrains.kotlin.gradle.dsl.JvmTarget

plugins {
    `kotlin-dsl`
}

group = "PACKAGE_NAME.convention"

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

## 3. Convention Plugin Templates

### `build-logic/convention/src/main/java/PACKAGE_PATH/convention/ProjectConfig.kt`
```kotlin
package PACKAGE_NAME.convention

object ProjectConfig {
    const val MIN_SDK = 25
    const val COMPILE_SDK = 35
    const val TARGET_SDK = 35
}
```

### `build-logic/convention/src/main/java/PACKAGE_PATH/convention/ProjectExtensions.kt`
```kotlin
package PACKAGE_NAME.convention

import org.gradle.api.Project
import org.gradle.api.artifacts.VersionCatalog
import org.gradle.api.artifacts.VersionCatalogsExtension
import org.gradle.kotlin.dsl.getByType

internal val Project.libs
    get(): VersionCatalog = extensions.getByType<VersionCatalogsExtension>().named("libs")
```

### `build-logic/convention/src/main/java/PACKAGE_PATH/convention/Kotlin.kt`
```kotlin
package PACKAGE_NAME.convention

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

### `build-logic/convention/src/main/java/PACKAGE_PATH/convention/Serializer.kt`
```kotlin
package PACKAGE_NAME.convention

import org.gradle.api.Project
import org.gradle.kotlin.dsl.dependencies

internal fun Project.configureSerializer() {
    with(pluginManager) {
        apply(libs.findPlugin("kotlinx.serialization").get().get().pluginId)
    }

    dependencies {
        add("implementation", libs.findLibrary("kotlinx.serialization.json").get())
    }
}
```

### `build-logic/convention/src/main/java/PACKAGE_PATH/convention/Coroutines.kt`
```kotlin
package PACKAGE_NAME.convention

import org.gradle.api.Project
import org.gradle.kotlin.dsl.dependencies

internal fun Project.configureCoroutines() {
    dependencies {
        add("implementation", libs.findLibrary("kotlinx.coroutines").get())
        add("testImplementation", libs.findLibrary("kotlinx.coroutines.test").get())
    }
}
```

### `build-logic/convention/src/main/java/PACKAGE_PATH/convention/AndroidCompose.kt`
```kotlin
package PACKAGE_NAME.convention

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

### `build-logic/convention/src/main/java/PACKAGE_PATH/convention/BuildTypes.kt`
```kotlin
package PACKAGE_NAME.convention

import com.android.build.api.dsl.ApplicationExtension
import com.android.build.api.dsl.BuildType
import com.android.build.api.dsl.CommonExtension
import com.android.build.api.dsl.LibraryExtension
import org.gradle.api.Project

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

## 4. Convention Plugin Implementations

### `build-logic/convention/src/main/java/AndroidApplicationConventionPlugin.kt`
```kotlin
import com.android.build.api.dsl.ApplicationExtension
import PACKAGE_NAME.convention.ProjectConfig
import PACKAGE_NAME.convention.configureBuildTypes
import PACKAGE_NAME.convention.configureKotlinAndroid
import PACKAGE_NAME.convention.libs
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
                    applicationId = "PACKAGE_NAME"
                    targetSdk = ProjectConfig.TARGET_SDK
                    versionCode = 1
                    versionName = "1.0"
                    
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

### `build-logic/convention/src/main/java/AndroidLibraryConventionPlugin.kt`
```kotlin
import com.android.build.gradle.LibraryExtension
import PACKAGE_NAME.convention.ProjectConfig
import PACKAGE_NAME.convention.configureKotlinAndroid
import PACKAGE_NAME.convention.libs
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

### `build-logic/convention/src/main/java/AndroidApplicationComposeConventionPlugin.kt`
```kotlin
import com.android.build.api.dsl.ApplicationExtension
import PACKAGE_NAME.convention.configureAndroidCompose
import PACKAGE_NAME.convention.libs
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

### `build-logic/convention/src/main/java/AndroidLibraryComposeConventionPlugin.kt`
```kotlin
import com.android.build.gradle.LibraryExtension
import PACKAGE_NAME.convention.configureAndroidCompose
import PACKAGE_NAME.convention.libs
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

### `build-logic/convention/src/main/java/AndroidHiltConventionPlugin.kt`
```kotlin
import PACKAGE_NAME.convention.libs
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

### `build-logic/convention/src/main/java/AndroidRoomConventionPlugin.kt`
```kotlin
import PACKAGE_NAME.convention.libs
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

### `build-logic/convention/src/main/java/JvmLibraryConventionPlugin.kt`
```kotlin
import PACKAGE_NAME.convention.configureKotlinJvm
import PACKAGE_NAME.convention.libs
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

### `build-logic/convention/src/main/java/ArchViewConventionPlugin.kt`
```kotlin
import PACKAGE_NAME.convention.libs
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

### `build-logic/convention/src/main/java/ArchViewModelConventionPlugin.kt`
```kotlin
import PACKAGE_NAME.convention.libs
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

## 5. App Module Templates

### `app/build.gradle.kts`
```kotlin
plugins {
    alias(libs.plugins.project.android.application)
    alias(libs.plugins.project.android.application.compose)
    alias(libs.plugins.project.android.hilt)
}

dependencies {
    implementation(project(":core:database"))
    implementation(project(":core:network"))
    implementation(project(":core:navigation"))
    implementation(project(":ui:design-system"))
    
    // Example feature dependencies
    implementation(project(":features:home:view"))
    
    implementation(libs.androidx.core.splashscreen)
}
```

### `app/src/main/AndroidManifest.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:name=".MyApplication"
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.MyApp"
        tools:targetApi="31">
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:label="@string/app_name"
            android:theme="@style/Theme.MyApp">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

### `app/src/main/java/PACKAGE_PATH/MyApplication.kt`
```kotlin
package PACKAGE_NAME

import android.app.Application
import dagger.hilt.android.HiltAndroidApp

@HiltAndroidApp
class MyApplication : Application()
```

### `app/src/main/java/PACKAGE_PATH/MainActivity.kt`
```kotlin
package PACKAGE_NAME

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.core.splashscreen.SplashScreen.Companion.installSplashScreen
import dagger.hilt.android.AndroidEntryPoint
import PACKAGE_NAME.core.navigation.NavigationHost
import PACKAGE_NAME.ui.design.theme.MyAppTheme

@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        installSplashScreen()
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        
        setContent {
            MyAppTheme {
                NavigationHost()
            }
        }
    }
}
```

## 6. Basic Proguard Rules

### `app/proguard-rules.pro`
```proguard
# Add project specific ProGuard rules here.
# You can control the set of applied configuration files using the
# proguardFiles setting in build.gradle.

# Keep all application classes
-keep class PACKAGE_NAME.** { *; }

# Keep Hilt generated classes
-keep class dagger.hilt.** { *; }
-keep class javax.inject.** { *; }

# Keep Room entities
-keep @androidx.room.Entity class *
-keep @androidx.room.Database class * { *; }
-keep @androidx.room.Dao class * { *; }

# Keep Gson classes
-keepattributes Signature
-keepattributes *Annotation*
-keep class com.google.gson.** { *; }

# Keep Retrofit classes
-keep class retrofit2.** { *; }
-keepattributes Signature, InnerClasses, EnclosingMethod
-keepattributes RuntimeVisibleAnnotations, RuntimeVisibleParameterAnnotations

# Keep Kotlinx Serialization
-keepattributes *Annotation*, InnerClasses
-dontnote kotlinx.serialization.SerializationKt
-keep,includedescriptorclasses class PACKAGE_NAME.**$$serializer { *; }
-keepclassmembers class PACKAGE_NAME.** {
    *** Companion;
}
-keepclasseswithmembers class PACKAGE_NAME.** {
    kotlinx.serialization.KSerializer serializer(...);
}
```

## Usage Instructions

1. **Replace Placeholders**:
   - `PROJECT_NAME` → Your project name
   - `PACKAGE_NAME` → Your package name (e.g., `com.mycompany.myapp`)
   - `PACKAGE_PATH` → Package path (e.g., `com/mycompany/myapp`)

2. **Copy Files**: Copy these templates to create your project structure

3. **Run Initial Build**: Execute `./gradlew build` to verify setup

These templates provide a complete foundation for creating Android projects from scratch with all necessary configurations and patterns established.