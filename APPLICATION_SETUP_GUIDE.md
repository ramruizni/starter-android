# Application Setup Guide

This guide provides complete setup for the main application module and UI foundation modules, including theming, components, and the main entry point.

## Application Module Overview

### Main Components:
1. **Application Class** - Hilt entry point
2. **MainActivity** - Main activity with Compose
3. **Navigation Integration** - Root navigation setup
4. **Theme System** - Material Design 3 theming
5. **UI Foundation** - Design system and components

## 1. Application Module (:app)

### Directory Structure
```
app/
├── src/
│   ├── main/
│   │   ├── java/com/myproject/
│   │   │   ├── MyApplication.kt
│   │   │   └── MainActivity.kt
│   │   ├── res/
│   │   │   ├── values/
│   │   │   │   ├── strings.xml
│   │   │   │   ├── colors.xml
│   │   │   │   └── themes.xml
│   │   │   ├── mipmap-*/
│   │   │   │   └── ic_launcher.png
│   │   │   └── xml/
│   │   │       ├── backup_rules.xml
│   │   │       └── data_extraction_rules.xml
│   │   └── AndroidManifest.xml
│   ├── test/
│   │   └── java/com/myproject/
│   │       └── ExampleUnitTest.kt
│   └── androidTest/
│       └── java/com/myproject/
│           └── ExampleInstrumentedTest.kt
├── proguard-rules.pro
└── build.gradle.kts
```

### Build Configuration
```kotlin
// app/build.gradle.kts
plugins {
    alias(libs.plugins.project.android.application)
    alias(libs.plugins.project.android.application.compose)
    alias(libs.plugins.project.android.hilt)
}

dependencies {
    // Core modules
    implementation(project(":core:database"))
    implementation(project(":core:network"))
    implementation(project(":core:navigation"))
    implementation(project(":core:domain"))
    
    // UI modules
    implementation(project(":ui:design-system"))
    implementation(project(":ui:components"))
    
    // Feature modules (add as needed)
    implementation(project(":features:home:view"))
    
    // Additional dependencies
    implementation(libs.androidx.core.splashscreen)
    implementation(libs.coil)
    
    // Testing
    testImplementation(libs.junit5.api)
    androidTestImplementation(libs.androidx.junit)
    androidTestImplementation(libs.androidx.espresso.core)
    androidTestImplementation(libs.androidx.ui.test.junit4)
}
```

### Application Class
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

### MainActivity
```kotlin
// app/src/main/java/com/myproject/MainActivity.kt
package com.myproject

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.ui.Modifier
import androidx.core.splashscreen.SplashScreen.Companion.installSplashScreen
import dagger.hilt.android.AndroidEntryPoint
import com.myproject.core.navigation.NavigationHost
import com.myproject.ui.design.theme.MyAppTheme

@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        // Install splash screen
        val splashScreen = installSplashScreen()
        
        super.onCreate(savedInstanceState)
        
        // Configure splash screen
        configureSplashScreen(splashScreen)
        
        // Enable edge-to-edge
        enableEdgeToEdge()
        
        setContent {
            MyAppTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    NavigationHost()
                }
            }
        }
    }
    
    private fun configureSplashScreen(splashScreen: SplashScreen) {
        // Keep splash screen visible until data is ready
        splashScreen.setKeepOnScreenCondition {
            // Return true to keep splash screen visible
            // Return false to dismiss splash screen
            false
        }
    }
}
```

### Android Manifest
```xml
<!-- app/src/main/AndroidManifest.xml -->
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

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

### Resource Files

#### Strings
```xml
<!-- app/src/main/res/values/strings.xml -->
<resources>
    <string name="app_name">My App</string>
    
    <!-- Common strings -->
    <string name="ok">OK</string>
    <string name="cancel">Cancel</string>
    <string name="retry">Retry</string>
    <string name="loading">Loading…</string>
    <string name="error_generic">Something went wrong</string>
    <string name="error_network">Network error. Please check your connection.</string>
    <string name="empty_state">No data available</string>
</resources>
```

#### Colors
```xml
<!-- app/src/main/res/values/colors.xml -->
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- Material Design 3 color tokens will be defined in design-system module -->
    <!-- These are fallback colors for non-Compose usage -->
    <color name="purple_200">#FFBB86FC</color>
    <color name="purple_500">#FF6200EE</color>
    <color name="purple_700">#FF3700B3</color>
    <color name="teal_200">#FF03DAC5</color>
    <color name="teal_700">#FF018786</color>
    <color name="black">#FF000000</color>
    <color name="white">#FFFFFFFF</color>
