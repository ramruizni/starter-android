# Core Modules Setup Guide

This guide provides complete setup for essential core modules that form the foundation of your Android project. These modules provide shared functionality across features.

## Core Module Overview

### Required Core Modules:
1. **`:core:database`** - Room database configuration and DAOs
2. **`:core:network`** - HTTP client and API configuration  
3. **`:core:domain`** - Base domain classes and utilities
4. **`:core:navigation`** - Navigation utilities and route definitions

### Optional Core Modules:
5. **`:core:testing`** - Shared testing utilities
6. **`:core:analytics`** - Analytics and tracking
7. **`:core:preferences`** - DataStore preferences management

## 1. Core Database Module (:core:database)

### Directory Structure
```
core/database/
├── src/
│   ├── main/
│   │   ├── java/com/myproject/core/database/
│   │   │   ├── MyAppDatabase.kt
│   │   │   ├── DatabaseModule.kt
│   │   │   ├── converters/
│   │   │   │   └── TypeConverters.kt
│   │   │   └── migrations/
│   │   │       └── DatabaseMigrations.kt
│   │   └── AndroidManifest.xml
│   └── test/
│       └── java/com/myproject/core/database/
│           └── DatabaseTest.kt
└── build.gradle.kts
```

### Build Configuration
```kotlin
// core/database/build.gradle.kts
plugins {
    alias(libs.plugins.project.android.library)
    alias(libs.plugins.project.android.hilt)
    alias(libs.plugins.project.android.room)
}

dependencies {
    implementation(project(":core:domain"))
    
    implementation(libs.kotlinx.coroutines)
    implementation(libs.gson)
    
    // Testing
    testImplementation(libs.junit5.api)
    testImplementation(libs.kotlinx.coroutines.test)
}
```

### Database Implementation
```kotlin
// core/database/src/main/java/com/myproject/core/database/MyAppDatabase.kt
package com.myproject.core.database

import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase
import androidx.room.TypeConverters
import com.myproject.core.database.converters.MyAppTypeConverters
import com.myproject.core.database.migrations.MIGRATION_1_2

@Database(
    entities = [
        // Add your entities here as you create them
        // ExampleEntity::class,
    ],
    version = 1,
    exportSchema = true
)
@TypeConverters(MyAppTypeConverters::class)
abstract class MyAppDatabase : RoomDatabase() {
    
    // Add DAOs here as you create them
    // abstract fun exampleDao(): ExampleDao
    
    companion object {
        const val DATABASE_NAME = "myapp_database"
    }
}
```

### Type Converters
```kotlin
// core/database/src/main/java/com/myproject/core/database/converters/TypeConverters.kt
package com.myproject.core.database.converters

import androidx.room.TypeConverter
import com.google.gson.Gson
import com.google.gson.reflect.TypeToken

class MyAppTypeConverters {
    
    private val gson = Gson()
    
    @TypeConverter
    fun fromStringList(value: List<String>): String {
        return gson.toJson(value)
    }
    
    @TypeConverter
    fun toStringList(value: String): List<String> {
        val listType = object : TypeToken<List<String>>() {}.type
        return gson.fromJson(value, listType)
    }
    
    @TypeConverter
    fun fromStringMap(value: Map<String, String>): String {
        return gson.toJson(value)
    }
    
    @TypeConverter
    fun toStringMap(value: String): Map<String, String> {
        val mapType = object : TypeToken<Map<String, String>>() {}.type
        return gson.fromJson(value, mapType)
    }
    
    @TypeConverter
    fun fromTimestamp(value: Long?): java.util.Date? {
        return value?.let { java.util.Date(it) }
    }
    
    @TypeConverter
    fun dateToTimestamp(date: java.util.Date?): Long? {
        return date?.time
    }
}
```

