# Navigation & Screens Architecture Guide

This document defines the standardized approach for creating navigation graphs, routes, navigators, and screens in our Android Compose project, based on analysis of the Sunshine-Photos architecture.

## Core Navigation Principles

1. **Type-Safe Navigation**: Use Kotlin Serialization for compile-time route validation
2. **Feature-Based Graphs**: Organize navigation by feature domains
3. **Interface-Driven Navigators**: Abstract navigation logic through interfaces
4. **Complex Parameter Handling**: Support both simple and complex object parameters
5. **Consistent Animations**: Standardized screen transitions

## Project Structure

```
navigation/
├── app/                           # Main navigation module
│   ├── src/main/java/
│   │   └── com/project/navigation/
│   │       ├── NavigationHost.kt  # Root navigation host
│   │       ├── RootGraphRoute.kt  # Root graph definition
│   │       ├── utils/
│   │       │   └── RouteAnimations.kt
│   │       └── [feature]/         # Feature-specific navigation
│   │           ├── FeatureGraph.kt
│   │           ├── FeatureNavigator.kt
│   │           └── routes/
│   │               ├── FeatureGraphRoute.kt
│   │               └── FeatureRoute.kt
│   └── build.gradle.kts
└── feature/
    └── [feature]/
        └── view/
            ├── src/main/java/
            │   └── com/project/feature/view/
            │       ├── FeatureScreen.kt
            │       └── IFeatureNavigator.kt
            └── build.gradle.kts
```

## 1. Route Definitions

### Simple Routes (No Parameters)
```kotlin
// navigation/home/routes/HomeRoute.kt
package com.project.navigation.home.routes

import kotlinx.serialization.Serializable

@Serializable 
data object HomeRoute

@Serializable 
data object HomeGraphRoute
```

### Parameterized Routes
```kotlin
// navigation/detail/routes/DetailRoute.kt
package com.project.navigation.detail.routes

import kotlinx.serialization.Serializable

@Serializable
data class ItemDetailRoute(
    val itemId: String,
    val comingFromCreation: Boolean? = null,
    val selectedFilter: String? = null,
)

@Serializable 
data object DetailGraphRoute
```

### Complex JSON Routes
```kotlin
// navigation/share/routes/ShareRoute.kt
package com.project.navigation.share.routes

import kotlinx.serialization.Serializable

@Serializable
data class ShareItemsRoute(
    val selectedItems: String,        // JSON serialized List<Item>
    val preSelectedFilters: String,   // JSON serialized Set<String>
    val metadata: String? = null,     // JSON serialized Metadata object
)

@Serializable 
data object ShareGraphRoute
```

## 2. Navigation Graph Structure

### Root Navigation Host
```kotlin
// navigation/NavigationHost.kt
package com.project.navigation

import androidx.compose.runtime.Composable
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.rememberNavController
import com.project.domain.ToJson

@Composable
fun NavigationHost(toJson: ToJson, initialRoute: InitialRoute) {
    val navController = rememberNavController()

    NavHost(
        route = RootGraphRoute::class,
        navController = navController,
        startDestination = when (initialRoute) {
            InitialRoute.Login -> LoginGraphRoute
            InitialRoute.Home -> HomeGraphRoute
            is InitialRoute.Detail -> DetailGraphRoute
        },
    ) {
        // Feature graphs
        loginGraph(navController)
        homeGraph(navController, toJson)
        detailGraph(navController, toJson)
        profileGraph(navController)
        settingsGraph(navController, toJson)
    }
}
```

### Feature Graph Implementation
```kotlin
// navigation/home/HomeGraph.kt
package com.project.navigation.home

import androidx.navigation.NavController
import androidx.navigation.NavGraphBuilder
import androidx.navigation.compose.navigation
import com.project.domain.ToJson
import com.project.navigation.home.routes.HomeGraphRoute
import com.project.navigation.home.routes.HomeRoute
import com.project.navigation.utils.animatedScreen
import com.project.feature.home.view.HomeScreen

fun NavGraphBuilder.homeGraph(navController: NavController, toJson: ToJson) {
    val navigator = HomeNavigator(navController, toJson)

    navigation<HomeGraphRoute>(startDestination = HomeRoute) {
        animatedScreen<HomeRoute> { 
            HomeScreen(navigator = navigator) 
        }
        
        // Add related screens within the same graph
        animatedScreen<HomeSettingsRoute> { 
            HomeSettingsScreen(navigator = navigator) 
        }
    }
}
```