</resources>
```

#### Themes
```xml
<!-- app/src/main/res/values/themes.xml -->
<resources xmlns:tools="http://schemas.android.com/tools">
    <!-- Base application theme. -->
    <style name="Theme.MyApp" parent="Theme.Material3.DayNight.NoActionBar">
        <!-- Customize your theme here. -->
    </style>
</resources>
```

#### Backup Rules
```xml
<!-- app/src/main/res/xml/backup_rules.xml -->
<?xml version="1.0" encoding="utf-8"?>
<full-backup-content>
    <!-- Exclude sensitive data from backup -->
    <exclude domain="sharedpref" path="secure_prefs.xml"/>
    <exclude domain="database" path="sensitive_data.db"/>
</full-backup-content>
```

#### Data Extraction Rules
```xml
<!-- app/src/main/res/xml/data_extraction_rules.xml -->
<?xml version="1.0" encoding="utf-8"?>
<data-extraction-rules>
    <cloud-backup>
        <exclude domain="sharedpref" path="secure_prefs.xml"/>
        <exclude domain="database" path="sensitive_data.db"/>
    </cloud-backup>
    <device-transfer>
        <exclude domain="sharedpref" path="secure_prefs.xml"/>
        <exclude domain="database" path="sensitive_data.db"/>
    </device-transfer>
</data-extraction-rules>
```

## 2. UI Design System Module (:ui:design-system)

### Directory Structure
```
ui/design-system/
├── src/
│   ├── main/
│   │   ├── java/com/myproject/ui/design/
│   │   │   ├── theme/
│   │   │   │   ├── Color.kt
│   │   │   │   ├── Theme.kt
│   │   │   │   ├─��─ Type.kt
│   │   │   │   └── Dimension.kt
│   │   │   ├── tokens/
│   │   │   │   ├── ColorTokens.kt
│   │   │   │   └── TypographyTokens.kt
│   │   │   └── utils/
│   │   │       └── ThemeUtils.kt
│   │   └── AndroidManifest.xml
│   └── test/
│       └── java/com/myproject/ui/design/
│           └── ThemeTest.kt
└── build.gradle.kts
```

### Build Configuration
```kotlin
// ui/design-system/build.gradle.kts
plugins {
    alias(libs.plugins.project.android.library)
    alias(libs.plugins.project.android.library.compose)
}

dependencies {
    implementation(libs.androidx.compose.bom)
    implementation(libs.androidx.material3)
    implementation(libs.androidx.ui.tooling.preview)
    
    debugImplementation(libs.androidx.ui.tooling)
}
```

### Color System
```kotlin
// ui/design-system/src/main/java/com/myproject/ui/design/theme/Color.kt
package com.myproject.ui.design.theme

import androidx.compose.material3.darkColorScheme
import androidx.compose.material3.lightColorScheme
import androidx.compose.ui.graphics.Color

// Light theme colors
val LightPrimary = Color(0xFF6750A4)
val LightOnPrimary = Color(0xFFFFFFFF)
val LightPrimaryContainer = Color(0xFFE9DDFF)
val LightOnPrimaryContainer = Color(0xFF22005D)

val LightSecondary = Color(0xFF625B71)
val LightOnSecondary = Color(0xFFFFFFFF)
val LightSecondaryContainer = Color(0xFFE8DEF8)
val LightOnSecondaryContainer = Color(0xFF1E192B)

val LightTertiary = Color(0xFF7E5260)
val LightOnTertiary = Color(0xFFFFFFFF)
val LightTertiaryContainer = Color(0xFFFFD9E3)
val LightOnTertiaryContainer = Color(0xFF30111D)

val LightError = Color(0xFFBA1A1A)
val LightOnError = Color(0xFFFFFFFF)
val LightErrorContainer = Color(0xFFFFDAD6)
val LightOnErrorContainer = Color(0xFF410002)

val LightBackground = Color(0xFFFFFBFF)
val LightOnBackground = Color(0xFF1C1B1E)
val LightSurface = Color(0xFFFFFBFF)
val LightOnSurface = Color(0xFF1C1B1E)

// Dark theme colors
val DarkPrimary = Color(0xFFCFBCFF)
val DarkOnPrimary = Color(0xFF381E72)
val DarkPrimaryContainer = Color(0xFF4F378A)
val DarkOnPrimaryContainer = Color(0xFFE9DDFF)

val DarkSecondary = Color(0xFFCBC2DB)
val DarkOnSecondary = Color(0xFF332D41)
val DarkSecondaryContainer = Color(0xFF4A4458)
val DarkOnSecondaryContainer = Color(0xFFE8DEF8)