### Database Module
```kotlin
// core/database/src/main/java/com/myproject/core/database/DatabaseModule.kt
package com.myproject.core.database

import android.content.Context
import androidx.room.Room
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.android.qualifiers.ApplicationContext
import dagger.hilt.components.SingletonComponent
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {
    
    @Provides
    @Singleton
    fun provideMyAppDatabase(
        @ApplicationContext context: Context
    ): MyAppDatabase {
        return Room.databaseBuilder(
            context,
            MyAppDatabase::class.java,
            MyAppDatabase.DATABASE_NAME
        )
            .fallbackToDestructiveMigration() // Remove in production
            .build()
    }
    
    // Add DAO providers here as you create them
    // @Provides
    // fun provideExampleDao(database: MyAppDatabase): ExampleDao = database.exampleDao()
}
```

### Database Migrations
```kotlin
// core/database/src/main/java/com/myproject/core/database/migrations/DatabaseMigrations.kt
package com.myproject.core.database.migrations

import androidx.room.migration.Migration
import androidx.sqlite.db.SupportSQLiteDatabase

val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        // Add migration logic here when you need to migrate from version 1 to 2
        // Example:
        // database.execSQL("ALTER TABLE example_table ADD COLUMN new_column TEXT")
    }
}
```

## 2. Core Network Module (:core:network)

### Directory Structure
```
core/network/
├── src/
│   ├── main/
│   │   ├── java/com/myproject/core/network/
│   │   │   ├── NetworkModule.kt
│   │   │   ├── interceptors/
│   │   │   │   ├── AuthInterceptor.kt
│   │   │   │   └── LoggingInterceptor.kt
│   │   │   ├── utils/
│   │   │   │   └── NetworkUtils.kt
│   │   │   └── models/
│   │   │       ├── ApiResponse.kt
│   │   │       └── NetworkResult.kt
│   │   └── AndroidManifest.xml
│   └── test/
│       └── java/com/myproject/core/network/
│           └── NetworkModuleTest.kt
└── build.gradle.kts
```

### Build Configuration
```kotlin
// core/network/build.gradle.kts
plugins {
    alias(libs.plugins.project.android.library)
    alias(libs.plugins.project.android.hilt)
}

dependencies {
    implementation(project(":core:domain"))
    
    implementation(libs.bundle.network)
    implementation(libs.kotlinx.coroutines)
    implementation(libs.kotlinx.serialization.json)
    
    // Testing
    testImplementation(libs.junit5.api)
    testImplementation(libs.kotlinx.coroutines.test)
}
```

### Network Models
```kotlin
// core/network/src/main/java/com/myproject/core/network/models/ApiResponse.kt
package com.myproject.core.network.models

import kotlinx.serialization.Serializable

@Serializable
data class ApiResponse<T>(
    val data: T? = null,
    val message: String? = null,
    val success: Boolean = true,
    val errorCode: String? = null
)

@Serializable
data class ApiError(
    val code: String,
    val message: String,
    val details: Map<String, String>? = null
)
```

### Network Result
```kotlin
// core/network/src/main/java/com/myproject/core/network/models/NetworkResult.kt
package com.myproject.core.network.models

sealed class NetworkResult<out T> {
    data class Success<T>(val data: T) : NetworkResult<T>()
    data class Error(val exception: Throwable) : NetworkResult<Nothing>()
    data object Loading : NetworkResult<Nothing>()
}

inline fun <T> NetworkResult<T>.onSuccess(action: (value: T) -> Unit): NetworkResult<T> {
    if (this is NetworkResult.Success) action(data)
    return this
}

inline fun <T> NetworkResult<T>.onError(action: (exception: Throwable) -> Unit): NetworkResult<T> {
    if (this is NetworkResult.Error) action(exception)
    return this
}
```