## 3. Navigator Pattern

### Navigator Interface
```kotlin
// feature/home/view/IHomeNavigator.kt
package com.project.feature.home.view

import com.project.models.Item
import com.project.models.Filter

interface IHomeNavigator {
    fun navigateUp()
    fun navigateToDetail(itemId: String, filter: String? = null)
    fun navigateToSettings()
    fun navigateToShare(
        selectedItems: List<Item>,
        preSelectedFilters: Set<String> = emptySet()
    )
    fun navigateToProfile()
}
```

### Navigator Implementation
```kotlin
// navigation/home/HomeNavigator.kt
package com.project.navigation.home

import androidx.navigation.NavController
import com.project.domain.ToJson
import com.project.navigation.detail.routes.ItemDetailRoute
import com.project.navigation.share.routes.ShareItemsRoute
import com.project.navigation.profile.routes.ProfileRoute
import com.project.navigation.settings.routes.SettingsRoute
import com.project.feature.home.view.IHomeNavigator
import com.project.models.Item

class HomeNavigator(
    private val navController: NavController, 
    private val toJson: ToJson
) : IHomeNavigator {

    override fun navigateUp() {
        navController.navigateUp()
    }

    override fun navigateToDetail(itemId: String, filter: String?) {
        navController.navigate(
            ItemDetailRoute(
                itemId = itemId,
                selectedFilter = filter
            )
        )
    }

    override fun navigateToSettings() {
        navController.navigate(SettingsRoute)
    }

    override fun navigateToShare(
        selectedItems: List<Item>,
        preSelectedFilters: Set<String>
    ) {
        navController.navigate(
            ShareItemsRoute(
                selectedItems = toJson(selectedItems),
                preSelectedFilters = toJson(preSelectedFilters)
            )
        )
    }

    override fun navigateToProfile() {
        navController.navigate(ProfileRoute)
    }
}
```

## 4. Screen Architecture

### Basic Screen Structure
```kotlin
// feature/home/view/HomeScreen.kt
package com.project.feature.home.view

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

@Composable
fun HomeScreen(
    navigator: IHomeNavigator,
    viewModel: HomeViewModel = hiltViewModel(),
    // Additional feature-specific ViewModels
    itemsViewModel: ItemsViewModel = hiltViewModel(),
    notificationsViewModel: NotificationsViewModel = hiltViewModel(),
) {
    val context = LocalContext.current
    val state by viewModel.state.collectAsStateWithLifecycle()
    val items by itemsViewModel.items.collectAsStateWithLifecycle()

    // Event handling
    LaunchedEffect(context) {
        viewModel.events.collect { event ->
            when (event) {
                is HomeViewModel.Event.NavigateToDetail -> {
                    navigator.navigateToDetail(
                        itemId = event.itemId,
                        filter = event.filter
                    )
                }
                is HomeViewModel.Event.ShowError -> {
                    // Handle error display
                }
            }
        }
    }

    LaunchedEffect(context) {
        itemsViewModel.events.collect { event ->
            when (event) {
                is ItemsViewModel.Event.NavigateToShare -> {
                    navigator.navigateToShare(
                        selectedItems = event.items,
                        preSelectedFilters = event.filters
                    )
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
            items = items,
            onItemClick = viewModel::selectItem,
            onRefresh = viewModel::refresh,
            onSettingsClick = navigator::navigateToSettings
        )
    }
}
```