val DarkTertiary = Color(0xFFEFB8C8)
val DarkOnTertiary = Color(0xFF4A2532)
val DarkTertiaryContainer = Color(0xFF633B48)
val DarkOnTertiaryContainer = Color(0xFFFFD9E3)

val DarkError = Color(0xFFFFB4AB)
val DarkOnError = Color(0xFF690005)
val DarkErrorContainer = Color(0xFF93000A)
val DarkOnErrorContainer = Color(0xFFFFDAD6)

val DarkBackground = Color(0xFF1C1B1E)
val DarkOnBackground = Color(0xFFE6E1E6)
val DarkSurface = Color(0xFF1C1B1E)
val DarkOnSurface = Color(0xFFE6E1E6)

val LightColorScheme = lightColorScheme(
    primary = LightPrimary,
    onPrimary = LightOnPrimary,
    primaryContainer = LightPrimaryContainer,
    onPrimaryContainer = LightOnPrimaryContainer,
    secondary = LightSecondary,
    onSecondary = LightOnSecondary,
    secondaryContainer = LightSecondaryContainer,
    onSecondaryContainer = LightOnSecondaryContainer,
    tertiary = LightTertiary,
    onTertiary = LightOnTertiary,
    tertiaryContainer = LightTertiaryContainer,
    onTertiaryContainer = LightOnTertiaryContainer,
    error = LightError,
    onError = LightOnError,
    errorContainer = LightErrorContainer,
    onErrorContainer = LightOnErrorContainer,
    background = LightBackground,
    onBackground = LightOnBackground,
    surface = LightSurface,
    onSurface = LightOnSurface,
)

val DarkColorScheme = darkColorScheme(
    primary = DarkPrimary,
    onPrimary = DarkOnPrimary,
    primaryContainer = DarkPrimaryContainer,
    onPrimaryContainer = DarkOnPrimaryContainer,
    secondary = DarkSecondary,
    onSecondary = DarkOnSecondary,
    secondaryContainer = DarkSecondaryContainer,
    onSecondaryContainer = DarkOnSecondaryContainer,
    tertiary = DarkTertiary,
    onTertiary = DarkOnTertiary,
    tertiaryContainer = DarkTertiaryContainer,
    onTertiaryContainer = DarkOnTertiaryContainer,
    error = DarkError,
    onError = DarkOnError,
    errorContainer = DarkErrorContainer,
    onErrorContainer = DarkOnErrorContainer,
    background = DarkBackground,
    onBackground = DarkOnBackground,
    surface = DarkSurface,
    onSurface = DarkOnSurface,
)
```

### Typography
```kotlin
// ui/design-system/src/main/java/com/myproject/ui/design/theme/Type.kt
package com.myproject.ui.design.theme

import androidx.compose.material3.Typography
import androidx.compose.ui.text.TextStyle
import androidx.compose.ui.text.font.Font
import androidx.compose.ui.text.font.FontFamily
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.sp

// Default font family (you can replace with custom fonts)
val DefaultFontFamily = FontFamily.Default