### Auth Interceptor
```kotlin
// core/network/src/main/java/com/myproject/core/network/interceptors/AuthInterceptor.kt
package com.myproject.core.network.interceptors

import okhttp3.Interceptor
import okhttp3.Response
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class AuthInterceptor @Inject constructor(
    // Inject your token provider here
    // private val tokenProvider: TokenProvider
) : Interceptor {
    
    override fun intercept(chain: Interceptor.Chain): Response {
        val originalRequest = chain.request()
        
        // Add authentication header if token is available
        val token = getAuthToken()
        
        val requestBuilder = originalRequest.newBuilder()
        
        if (token.isNotEmpty()) {
            requestBuilder.addHeader("Authorization", "Bearer $token")
        }
        
        requestBuilder.addHeader("Content-Type", "application/json")
        requestBuilder.addHeader("Accept", "application/json")
        
        return chain.proceed(requestBuilder.build())
    }
    
    private fun getAuthToken(): String {
        // Implement token retrieval logic
        // return tokenProvider.getToken() ?: ""
        return ""
    }
}
```

### Network Module
```kotlin
// core/network/src/main/java/com/myproject/core/network/NetworkModule.kt
package com.myproject.core.network

import com.google.gson.Gson
import com.google.gson.GsonBuilder
import com.myproject.core.network.interceptors.AuthInterceptor
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent
import okhttp3.OkHttpClient
import okhttp3.logging.HttpLoggingInterceptor
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory
import java.util.concurrent.TimeUnit
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    
    private const val BASE_URL = "https://api.myapp.com/"
    private const val TIMEOUT_SECONDS = 30L
    
    @Provides
    @Singleton
    fun provideGson(): Gson {
        return GsonBuilder()
            .setLenient()
            .create()
    }
    
    @Provides
    @Singleton
    fun provideLoggingInterceptor(): HttpLoggingInterceptor {
        return HttpLoggingInterceptor().apply {
            level = HttpLoggingInterceptor.Level.BODY
        }
    }
    
    @Provides
    @Singleton
    fun provideOkHttpClient(
        authInterceptor: AuthInterceptor,
        loggingInterceptor: HttpLoggingInterceptor
    ): OkHttpClient {
        return OkHttpClient.Builder()
            .addInterceptor(authInterceptor)
            .addInterceptor(loggingInterceptor)
            .connectTimeout(TIMEOUT_SECONDS, TimeUnit.SECONDS)
            .readTimeout(TIMEOUT_SECONDS, TimeUnit.SECONDS)
            .writeTimeout(TIMEOUT_SECONDS, TimeUnit.SECONDS)
            .build()
    }
    
    @Provides
    @Singleton
    fun provideRetrofit(
        okHttpClient: OkHttpClient,
        gson: Gson
    ): Retrofit {
        return Retrofit.Builder()
            .baseUrl(BASE_URL)
            .client(okHttpClient)
            .addConverterFactory(GsonConverterFactory.create(gson))
            .build()
    }
}
```

## 3. Core Domain Module (:core:domain)

### Directory Structure
```
core/domain/
├── src/
│   ├── main/
│   │   └── java/com/myproject/core/domain/
│   │       ├── models/
│   │       │   ├── BaseEntity.kt
│   │       │   └── Result.kt
│   │       ├── repository/
│   │       │   └── BaseRepository.kt
│   │       └── utils/
│   │           ├── DateUtils.kt
│   │           └── Extensions.kt
│   └── test/
│       └── java/com/myproject/core/domain/
│           └── UtilsTest.kt
└── build.gradle.kts
```

### Build Configuration
```kotlin
// core/domain/build.gradle.kts
plugins {
    alias(libs.plugins.project.jvm.library)
}

dependencies {
    implementation(libs.kotlinx.coroutines)
    implementation(libs.kotlinx.serialization.json)
    implementation(libs.kotlinx.collections.immutable)
    
    // Testing
    testImplementation(libs.junit5.api)
    testImplementation(libs.kotlinx.coroutines.test)
}
```

### Base Entity
```kotlin
// core/domain/src/main/java/com/myproject/core/domain/models/BaseEntity.kt
package com.myproject.core.domain.models

import kotlinx.serialization.Serializable

@Serializable
abstract class BaseEntity {
    abstract val id: String
    abstract val createdAt: Long
    abstract val updatedAt: Long
}

interface Identifiable {
    val id: String
}

interface Timestamped {
    val createdAt: Long
    val updatedAt: Long
}
```