### Complex Screen with Multiple ViewModels
```kotlin
@Composable
fun DetailScreen(
    navigator: IDetailNavigator,
    viewModel: DetailViewModel = hiltViewModel(),
    commentsViewModel: CommentsViewModel = hiltViewModel(),
    mediaViewModel: MediaViewModel = hiltViewModel(),
    shareViewModel: ShareViewModel = hiltViewModel(),
) {
    val context = LocalContext.current
    val state by viewModel.state.collectAsStateWithLifecycle()
    val comments by commentsViewModel.comments.collectAsStateWithLifecycle()
    val media by mediaViewModel.media.collectAsStateWithLifecycle()

    // Permission handling
    val permissionLauncher = rememberLauncherForActivityResult(
        ActivityResultContracts.RequestPermission()
    ) { isGranted ->
        mediaViewModel.handlePermissionResult(isGranted)
    }

    // Multiple event handling
    LaunchedEffect(context) {
        viewModel.events.collect { event ->
            when (event) {
                is DetailViewModel.Event.NavigateBack -> navigator.navigateUp()
                is DetailViewModel.Event.ShareItem -> {
                    navigator.navigateToShare(event.item)
                }
            }
        }
    }

    LaunchedEffect(context) {
        mediaViewModel.events.collect { event ->
            when (event) {
                is MediaViewModel.Event.RequestPermission -> {
                    permissionLauncher.launch(event.permission)
                }
                is MediaViewModel.Event.ShowError -> {
                    // Handle media errors
                }
            }
        }
    }

    Scaffold(
        topBar = {
            DetailTopBar(
                title = state.item?.title.orEmpty(),
                onNavigateUp = navigator::navigateUp
            )
        }
    ) { innerPadding ->
        DetailContent(
            modifier = Modifier.padding(innerPadding),
            state = state,
            comments = comments,
            media = media,
            onAddComment = commentsViewModel::addComment,
            onSelectMedia = mediaViewModel::selectMedia
        )
    }
}
```

## 5. Route Animations

### Animation Utilities
```kotlin
// navigation/utils/RouteAnimations.kt
package com.project.navigation.utils

import androidx.compose.animation.AnimatedContentTransitionScope
import androidx.compose.animation.EnterTransition
import androidx.compose.animation.ExitTransition
import androidx.compose.animation.core.tween
import androidx.compose.animation.fadeIn
import androidx.compose.animation.fadeOut
import androidx.compose.animation.slideInHorizontally
import androidx.compose.animation.slideOutHorizontally
import androidx.compose.animation.slideInVertically
import androidx.compose.animation.slideOutVertically
import androidx.navigation.NavBackStackEntry

// Horizontal slide animations (default)
fun AnimatedContentTransitionScope<NavBackStackEntry>.slideInFromRight(): EnterTransition {
    return slideInHorizontally(
        initialOffsetX = { it },
        animationSpec = tween(300)
    ) + fadeIn(animationSpec = tween(300))
}

fun AnimatedContentTransitionScope<NavBackStackEntry>.slideOutToLeft(): ExitTransition {
    return slideOutHorizontally(
        targetOffsetX = { -it },
        animationSpec = tween(300)
    ) + fadeOut(animationSpec = tween(300))
}

// Vertical slide animations (for modals/bottom sheets)
fun AnimatedContentTransitionScope<NavBackStackEntry>.slideInFromBottom(): EnterTransition {
    return slideInVertically(
        initialOffsetY = { it },
        animationSpec = tween(300)
    ) + fadeIn(animationSpec = tween(300))
}

fun AnimatedContentTransitionScope<NavBackStackEntry>.slideOutToBottom(): ExitTransition {
    return slideOutVertically(
        targetOffsetY = { it },
        animationSpec = tween(300)
    ) + fadeOut(animationSpec = tween(300))
}
```

### Animated Screen Extensions
```kotlin
// navigation/utils/NavGraphBuilderExtensions.kt
package com.project.navigation.utils

import androidx.compose.runtime.Composable
import androidx.navigation.NavGraphBuilder
import androidx.navigation.compose.composable
import androidx.navigation.toRoute

// Standard horizontal slide animation
inline fun <reified T : Any> NavGraphBuilder.animatedScreen(
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

// Bottom slide animation for modals
inline fun <reified T : Any> NavGraphBuilder.bottomAnimatedScreen(
    noinline content: @Composable (T) -> Unit
) {
    composable<T>(
        enterTransition = { slideInFromBottom() },
        exitTransition = { slideOutToBottom() },
        popEnterTransition = { slideInFromBottom() },
        popExitTransition = { slideOutToBottom() }
    ) { backStackEntry ->
        content(backStackEntry.toRoute<T>())
    }
}
```