// Define custom typography
val MyAppTypography = Typography(
    displayLarge = TextStyle(
        fontFamily = DefaultFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 57.sp,
        lineHeight = 64.sp,
        letterSpacing = (-0.25).sp,
    ),
    displayMedium = TextStyle(
        fontFamily = DefaultFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 45.sp,
        lineHeight = 52.sp,
        letterSpacing = 0.sp,
    ),
    displaySmall = TextStyle(
        fontFamily = DefaultFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 36.sp,
        lineHeight = 44.sp,
        letterSpacing = 0.sp,
    ),
    headlineLarge = TextStyle(
        fontFamily = DefaultFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 32.sp,
        lineHeight = 40.sp,
        letterSpacing = 0.sp,
    ),
    headlineMedium = TextStyle(
        fontFamily = DefaultFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 28.sp,
        lineHeight = 36.sp,
        letterSpacing = 0.sp,
    ),
    headlineSmall = TextStyle(
        fontFamily = DefaultFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 24.sp,
        lineHeight = 32.sp,
        letterSpacing = 0.sp,
    ),
    titleLarge = TextStyle(
        fontFamily = DefaultFontFamily,
        fontWeight = FontWeight.Medium,
        fontSize = 22.sp,
        lineHeight = 28.sp,
        letterSpacing = 0.sp,
    ),
    titleMedium = TextStyle(
        fontFamily = DefaultFontFamily,
        fontWeight = FontWeight.Medium,
        fontSize = 16.sp,
        lineHeight = 24.sp,
        letterSpacing = 0.1.sp,
    ),
    titleSmall = TextStyle(
        fontFamily = DefaultFontFamily,
        fontWeight = FontWeight.Medium,
        fontSize = 14.sp,
        lineHeight = 20.sp,
        letterSpacing = 0.1.sp,
    ),
    bodyLarge = TextStyle(
        fontFamily = DefaultFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 16.sp,
        lineHeight = 24.sp,
        letterSpacing = 0.5.sp,
    ),
    bodyMedium = TextStyle(
        fontFamily = DefaultFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 14.sp,
        lineHeight = 20.sp,
        letterSpacing = 0.25.sp,
    ),
    bodySmall = TextStyle(
        fontFamily = DefaultFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 12.sp,
        lineHeight = 16.sp,
        letterSpacing = 0.4.sp,
    ),
    labelLarge = TextStyle(
        fontFamily = DefaultFontFamily,
        fontWeight = FontWeight.Medium,
        fontSize = 14.sp,
        lineHeight = 20.sp,
        letterSpacing = 0.1.sp,
    ),
    labelMedium = TextStyle(
        fontFamily = DefaultFontFamily,
        fontWeight = FontWeight.Medium,
        fontSize = 12.sp,
        lineHeight = 16.sp,
        letterSpacing = 0.5.sp,
    ),
    labelSmall = TextStyle(
        fontFamily = DefaultFontFamily,
        fontWeight = FontWeight.Medium,
        fontSize = 11.sp,
        lineHeight = 16.sp,
        letterSpacing = 0.5.sp,
    ),
)
```

### Dimensions
```kotlin
// ui/design-system/src/main/java/com/myproject/ui/design/theme/Dimension.kt
package com.myproject.ui.design.theme

import androidx.compose.ui.unit.dp

object Dimensions {
    // Spacing
    val spacing0 = 0.dp
    val spacing4 = 4.dp
    val spacing8 = 8.dp
    val spacing12 = 12.dp
    val spacing16 = 16.dp
    val spacing20 = 20.dp
    val spacing24 = 24.dp
    val spacing32 = 32.dp
    val spacing48 = 48.dp
    val spacing64 = 64.dp
    
    // Border radius
    val radiusSmall = 4.dp
    val radiusMedium = 8.dp
    val radiusLarge = 12.dp
    val radiusExtraLarge = 16.dp
    
    // Elevation
    val elevationSmall = 2.dp
    val elevationMedium = 4.dp
    val elevationLarge = 8.dp
    
    // Component sizes
    val buttonHeight = 48.dp
    val iconSize = 24.dp
    val iconSizeSmall = 16.dp
    val iconSizeLarge = 32.dp
    
    // Layout
    val minTouchTarget = 48.dp
}
```

### Theme
```kotlin
// ui/design-system/src/main/java/com/myproject/ui/design/theme/Theme.kt
package com.myproject.ui.design.theme

import android.app.Activity
import androidx.compose.foundation.isSystemInDarkTheme
import androidx.compose.material3.MaterialTheme
import androidx.compose.runtime.Composable
import androidx.compose.runtime.SideEffect
import androidx.compose.ui.graphics.toArgb
import androidx.compose.ui.platform.LocalView
import androidx.core.view.WindowCompat

@Composable
fun MyAppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }
    
    val view = LocalView.current
    if (!view.isInEditMode) {
        SideEffect {
            val window = (view.context as Activity).window
            window.statusBarColor = colorScheme.primary.toArgb()
            WindowCompat.getInsetsController(window, view).isAppearanceLightStatusBars = darkTheme
        }
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = MyAppTypography,
        content = content
    )
}
```

## 3. UI Components Module (:ui:components)

### Directory Structure
```
ui/components/
├── src/
│   ├── main/
│   │   ├── java/com/myproject/ui/components/
│   │   │   ├── buttons/
│   │   │   │   ├── PrimaryButton.kt
│   │   │   │   └── SecondaryButton.kt
│   │   │   ├── cards/
│   │   │   │   └── AppCard.kt
│   │   │   ├── loading/
│   │   │   │   └── LoadingIndicator.kt
│   │   │   ├── text/
│   │   │   │   └── AppTextField.kt
│   │   │   └── common/
│   │   │       ├── Spacer.kt
│   │   │       └── EmptyState.kt
│   │   └── AndroidManifest.xml
│   └── test/
│       └── java/com/myproject/ui/components/
│           └── ComponentTest.kt
└── build.gradle.kts
```

### Build Configuration
```kotlin
// ui/components/build.gradle.kts
plugins {
    alias(libs.plugins.project.android.library)
    alias(libs.plugins.project.android.library.compose)
}

