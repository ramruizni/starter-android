# Dependency Injection Setup Guide

This guide provides comprehensive setup for Hilt dependency injection across all modules, including database configuration, network setup, repository bindings, and use case provisioning.

## Dependency Injection Overview

### Architecture Pattern:
1. **Application Module** - Root Hilt setup
2. **Core Modules** - Infrastructure DI (Database, Network)
3. **Data Modules** - Repository and DataSource bindings
4. **Domain Modules** - Use case provisioning
5. **Feature Modules** - ViewModel injection

## 1. Application-Level DI Setup

### Application Class with Hilt
```kotlin
// app/src/main/java/com/myproject/MyApplication.kt
package com.myproject

import android.app.Application
import dagger.hilt.android.HiltAndroidApp

@HiltAndroidApp
class MyApplication : Application() {
    
    override fun onCreate() {
        super.onCreate()
        
        // Initialize any global configurations here
        setupLogging()
        setupCrashReporting()
    }
    
    private fun setupLogging() {
        // Set up logging configuration
        // Example: Timber.plant(Timber.DebugTree())
    }
    
    private fun setupCrashReporting() {
        // Set up crash reporting
        // Example: Firebase Crashlytics initialization
    }
}
```

### Main Activity with Hilt
```kotlin
// app/src/main/java/com/myproject/MainActivity.kt
package com.myproject

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import dagger.hilt.android.AndroidEntryPoint
import com.myproject.core.navigation.NavigationHost
import com.myproject.ui.design.theme.MyAppTheme

@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        setContent {
            MyAppTheme {
                NavigationHost()
            }
        }
    }
}
```

## 2. Core Database DI Module

### Database Module
```kotlin
// core/database/src/main/java/com/myproject/core/database/di/DatabaseModule.kt
package com.myproject.core.database.di

import android.content.Context
import androidx.room.Room
import com.myproject.core.database.MyAppDatabase
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
    // Example:
    // @Provides
    // fun provideExampleDao(database: MyAppDatabase): ExampleDao = database.exampleDao()
}
```

## 3. Core Network DI Module

### Network Module
```kotlin
// core/network/src/main/java/com/myproject/core/network/di/NetworkModule.kt
package com.myproject.core.network.di

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
            .setDateFormat("yyyy-MM-dd HH:mm:ss")
            .setLenient()
            .create()
    }
    
    @Provides
    @Singleton
    fun provideLoggingInterceptor(): HttpLoggingInterceptor {
        return HttpLoggingInterceptor().apply {
            level = if (BuildConfig.DEBUG) {
                HttpLoggingInterceptor.Level.BODY
            } else {
                HttpLoggingInterceptor.Level.NONE
            }
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
            .retryOnConnectionFailure(true)
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

### Auth Interceptor with DI
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
        
        val requestBuilder = originalRequest.newBuilder()
            .addHeader("Content-Type", "application/json")
            .addHeader("Accept", "application/json")
        
        // Add authentication header if token is available
        getAuthToken()?.let { token ->
            requestBuilder.addHeader("Authorization", "Bearer $token")
        }
        
        return chain.proceed(requestBuilder.build())
    }
    
    private fun getAuthToken(): String? {
        // Implement token retrieval logic
        // return tokenProvider.getToken()
        return null
    }
}
```

## 4. Data Layer DI Setup

### Repository DI Module Template
```kotlin
// data/[feature]/src/main/java/com/myproject/data/[feature]/di/[Feature]DataModule.kt
package com.myproject.data.home.di

import com.myproject.data.home.HomeRepository
import com.myproject.data.home.local.HomeDao
import com.myproject.data.home.remote.HomeApiService
import com.myproject.core.database.MyAppDatabase
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent
import retrofit2.Retrofit
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object HomeDataModule {
    
    @Provides
    fun provideHomeDao(database: MyAppDatabase): HomeDao {
        return database.homeDao()
    }
    
    @Provides
    @Singleton
    fun provideHomeApiService(retrofit: Retrofit): HomeApiService {
        return retrofit.create(HomeApiService::class.java)
    }
    
    @Provides
    @Singleton
    fun provideHomeRepository(
        apiService: HomeApiService,
        dao: HomeDao
    ): HomeRepository {
        return HomeRepository(apiService, dao)
    }
}
```