## 6. Parameter Extraction & Type Safety

### Route Parameter Extraction
```kotlin
// In your graph definition
animatedScreen<ItemDetailRoute> { route ->
    ItemDetailScreen(
        navigator = navigator,
        itemId = route.itemId,
        filter = route.selectedFilter
    )
}

// Or extract in the screen composable
@Composable
fun ItemDetailScreen(navigator: IDetailNavigator) {
    val route = LocalNavBackStackEntry.current.toRoute<ItemDetailRoute>()
    
    // Use route.itemId, route.selectedFilter, etc.
}
```

### Complex Parameter Handling
```kotlin
// In Navigator
fun navigateToShare(items: List<Item>, filters: Set<String>) {
    navController.navigate(
        ShareItemsRoute(
            selectedItems = toJson(items),
            preSelectedFilters = toJson(filters)
        )
    )
}

// In Screen
@Composable
fun ShareItemsScreen(navigator: IShareNavigator) {
    val route = LocalNavBackStackEntry.current.toRoute<ShareItemsRoute>()
    val gson = remember { Gson() }
    
    val items = remember(route.selectedItems) {
        gson.fromJson<List<Item>>(route.selectedItems)
    }
    
    val filters = remember(route.preSelectedFilters) {
        gson.fromJson<Set<String>>(route.preSelectedFilters)
    }
}
```

## 7. Testing Navigation

### Navigator Testing
```kotlin
class HomeNavigatorTest {
    
    @MockK lateinit var navController: NavController
    @MockK lateinit var toJson: ToJson
    
    private lateinit var navigator: HomeNavigator
    
    @Before
    fun setup() {
        navigator = HomeNavigator(navController, toJson)
    }
    
    @Test
    fun `navigateToDetail should navigate with correct route`() {
        // Given
        val itemId = "123"
        val filter = "category"
        
        // When
        navigator.navigateToDetail(itemId, filter)
        
        // Then
        verify { 
            navController.navigate(
                ItemDetailRoute(itemId = itemId, selectedFilter = filter)
            ) 
        }
    }
}
```

### Screen Testing
```kotlin
@HiltAndroidTest
class HomeScreenTest {
    
    @get:Rule val composeTestRule = createAndroidComposeRule<ComponentActivity>()
    @get:Rule val hiltRule = HiltAndroidRule(this)
    
    @Test
    fun homeScreen_displaysCorrectContent() {
        composeTestRule.setContent {
            HomeScreen(navigator = FakeHomeNavigator())
        }
        
        composeTestRule
            .onNodeWithText("Home")
            .assertIsDisplayed()
    }
}
```

## 8. Best Practices

### Do's ✅
- Use `@Serializable` for all route definitions
- Implement navigator interfaces for testability
- Handle complex objects through JSON serialization
- Use consistent animation patterns
- Collect ViewModel events in LaunchedEffect
- Extract routes to separate files for organization
- Use appropriate coroutine scopes for event collection

### Don'ts ❌
- Don't pass large objects directly in route parameters
- Don't forget to handle navigation events in screens
- Don't ignore animation consistency across the app
- Don't create circular navigation dependencies
- Don't forget to inject ToJson where needed
- Don't hardcode route strings - use typed routes

## 9. Dependencies Setup

### Navigation Module build.gradle.kts
```kotlin
dependencies {
    implementation(libs.androidx.navigation.compose)
    implementation(libs.kotlinx.serialization.json)
    implementation(libs.hilt.navigation.compose)
    
    // Feature modules
    implementation(project(":feature:home:view"))
    implementation(project(":feature:detail:view"))
    
    // Domain
    implementation(project(":domain"))
}
```

This architecture ensures type-safe, testable, and maintainable navigation across the entire application while providing flexibility for complex parameter passing and consistent user experience through standardized animations.