### Result Wrapper
```kotlin
// core/domain/src/main/java/com/myproject/core/domain/models/Result.kt
package com.myproject.core.domain.models

sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val exception: Throwable) : Result<Nothing>()
    data object Loading : Result<Nothing>()
}

inline fun <T> Result<T>.onSuccess(action: (value: T) -> Unit): Result<T> {
    if (this is Result.Success) action(data)
    return this
}

inline fun <T> Result<T>.onError(action: (exception: Throwable) -> Unit): Result<T> {
    if (this is Result.Error) action(exception)
    return this
}

inline fun <T> Result<T>.onLoading(action: () -> Unit): Result<T> {
    if (this is Result.Loading) action()
    return this
}

fun <T> Result<T>.getOrNull(): T? = when (this) {
    is Result.Success -> data
    else -> null
}

fun <T> Result<T>.getOrThrow(): T = when (this) {
    is Result.Success -> data
    is Result.Error -> throw exception
    is Result.Loading -> throw IllegalStateException("Result is loading")
}
```

### Base Repository
```kotlin
// core/domain/src/main/java/com/myproject/core/domain/repository/BaseRepository.kt
package com.myproject.core.domain.repository

import com.myproject.core.domain.models.Result
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.catch
import kotlinx.coroutines.flow.map
import kotlinx.coroutines.flow.onStart

abstract class BaseRepository {
    
    protected fun <T> Flow<T>.asResult(): Flow<Result<T>> {
        return this
            .map<T, Result<T>> { Result.Success(it) }
            .onStart { emit(Result.Loading) }
            .catch { emit(Result.Error(it)) }
    }
    
    protected suspend fun <T> safeCall(action: suspend () -> T): Result<T> {
        return try {
            Result.Success(action())
        } catch (e: Exception) {
            Result.Error(e)
        }
    }
}
```

### Date Utils
```kotlin
// core/domain/src/main/java/com/myproject/core/domain/utils/DateUtils.kt
package com.myproject.core.domain.utils

import java.text.SimpleDateFormat
import java.util.*

object DateUtils {
    
    private const val DEFAULT_DATE_FORMAT = "yyyy-MM-dd HH:mm:ss"
    private const val DATE_ONLY_FORMAT = "yyyy-MM-dd"
    private const val TIME_ONLY_FORMAT = "HH:mm:ss"
    
    fun getCurrentTimestamp(): Long = System.currentTimeMillis()
    
    fun formatTimestamp(timestamp: Long, format: String = DEFAULT_DATE_FORMAT): String {
        val date = Date(timestamp)
        val formatter = SimpleDateFormat(format, Locale.getDefault())
        return formatter.format(date)
    }
    
    fun parseDate(dateString: String, format: String = DEFAULT_DATE_FORMAT): Long? {
        return try {
            val formatter = SimpleDateFormat(format, Locale.getDefault())
            formatter.parse(dateString)?.time
        } catch (e: Exception) {
            null
        }
    }
    
    fun isToday(timestamp: Long): Boolean {
        val today = Calendar.getInstance()
        val date = Calendar.getInstance().apply { timeInMillis = timestamp }
        
        return today.get(Calendar.YEAR) == date.get(Calendar.YEAR) &&
                today.get(Calendar.DAY_OF_YEAR) == date.get(Calendar.DAY_OF_YEAR)
    }
    
    fun daysBetween(startTimestamp: Long, endTimestamp: Long): Int {
        val startDate = Calendar.getInstance().apply { timeInMillis = startTimestamp }
        val endDate = Calendar.getInstance().apply { timeInMillis = endTimestamp }
        
        val diffInMillis = endDate.timeInMillis - startDate.timeInMillis
        return (diffInMillis / (24 * 60 * 60 * 1000)).toInt()
    }
}
```