### API Service Creation
```kotlin
// data/[feature]/src/main/java/com/myproject/data/[feature]/remote/[Feature]ApiService.kt
package com.myproject.data.home.remote

import com.myproject.data.home.remote.dto.HomeItemDto
import retrofit2.http.GET
import retrofit2.http.POST
import retrofit2.http.Path
import retrofit2.http.Body
import retrofit2.http.Query

interface HomeApiService {
    
    @GET("home/items")
    suspend fun getHomeItems(
        @Query("userId") userId: String? = null,
        @Query("limit") limit: Int = 20,
        @Query("offset") offset: Int = 0
    ): List<HomeItemDto>
    
    @POST("home/items")
    suspend fun createHomeItem(
        @Body item: HomeItemDto
    ): HomeItemDto
    
    @GET("home/items/{id}")
    suspend fun getHomeItem(
        @Path("id") itemId: String
    ): HomeItemDto
}
```

### DAO with Room Integration
```kotlin
// data/[feature]/src/main/java/com/myproject/data/[feature]/local/[Feature]Dao.kt
package com.myproject.data.home.local

import androidx.room.Dao
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query
import androidx.room.Update
import androidx.room.Delete
import com.myproject.data.home.local.entities.HomeItemEntity
import kotlinx.coroutines.flow.Flow

@Dao
interface HomeDao {
    
    @Query("SELECT * FROM home_items ORDER BY created_at DESC")
    fun observeAll(): Flow<List<HomeItemEntity>>
    
    @Query("SELECT * FROM home_items ORDER BY created_at DESC LIMIT :limit OFFSET :offset")
    suspend fun getAll(limit: Int, offset: Int): List<HomeItemEntity>
    
    @Query("SELECT * FROM home_items WHERE id = :id")
    suspend fun getById(id: String): HomeItemEntity?
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(item: HomeItemEntity)
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertAll(items: List<HomeItemEntity>)
    
    @Update
    suspend fun update(item: HomeItemEntity)
    
    @Delete
    suspend fun delete(item: HomeItemEntity)
    
    @Query("DELETE FROM home_items WHERE id = :id")
    suspend fun deleteById(id: String)
    
    @Query("DELETE FROM home_items")
    suspend fun deleteAll()
}
```

### Repository Implementation with DI
```kotlin
// data/[feature]/src/main/java/com/myproject/data/[feature]/[Feature]Repository.kt
package com.myproject.data.home

import com.myproject.data.home.local.HomeDao
import com.myproject.data.home.remote.HomeApiService
import com.myproject.domain.home.models.HomeItem
import com.myproject.data.home.mappers.toDomain
import com.myproject.data.home.mappers.toEntity
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class HomeRepository @Inject constructor(
    private val apiService: HomeApiService,
    private val dao: HomeDao,
) {
    
    suspend fun getHomeItems(
        userId: String? = null,
        limit: Int = 20,
        offset: Int = 0
    ): List<HomeItem> {
        return try {
            // Try remote first
            val remoteItems = apiService.getHomeItems(userId, limit, offset)
            
            // Cache locally
            dao.insertAll(remoteItems.map { it.toEntity() })
            
            remoteItems.map { it.toDomain() }
        } catch (e: Exception) {
            // Fallback to local cache
            dao.getAll(limit, offset).map { it.toDomain() }
        }
    }
    
    fun observeHomeItems(): Flow<List<HomeItem>> {
        return dao.observeAll().map { entities ->
            entities.map { it.toDomain() }
        }
    }
    
    suspend fun createHomeItem(item: HomeItem): HomeItem {
        val dto = item.toDto()
        val createdItem = apiService.createHomeItem(dto)
        dao.insert(createdItem.toEntity())
        return createdItem.toDomain()
    }
}
```

## 5. Domain Layer DI Setup

