# Android Project Starter

**AI-assisted Android project creation toolkit with enterprise-grade architecture documentation and templates.**

This repository is specifically designed for use with AI coding assistants like Claude Code to create new Android projects and features. It contains comprehensive guides, templates, and configuration files that AI tools can reference to implement clean architecture, dependency injection, Jetpack Compose, and modular build systems following industry best practices.

## Components

- **Enterprise Architecture** - Clean Architecture with MVVM, Repository pattern, and proper layer separation
- **Convention Plugins** - Build configurations with type-safe dependency management
- **Jetpack Compose** - Modern UI toolkit with Material Design components
- **Dependency Injection** - Hilt integration across all layers
- **Database Setup** - Room database configuration with migration support
- **Network Layer** - Retrofit + OkHttp with serialization
- **Testing Framework** - Unit tests, instrumented tests, and JUnit5 support
- **Multi-Module** - Scalable module structure for large teams
- **Build Optimization** - Release builds with code obfuscation

## Usage

### With AI Tools (Primary Method)

This repository is specifically designed for AI coding assistants:

1. **Clone or download** this repository to your workspace
2. **Ask your AI assistant** to create new projects or features using these guides
3. **Reference specific documentation** (e.g., "use the CONVENTION_PLUGINS_GUIDE.md to set up build logic")
4. **Point to template files** for AI to follow patterns and structure
5. **Use for both** new project creation and adding features to existing projects

### With Project Creation Script

```bash
# Clone this repository
git clone <repository-url>
cd starter-android

# Create a new project using the script
./create-android-project.sh MyAwesomeApp com.mycompany myapp

# Navigate to your new project
cd MyAwesomeApp
```

### Prerequisites

- Android Studio Hedgehog | 2023.1.1 or newer
- JDK 11 or newer
- Git (for version control)

## Script Parameters

```bash
./create-android-project.sh <project-name> <base-package> <app-package> [directory]
```

### Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `project-name` | Project folder and display name | `MyAwesomeApp` |
| `base-package` | Company/organization package | `com.mycompany` |
| `app-package` | Application identifier | `myapp` |
| `directory` | Target directory (optional) | `/path/to/projects` |

### Examples

```bash
# Personal project
./create-android-project.sh WeatherApp com.johnsmith weather

# Company project
./create-android-project.sh TaskManager com.acmecorp tasks

# E-commerce app
./create-android-project.sh ShopApp com.retailcorp shop

# Gaming app
./create-android-project.sh PuzzleGame com.gamedev puzzle
```

## 📦 Generated Project Structure

```
MyAwesomeApp/
├── app/                           # Application module
├── core/
│   ├── database/                  # Room database
│   ├── network/                   # Retrofit + networking
│   ├── navigation/                # Navigation component
│   └── ui/                        # Shared UI components
├── features/
│   ├── home/
│   │   ├── view/                  # Compose UI screens
│   │   └── viewmodel/             # ViewModels & state
│   └── profile/
│       ├── view/
│       └── viewmodel/
├── domain/
│   ├── models/                    # Data models
│   └── usecases/                  # Business logic
├── data/
│   ├── repositories/              # Data repositories
│   └── datasources/               # Local & remote data
├── ui/
│   ├── design-system/             # Design tokens & themes
│   ├── composable/                # Reusable components
│   └── icons/                     # App icons
└── build-logic/                   # Convention plugins
    └── convention/
```

## 🎯 Package Structure

Your project will use professional package naming:

```kotlin
// Base structure
com.mycompany.myapp                 // Application module
com.mycompany.myapp.navigation      // Navigation
com.mycompany.myapp.features.home   // Feature modules
com.mycompany.myapp.ui.designsystem // UI modules
com.mycompany.myapp.core.database   // Core modules
```

## 🛠️ Architecture Overview

### Clean Architecture Layers

```
┌─────────────────────────────────────┐
│              UI Layer               │  ← Compose + ViewModels
├─────────────────────────────────────┤
│            Domain Layer             │  ← Use Cases + Models
├─────────────────────────────────────┤
│             Data Layer              │  ← Repositories + DataSources
└─────────────────────────────────────┘
```

### Convention Plugins

- **🏗️ `project.android.application`** - Main app configuration
- **📱 `project.android.library`** - Library modules
- **🎨 `project.android.library.compose`** - Compose UI modules
- **💉 `project.android.hilt`** - Dependency injection
- **🗄️ `project.android.room`** - Database modules
- **🧪 `project.android.test`** - Testing configuration
- **🏛️ `project.arch.view`** - Feature view modules
- **🧠 `project.arch.viewmodel`** - ViewModel modules

### Key Technologies

- **🎨 Jetpack Compose** - Modern declarative UI
- **💉 Hilt** - Dependency injection
- **🗄️ Room** - Local database
- **🌐 Retrofit** - Network client
- **🧭 Navigation Component** - Type-safe navigation
- **🔄 Coroutines** - Asynchronous programming
- **✅ JUnit5** - Modern testing framework

## 📚 Documentation

Comprehensive guides are included:

- **[Convention Plugins Guide](CONVENTION_PLUGINS_GUIDE.md)** - Build system setup
- **[Architecture Guide](ARCHITECTURE_UPDATES.md)** - Project architecture
- **[Module Creation](MODULE_CREATION_TEMPLATES.md)** - Adding new modules
- **[Navigation Setup](NAVIGATION_ARCHITECTURE.md)** - Navigation patterns
- **[Dependency Injection](DEPENDENCY_INJECTION_SETUP.md)** - Hilt configuration

## 🔧 Customization

### Adding New Features

```bash
# Feature modules follow this pattern:
features/
├── feature-name/
│   ├── view/          # UI components
│   ├── viewmodel/     # State management
│   └── domain/        # Business logic
```

### Custom Build Configuration

Modify convention plugins in `build-logic/convention/` to customize:
- Build types and variants
- Dependency versions
- Code generation
- Testing frameworks

## 🚦 Getting Started Checklist

After creating your project:

- [ ] **Open in Android Studio** and let it sync
- [ ] **Update app icon** in `app/src/main/res/mipmap/`
- [ ] **Configure signing** in `app/build.gradle.kts`
- [ ] **Set app name** in `app/src/main/res/values/strings.xml`
- [ ] **Review package names** match your organization
- [ ] **Run initial build** to verify setup
- [ ] **Run tests** to ensure everything works
- [ ] **Initialize Git repository** for version control


## License

This project is licensed under the MIT License. You are free to fork, modify, and use this code for your own projects.

## Acknowledgments

Based on:
- Android Architecture Samples
- Google's recommended architecture guidelines
- Real-world enterprise Android projects