### Extensions
```kotlin
// core/domain/src/main/java/com/myproject/core/domain/utils/Extensions.kt
package com.myproject.core.domain.utils

// String Extensions
fun String?.isNotNullOrEmpty(): Boolean = !this.isNullOrEmpty()

fun String?.isNotNullOrBlank(): Boolean = !this.isNullOrBlank()

fun String.capitalize(): String = 
    this.replaceFirstChar { if (it.isLowerCase()) it.titlecase() else it.toString() }

// Collection Extensions
fun <T> List<T>?.isNotNullOrEmpty(): Boolean = !this.isNullOrEmpty()

fun <T> List<T>.safeGet(index: Int): T? = 
    if (index >= 0 && index < size) this[index] else null

// Numeric Extensions
fun Int?.orZero(): Int = this ?: 0

fun Long?.orZero(): Long = this ?: 0L

fun Double?.orZero(): Double = this ?: 0.0

fun Float?.orZero(): Float = this ?: 0f

// Boolean Extensions
fun Boolean?.orFalse(): Boolean = this ?: false

fun Boolean?.orTrue(): Boolean = this ?: true
```

## 4. Core Navigation Module (:core:navigation)

### Directory Structure
```
core/navigation/
├── src/
│   ├── main/
│   │   ├── java/com/myproject/core/navigation/
│   │   │   ├── NavigationHost.kt
│   │   │   ├── routes/
│   │   │   │   ├── RootRoute.kt
│   │   │   │   └── NavigationRoute.kt
│   │   │   ├── utils/
│   │   │   │   ├── NavigationExtensions.kt
│   │   │   │   └── RouteAnimations.kt
│   │   │   └── di/
│   │   │       └── NavigationModule.kt
│   │   └── AndroidManifest.xml
│   └── test/
│       └── java/com/myproject/core/navigation/
│           └── NavigationTest.kt
└── build.gradle.kts
```

### Build Configuration
```kotlin
// core/navigation/build.gradle.kts
plugins {
    alias(libs.plugins.project.android.library)
    alias(libs.plugins.project.android.library.compose)
    alias(libs.plugins.project.android.hilt)
}

dependencies {
    implementation(project(":core:domain"))
    implementation(project(":ui:design-system"))
    
    implementation(libs.androidx.navigation.compose)
    implementation(libs.androidx.hilt.navigation.compose)
    implementation(libs.kotlinx.serialization.json)
    implementation(libs.gson)
    
    // Testing
    testImplementation(libs.junit5.api)
}
```

### Navigation Routes
```kotlin
// core/navigation/src/main/java/com/myproject/core/navigation/routes/NavigationRoute.kt
package com.myproject.core.navigation.routes

import kotlinx.serialization.Serializable

@Serializable
sealed class NavigationRoute

@Serializable
data object RootRoute : NavigationRoute()

// Main navigation routes
@Serializable
data object HomeRoute : NavigationRoute()

@Serializable
data object ProfileRoute : NavigationRoute()

@Serializable
data object SettingsRoute : NavigationRoute()

// Parameterized routes examples
@Serializable
data class DetailRoute(
    val itemId: String,
    val category: String? = null
) : NavigationRoute()

@Serializable
data class EditRoute(
    val itemId: String,
    val mode: String = "edit"
) : NavigationRoute()
```