### Use Cases DI Module
```kotlin
// domain/[feature]/usecases/src/main/java/com/myproject/domain/[feature]/usecases/di/[Feature]UseCasesModule.kt
package com.myproject.domain.home.usecases.di

import com.myproject.data.home.HomeRepository
import com.myproject.domain.home.usecases.GetHomeItemsUseCase
import com.myproject.domain.home.usecases.CreateHomeItemUseCase
import com.myproject.domain.home.usecases.ObserveHomeItemsUseCase
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent

@Module
@InstallIn(SingletonComponent::class)
object HomeUseCasesModule {
    
    @Provides
    fun provideGetHomeItemsUseCase(
        repository: HomeRepository
    ): GetHomeItemsUseCase {
        return GetHomeItemsUseCase(repository)
    }
    
    @Provides
    fun provideCreateHomeItemUseCase(
        repository: HomeRepository
    ): CreateHomeItemUseCase {
        return CreateHomeItemUseCase(repository)
    }
    
    @Provides
    fun provideObserveHomeItemsUseCase(
        repository: HomeRepository
    ): ObserveHomeItemsUseCase {
        return ObserveHomeItemsUseCase(repository)
    }
}
```

### Use Case Implementations
```kotlin
// domain/[feature]/usecases/src/main/java/com/myproject/domain/[feature]/usecases/GetHomeItemsUseCase.kt
package com.myproject.domain.home.usecases

import com.myproject.domain.home.models.HomeItem
import com.myproject.data.home.HomeRepository
import javax.inject.Inject

class GetHomeItemsUseCase @Inject constructor(
    private val repository: HomeRepository
) {
    suspend operator fun invoke(
        userId: String? = null,
        limit: Int = 20,
        offset: Int = 0
    ): List<HomeItem> {
        return repository.getHomeItems(userId, limit, offset)
    }
}

class CreateHomeItemUseCase @Inject constructor(
    private val repository: HomeRepository
) {
    suspend operator fun invoke(item: HomeItem): HomeItem {
        return repository.createHomeItem(item)
    }
}

class ObserveHomeItemsUseCase @Inject constructor(
    private val repository: HomeRepository
) {
    operator fun invoke(): Flow<List<HomeItem>> {
        return repository.observeHomeItems()
    }
}
```

## 6. Feature Layer DI Setup

### ViewModel with Hilt
```kotlin
// features/[feature]/viewmodel/src/main/java/com/myproject/features/[feature]/viewmodel/[Feature]ViewModel.kt
package com.myproject.features.home.viewmodel

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
import com.myproject.domain.home.usecases.GetHomeItemsUseCase
import com.myproject.domain.home.usecases.CreateHomeItemUseCase
import com.myproject.domain.home.usecases.ObserveHomeItemsUseCase
import com.myproject.domain.home.models.HomeItem
import javax.inject.Inject

@HiltViewModel
class HomeViewModel @Inject constructor(
    private val getHomeItemsUseCase: GetHomeItemsUseCase,
    private val createHomeItemUseCase: CreateHomeItemUseCase,
    private val observeHomeItemsUseCase: ObserveHomeItemsUseCase,
) : ViewModel() {

    private val _state = MutableStateFlow(State())
    val state = _state.asStateFlow()

    private val _events = MutableSharedFlow<Event>()
    val events = _events.asSharedFlow()

    init {
        observeItems()
        loadItems()
    }

    private fun observeItems() {
        viewModelScope.launch {
            observeHomeItemsUseCase().collect { items ->
                _state.update { it.copy(items = items) }
            }
        }
    }

    fun handleAction(action: Action) {
        when (action) {
            is Action.LoadItems -> loadItems()
            is Action.CreateItem -> createItem(action.title, action.description)
            is Action.Refresh -> refresh()
        }
    }

    private fun loadItems() {
        viewModelScope.launch(Dispatchers.IO) {
            _state.update { it.copy(isLoading = true) }
            
            try {
                getHomeItemsUseCase()
                _state.update { it.copy(isLoading = false, hasError = false) }
            } catch (error: Exception) {
                handleError(error)
            }
        }
    }

    private fun createItem(title: String, description: String) {
        viewModelScope.launch(Dispatchers.IO) {
            try {
                val newItem = HomeItem(
                    id = "", // Will be assigned by backend
                    title = title,
                    description = description,
                    createdAt = System.currentTimeMillis(),
                    updatedAt = System.currentTimeMillis()
                )
                createHomeItemUseCase(newItem)
            } catch (error: Exception) {
                handleError(error)
            }
        }
    }

    private fun refresh() {
        loadItems()
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
        val items: List<HomeItem> = emptyList(),
        val isLoading: Boolean = false,
        val hasError: Boolean = false,
        val errorMessage: String? = null,
    )

    sealed interface Action {
        data object LoadItems : Action
        data class CreateItem(val title: String, val description: String) : Action
        data object Refresh : Action
    }

    sealed interface Event {
        data class ShowError(val message: String) : Event
    }
}
```