dependencies {
    implementation(project(":ui:design-system"))
    
    implementation(libs.androidx.compose.bom)
    implementation(libs.androidx.material3)
    implementation(libs.androidx.ui.tooling.preview)
    implementation(libs.coil)
    
    debugImplementation(libs.androidx.ui.tooling)
}
```

### Primary Button Component
```kotlin
// ui/components/src/main/java/com/myproject/ui/components/buttons/PrimaryButton.kt
package com.myproject.ui.components.buttons

import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.material3.Button
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.tooling.preview.Preview
import com.myproject.ui.design.theme.Dimensions
import com.myproject.ui.design.theme.MyAppTheme

@Composable
fun PrimaryButton(
    text: String,
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
    enabled: Boolean = true,
) {
    Button(
        onClick = onClick,
        modifier = modifier
            .fillMaxWidth()
            .height(Dimensions.buttonHeight),
        enabled = enabled,
    ) {
        Text(
            text = text,
            style = MaterialTheme.typography.labelLarge,
            textAlign = TextAlign.Center
        )
    }
}

@Preview
@Composable
private fun PrimaryButtonPreview() {
    MyAppTheme {
        PrimaryButton(
            text = "Primary Button",
            onClick = { }
        )
    }
}
```

### Loading Indicator
```kotlin
// ui/components/src/main/java/com/myproject/ui/components/loading/LoadingIndicator.kt
package com.myproject.ui.components.loading

import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.size
import androidx.compose.material3.CircularProgressIndicator
import androidx.compose.material3.MaterialTheme
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import com.myproject.ui.design.theme.MyAppTheme

@Composable
fun LoadingIndicator(
    modifier: Modifier = Modifier,
    size: androidx.compose.ui.unit.Dp = 40.dp
) {
    Box(
        modifier = modifier.fillMaxSize(),
        contentAlignment = Alignment.Center
    ) {
        CircularProgressIndicator(
            modifier = Modifier.size(size),
            color = MaterialTheme.colorScheme.primary
        )
    }
}

@Preview
@Composable
private fun LoadingIndicatorPreview() {
    MyAppTheme {
        LoadingIndicator()
    }
}
```

### Empty State Component
```kotlin
// ui/components/src/main/java/com/myproject/ui/components/common/EmptyState.kt
package com.myproject.ui.components.common

import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.tooling.preview.Preview
import com.myproject.ui.design.theme.Dimensions
import com.myproject.ui.design.theme.MyAppTheme

@Composable
fun EmptyState(
    title: String,
    description: String,
    modifier: Modifier = Modifier,
    action: @Composable (() -> Unit)? = null
) {
    Column(
        modifier = modifier
            .fillMaxSize()
            .padding(Dimensions.spacing32),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text(
            text = title,
            style = MaterialTheme.typography.headlineSmall,
            textAlign = TextAlign.Center,
            color = MaterialTheme.colorScheme.onSurface
        )
        
        Spacer(modifier = Modifier.height(Dimensions.spacing16))
        
        Text(
            text = description,
            style = MaterialTheme.typography.bodyMedium,
            textAlign = TextAlign.Center,
            color = MaterialTheme.colorScheme.onSurfaceVariant
        )
        
        action?.let {
            Spacer(modifier = Modifier.height(Dimensions.spacing24))
            it()
        }
    }
}

@Preview
@Composable
private fun EmptyStatePreview() {
    MyAppTheme {
        EmptyState(
            title = "No items found",
            description = "There are no items to display at the moment. Try adding some content."
        )
    }
}
```

## 4. Integration Steps

### Update Settings.gradle.kts
```kotlin
// Add to settings.gradle.kts
include(":app")
include(":ui:design-system")
include(":ui:components")
```

### Update Navigation Host
```kotlin
// Update core/navigation/src/main/java/com/myproject/core/navigation/NavigationHost.kt
@Composable
fun NavigationHost() {
    val navController = rememberNavController()

    NavHost(
        route = RootRoute::class,
        navController = navController,
        startDestination = HomeRoute
    ) {
        // Add your feature graphs here
        animatedComposable<HomeRoute> { 
            // Temporary home screen content
            Box(
                modifier = Modifier.fillMaxSize(),
                contentAlignment = Alignment.Center
            ) {
                Text(
                    text = "Welcome to My App!",
                    style = MaterialTheme.typography.headlineMedium
                )
            }
        }
    }
}
```

This application setup provides a complete foundation with theming, components, and proper navigation integration.