### Route Animations
```kotlin
// core/navigation/src/main/java/com/myproject/core/navigation/utils/RouteAnimations.kt
package com.myproject.core.navigation.utils

import androidx.compose.animation.AnimatedContentTransitionScope
import androidx.compose.animation.EnterTransition
import androidx.compose.animation.ExitTransition
import androidx.compose.animation.core.tween
import androidx.compose.animation.fadeIn
import androidx.compose.animation.fadeOut
import androidx.compose.animation.slideInHorizontally
import androidx.compose.animation.slideOutHorizontally
import androidx.navigation.NavBackStackEntry

private const val ANIMATION_DURATION = 300

// Horizontal slide animations (default)
fun AnimatedContentTransitionScope<NavBackStackEntry>.slideInFromRight(): EnterTransition {
    return slideInHorizontally(
        initialOffsetX = { it },
        animationSpec = tween(ANIMATION_DURATION)
    ) + fadeIn(animationSpec = tween(ANIMATION_DURATION))
}

fun AnimatedContentTransitionScope<NavBackStackEntry>.slideOutToLeft(): ExitTransition {
    return slideOutHorizontally(
        targetOffsetX = { -it },
        animationSpec = tween(ANIMATION_DURATION)
    ) + fadeOut(animationSpec = tween(ANIMATION_DURATION))
}

fun AnimatedContentTransitionScope<NavBackStackEntry>.slideInFromLeft(): EnterTransition {
    return slideInHorizontally(
        initialOffsetX = { -it },
        animationSpec = tween(ANIMATION_DURATION)
    ) + fadeIn(animationSpec = tween(ANIMATION_DURATION))
}

fun AnimatedContentTransitionScope<NavBackStackEntry>.slideOutToRight(): ExitTransition {
    return slideOutHorizontally(
        targetOffsetX = { it },
        animationSpec = tween(ANIMATION_DURATION)
    ) + fadeOut(animationSpec = tween(ANIMATION_DURATION))
}
```

### Navigation Extensions
```kotlin
// core/navigation/src/main/java/com/myproject/core/navigation/utils/NavigationExtensions.kt
package com.myproject.core.navigation.utils

import androidx.compose.runtime.Composable
import androidx.navigation.NavGraphBuilder
import androidx.navigation.compose.composable

// Standard horizontal slide animation
inline fun <reified T : Any> NavGraphBuilder.animatedComposable(
    noinline content: @Composable (T) -> Unit
) {
    composable<T>(
        enterTransition = { slideInFromRight() },
        exitTransition = { slideOutToLeft() },
        popEnterTransition = { slideInFromLeft() },
        popExitTransition = { slideOutToRight() }
    ) { backStackEntry ->
        content(backStackEntry.toRoute<T>())
    }
}

// Fade animation for modals
inline fun <reified T : Any> NavGraphBuilder.fadeComposable(
    noinline content: @Composable (T) -> Unit
) {
    composable<T>(
        enterTransition = { fadeIn() },
        exitTransition = { fadeOut() },
        popEnterTransition = { fadeIn() },
        popExitTransition = { fadeOut() }
    ) { backStackEntry ->
        content(backStackEntry.toRoute<T>())
    }
}
```

### Navigation Host
```kotlin
// core/navigation/src/main/java/com/myproject/core/navigation/NavigationHost.kt
package com.myproject.core.navigation

import androidx.compose.runtime.Composable
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.rememberNavController
import com.myproject.core.navigation.routes.HomeRoute
import com.myproject.core.navigation.routes.RootRoute

@Composable
fun NavigationHost() {
    val navController = rememberNavController()

    NavHost(
        route = RootRoute::class,
        navController = navController,
        startDestination = HomeRoute
    ) {
        // Add your navigation graphs here
        // homeGraph(navController)
        // profileGraph(navController)
        // settingsGraph(navController)
    }
}
```

### Navigation Module
```kotlin
// core/navigation/src/main/java/com/myproject/core/navigation/di/NavigationModule.kt
package com.myproject.core.navigation.di

import com.google.gson.Gson
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object NavigationModule {
    
    @Provides
    @Singleton
    fun provideGson(): Gson = Gson()
}
```

## 5. Manifests for Core Modules

### Core Database Manifest
```xml
<!-- core/database/src/main/AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android" />
```

### Core Network Manifest
```xml
<!-- core/network/src/main/AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
</manifest>
```

### Core Navigation Manifest
```xml
<!-- core/navigation/src/main/AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android" />
```

## Usage Instructions

1. **Create Directory Structure**: Set up the directories as shown
2. **Copy Build Files**: Use the build.gradle.kts templates
3. **Implement Core Classes**: Add the provided implementations
4. **Update Settings**: Add modules to settings.gradle.kts
5. **Configure Dependencies**: Link modules in your features

These core modules provide the essential infrastructure for your Android project and can be extended as needed for specific requirements.