### Screen with Hilt ViewModel
```kotlin
// features/[feature]/view/src/main/java/com/myproject/features/[feature]/view/[Feature]Screen.kt
package com.myproject.features.home.view

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
import com.myproject.features.home.view.components.HomeContent
import com.myproject.features.home.viewmodel.HomeViewModel

@Composable
fun HomeScreen(
    navigator: IHomeNavigator,
    viewModel: HomeViewModel = hiltViewModel(), // Hilt automatically provides the ViewModel
) {
    val context = LocalContext.current
    val state by viewModel.state.collectAsStateWithLifecycle()

    // Event handling
    LaunchedEffect(context) {
        viewModel.events.collect { event ->
            when (event) {
                is HomeViewModel.Event.ShowError -> {
                    // Handle error display (e.g., show snackbar)
                }
            }
        }
    }

    Scaffold(
        modifier = Modifier.fillMaxSize()
    ) { innerPadding ->
        HomeContent(
            modifier = Modifier.padding(innerPadding),
            state = state,
            onAction = viewModel::handleAction
        )
    }
}
```

## 7. Testing DI Setup

### Test Application Class
```kotlin
// app/src/test/java/com/myproject/HiltTestApplication.kt
package com.myproject

import dagger.hilt.android.testing.HiltAndroidApp

@HiltAndroidApp
class HiltTestApplication : MyApplication()
```

### Test Runner
```kotlin
// app/src/androidTest/java/com/myproject/HiltTestRunner.kt
package com.myproject

import android.app.Application
import android.content.Context
import androidx.test.runner.AndroidJUnitRunner
import dagger.hilt.android.testing.HiltTestApplication

class HiltTestRunner : AndroidJUnitRunner() {
    override fun newApplication(
        cl: ClassLoader?,
        className: String?,
        context: Context?
    ): Application {
        return super.newApplication(cl, HiltTestApplication::class.java.name, context)
    }
}
```

### Example Integration Test
```kotlin
// features/[feature]/viewmodel/src/test/java/com/myproject/features/[feature]/viewmodel/[Feature]ViewModelTest.kt
package com.myproject.features.home.viewmodel

import androidx.arch.core.executor.testing.InstantTaskExecutorRule
import dagger.hilt.android.testing.HiltAndroidRule
import dagger.hilt.android.testing.HiltAndroidTest
import kotlinx.coroutines.ExperimentalCoroutinesApi
import kotlinx.coroutines.test.runTest
import org.junit.Before
import org.junit.Rule
import org.junit.Test
import javax.inject.Inject

@HiltAndroidTest
@ExperimentalCoroutinesApi
class HomeViewModelTest {

    @get:Rule
    var hiltRule = HiltAndroidRule(this)

    @get:Rule
    var instantTaskExecutorRule = InstantTaskExecutorRule()

    @Inject
    lateinit var viewModel: HomeViewModel

    @Before
    fun setup() {
        hiltRule.inject()
    }

    @Test
    fun `loadItems should update state correctly`() = runTest {
        // Given
        val initialState = viewModel.state.value

        // When
        viewModel.handleAction(HomeViewModel.Action.LoadItems)

        // Then
        // Add assertions here
    }
}
```

## 8. Build Configuration Updates

### Update build.gradle.kts files to include Hilt testing
```kotlin
// app/build.gradle.kts (add to existing dependencies)
dependencies {
    // ... existing dependencies
    
    // Hilt testing
    kspTest(libs.hilt.compiler)
    testImplementation(libs.hilt.android.testing)
    androidTestImplementation(libs.hilt.android.testing)
    kspAndroidTest(libs.hilt.compiler)
}
```

This comprehensive DI setup ensures proper dependency injection across all layers of your Android application while maintaining testability and scalability.