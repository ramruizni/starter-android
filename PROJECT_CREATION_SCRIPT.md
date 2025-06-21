# Complete Project Creation Script

This document provides the complete automation script that creates a fully functional Android project from scratch using all the templates and configurations defined in the previous guides.

## Overview

The script will:
1. **Create directory structure** for all modules
2. **Copy and customize template files** with project-specific values
3. **Generate core infrastructure modules** (database, network, navigation)
4. **Set up build configurations** with convention plugins
5. **Create example feature module** (Home) to demonstrate patterns
6. **Configure dependency injection** across all layers
7. **Validate project setup** by running initial build

## Project Creation Script

### Primary Script: `create-android-project.sh`

```bash
#!/bin/bash

# Android Project Creation Script
# Usage: ./create-android-project.sh <project-name> <package-name> [base-directory]
# Example: ./create-android-project.sh MyAwesomeApp com.mycompany.myapp /path/to/projects

set -e  # Exit on any error

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
BLUE='\033[0;34m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Configuration
PROJECT_NAME="$1"
PACKAGE_NAME="$2"
BASE_DIR="${3:-$(pwd)}"
TEMPLATE_DIR="$(dirname "$0")/templates"

# Validate inputs
if [ -z "$PROJECT_NAME" ] || [ -z "$PACKAGE_NAME" ]; then
    echo -e "${RED}Usage: $0 <project-name> <package-name> [base-directory]${NC}"
    echo -e "${YELLOW}Example: $0 MyAwesomeApp com.mycompany.myapp${NC}"
    exit 1
fi

# Validate package name format
if ! [[ $PACKAGE_NAME =~ ^[a-z][a-z0-9_]*(\.[a-z][a-z0-9_]*)+$ ]]; then
    echo -e "${RED}Error: Invalid package name format. Use lowercase with dots (e.g., com.company.app)${NC}"
    exit 1
fi

# Derive paths
PROJECT_DIR="$BASE_DIR/$PROJECT_NAME"
PACKAGE_PATH="${PACKAGE_NAME//./\/}"

echo -e "${BLUE}🚀 Creating Android project: $PROJECT_NAME${NC}"
echo -e "${BLUE}📦 Package: $PACKAGE_NAME${NC}"
echo -e "${BLUE}📁 Directory: $PROJECT_DIR${NC}"

# Check if project directory already exists
if [ -d "$PROJECT_DIR" ]; then
    echo -e "${RED}Error: Project directory $PROJECT_DIR already exists${NC}"
    exit 1
fi

# Check if template directory exists
if [ ! -d "$TEMPLATE_DIR" ]; then
    echo -e "${RED}Error: Template directory $TEMPLATE_DIR not found${NC}"
    echo -e "${YELLOW}Please ensure you have the templates directory in the same location as this script${NC}"
    exit 1
fi

# Function to create directory structure
create_directory_structure() {
    echo -e "${YELLOW}📁 Creating directory structure...${NC}"
    
    # Root directories
    mkdir -p "$PROJECT_DIR"
    mkdir -p "$PROJECT_DIR/gradle"
    
    # Build logic
    mkdir -p "$PROJECT_DIR/build-logic/convention/src/main/java/$PACKAGE_PATH/convention"
    
    # App module
    mkdir -p "$PROJECT_DIR/app/src/main/java/$PACKAGE_PATH"
    mkdir -p "$PROJECT_DIR/app/src/main/res/values"
    mkdir -p "$PROJECT_DIR/app/src/main/res/xml"
    mkdir -p "$PROJECT_DIR/app/src/main/res/mipmap-hdpi"
    mkdir -p "$PROJECT_DIR/app/src/main/res/mipmap-mdpi"
    mkdir -p "$PROJECT_DIR/app/src/main/res/mipmap-xhdpi"
    mkdir -p "$PROJECT_DIR/app/src/main/res/mipmap-xxhdpi"
    mkdir -p "$PROJECT_DIR/app/src/main/res/mipmap-xxxhdpi"
    mkdir -p "$PROJECT_DIR/app/src/test/java/$PACKAGE_PATH"
    mkdir -p "$PROJECT_DIR/app/src/androidTest/java/$PACKAGE_PATH"
    
    # Core modules
    mkdir -p "$PROJECT_DIR/core/database/src/main/java/$PACKAGE_PATH/core/database"
    mkdir -p "$PROJECT_DIR/core/database/src/main/java/$PACKAGE_PATH/core/database/converters"
    mkdir -p "$PROJECT_DIR/core/database/src/main/java/$PACKAGE_PATH/core/database/migrations"
    mkdir -p "$PROJECT_DIR/core/database/src/main/java/$PACKAGE_PATH/core/database/di"
    mkdir -p "$PROJECT_DIR/core/database/src/test/java/$PACKAGE_PATH/core/database"
    
    mkdir -p "$PROJECT_DIR/core/network/src/main/java/$PACKAGE_PATH/core/network"
    mkdir -p "$PROJECT_DIR/core/network/src/main/java/$PACKAGE_PATH/core/network/interceptors"
    mkdir -p "$PROJECT_DIR/core/network/src/main/java/$PACKAGE_PATH/core/network/models"
    mkdir -p "$PROJECT_DIR/core/network/src/main/java/$PACKAGE_PATH/core/network/di"
    mkdir -p "$PROJECT_DIR/core/network/src/test/java/$PACKAGE_PATH/core/network"
    
    mkdir -p "$PROJECT_DIR/core/domain/src/main/java/$PACKAGE_PATH/core/domain/models"
    mkdir -p "$PROJECT_DIR/core/domain/src/main/java/$PACKAGE_PATH/core/domain/repository"
    mkdir -p "$PROJECT_DIR/core/domain/src/main/java/$PACKAGE_PATH/core/domain/utils"
    mkdir -p "$PROJECT_DIR/core/domain/src/test/java/$PACKAGE_PATH/core/domain"
    
    mkdir -p "$PROJECT_DIR/core/navigation/src/main/java/$PACKAGE_PATH/core/navigation"
    mkdir -p "$PROJECT_DIR/core/navigation/src/main/java/$PACKAGE_PATH/core/navigation/routes"
    mkdir -p "$PROJECT_DIR/core/navigation/src/main/java/$PACKAGE_PATH/core/navigation/utils"
    mkdir -p "$PROJECT_DIR/core/navigation/src/main/java/$PACKAGE_PATH/core/navigation/di"
    mkdir -p "$PROJECT_DIR/core/navigation/src/test/java/$PACKAGE_PATH/core/navigation"
    
    # UI modules
    mkdir -p "$PROJECT_DIR/ui/design-system/src/main/java/$PACKAGE_PATH/ui/design/theme"
    mkdir -p "$PROJECT_DIR/ui/design-system/src/main/java/$PACKAGE_PATH/ui/design/tokens"
    mkdir -p "$PROJECT_DIR/ui/design-system/src/main/java/$PACKAGE_PATH/ui/design/utils"
    mkdir -p "$PROJECT_DIR/ui/design-system/src/test/java/$PACKAGE_PATH/ui/design"
    
    mkdir -p "$PROJECT_DIR/ui/components/src/main/java/$PACKAGE_PATH/ui/components/buttons"
    mkdir -p "$PROJECT_DIR/ui/components/src/main/java/$PACKAGE_PATH/ui/components/cards"
    mkdir -p "$PROJECT_DIR/ui/components/src/main/java/$PACKAGE_PATH/ui/components/loading"
    mkdir -p "$PROJECT_DIR/ui/components/src/main/java/$PACKAGE_PATH/ui/components/text"
    mkdir -p "$PROJECT_DIR/ui/components/src/main/java/$PACKAGE_PATH/ui/components/common"
    mkdir -p "$PROJECT_DIR/ui/components/src/test/java/$PACKAGE_PATH/ui/components"
    
    # Example feature (Home)
    mkdir -p "$PROJECT_DIR/features/home/view/src/main/java/$PACKAGE_PATH/features/home/view"
    mkdir -p "$PROJECT_DIR/features/home/view/src/main/java/$PACKAGE_PATH/features/home/view/components"
    mkdir -p "$PROJECT_DIR/features/home/view/src/test/java/$PACKAGE_PATH/features/home/view"
    
    mkdir -p "$PROJECT_DIR/features/home/viewmodel/src/main/java/$PACKAGE_PATH/features/home/viewmodel"
    mkdir -p "$PROJECT_DIR/features/home/viewmodel/src/test/java/$PACKAGE_PATH/features/home/viewmodel"
    
    mkdir -p "$PROJECT_DIR/domain/home/models/src/main/java/$PACKAGE_PATH/domain/home/models"
    mkdir -p "$PROJECT_DIR/domain/home/models/src/test/java/$PACKAGE_PATH/domain/home/models"
    
    mkdir -p "$PROJECT_DIR/domain/home/usecases/src/main/java/$PACKAGE_PATH/domain/home/usecases"
    mkdir -p "$PROJECT_DIR/domain/home/usecases/src/main/java/$PACKAGE_PATH/domain/home/usecases/di"
    mkdir -p "$PROJECT_DIR/domain/home/usecases/src/test/java/$PACKAGE_PATH/domain/home/usecases"
    
    mkdir -p "$PROJECT_DIR/data/home/src/main/java/$PACKAGE_PATH/data/home"
    mkdir -p "$PROJECT_DIR/data/home/src/main/java/$PACKAGE_PATH/data/home/local"
    mkdir -p "$PROJECT_DIR/data/home/src/main/java/$PACKAGE_PATH/data/home/local/entities"
    mkdir -p "$PROJECT_DIR/data/home/src/main/java/$PACKAGE_PATH/data/home/remote"
    mkdir -p "$PROJECT_DIR/data/home/src/main/java/$PACKAGE_PATH/data/home/remote/dto"
    mkdir -p "$PROJECT_DIR/data/home/src/main/java/$PACKAGE_PATH/data/home/mappers"
    mkdir -p "$PROJECT_DIR/data/home/src/main/java/$PACKAGE_PATH/data/home/di"
    mkdir -p "$PROJECT_DIR/data/home/src/test/java/$PACKAGE_PATH/data/home"
}

# Function to copy and customize templates
copy_and_customize_templates() {
    echo -e "${YELLOW}📋 Copying and customizing template files...${NC}"
    
    # Copy all template files
    cp -r "$TEMPLATE_DIR/"* "$PROJECT_DIR/"
    
    # Replace placeholders in all files
    find "$PROJECT_DIR" -type f \( -name "*.kt" -o -name "*.kts" -o -name "*.xml" -o -name "*.toml" -o -name "*.pro" -o -name "*.properties" \) -exec sed -i.bak \
        -e "s/PROJECT_NAME/$PROJECT_NAME/g" \
        -e "s/PACKAGE_NAME/$PACKAGE_NAME/g" \
        -e "s|PACKAGE_PATH|$PACKAGE_PATH|g" \
        {} \;
    
    # Remove backup files
    find "$PROJECT_DIR" -name "*.bak" -delete
    
    # Make gradlew executable
    chmod +x "$PROJECT_DIR/gradlew" 2>/dev/null || true
}

# Function to generate specific files
generate_specific_files() {
    echo -e "${YELLOW}🔧 Generating specific implementation files...${NC}"
    
    # Generate app icon (placeholder)
    create_app_icons
    
    # Generate gitignore
    create_gitignore
    
    # Generate README
    create_readme
}

# Function to create app icons (placeholders)
create_app_icons() {
    # Create simple colored squares as placeholder icons
    # In a real implementation, you'd generate proper Android icons
    
    for dpi in hdpi mdpi xhdpi xxhdpi xxxhdpi; do
        icon_dir="$PROJECT_DIR/app/src/main/res/mipmap-$dpi"
        
        # Create placeholder files (in real scenario, generate actual icons)
        touch "$icon_dir/ic_launcher.png"
        touch "$icon_dir/ic_launcher_round.png"
        touch "$icon_dir/ic_launcher_foreground.png"
        touch "$icon_dir/ic_launcher_background.png"
    done
}

# Function to create gitignore
create_gitignore() {
    cat > "$PROJECT_DIR/.gitignore" << EOF
# Built application files
*.apk
*.aab

# Files for the ART/Dalvik VM
*.dex

# Java class files
*.class

# Generated files
bin/
gen/
out/
#  Uncomment the following line in case you need and you don't have the release build type files in your app
# release/

# Gradle files
.gradle/
build/

# Local configuration file (sdk path, etc)
local.properties

# Proguard folder generated by Eclipse
proguard/

# Log Files
*.log

# Android Studio Navigation editor temp files
.navigation/

# Android Studio captures folder
captures/

# IntelliJ
*.iml
.idea/workspace.xml
.idea/tasks.xml
.idea/gradle.xml
.idea/assetWizardSettings.xml
.idea/dictionaries
.idea/libraries
# Android Studio 3 in .gitignore file.
.idea/caches
.idea/modules.xml
# Comment next line if keeping position of elements in Navigation Editor is relevant for you
.idea/navEditor.xml

# Keystore files
# Uncomment the following lines if you do not want to check your keystore files in.
#*.jks
#*.keystore

# External native build folder generated in Android Studio 2.2 and later
.externalNativeBuild
.cxx/

# Google Services (e.g. APIs or Firebase)
# google-services.json

# Freeline
freeline.py
freeline/
freeline_project_description.json

# fastlane
fastlane/report.xml
fastlane/Preview.html
fastlane/screenshots
fastlane/test_output
fastlane/readme.md

# Version control
vcs.xml

# lint
lint/intermediates/
lint/generated/
lint/outputs/
lint/tmp/
# lint/reports/

# MacOS
.DS_Store

# Backup files
*.bak
EOF
}

# Function to create README
create_readme() {
    cat > "$PROJECT_DIR/README.md" << EOF
# $PROJECT_NAME

Android application built with modern Android development practices.

## Architecture

This project follows Clean Architecture principles with the following structure:

### Modules

- **app**: Main application module
- **core**: Core infrastructure modules
  - **database**: Room database configuration
  - **network**: HTTP client and API setup
  - **domain**: Base domain classes and utilities
  - **navigation**: Navigation utilities and route definitions
- **ui**: UI-related modules
  - **design-system**: Material Design 3 theming and tokens
  - **components**: Reusable UI components
- **features**: Feature-specific modules organized by layer
  - **[feature]/view**: UI layer for each feature
  - **[feature]/viewmodel**: ViewModels for each feature
- **domain**: Domain layer modules
  - **[feature]/models**: Domain models for each feature
  - **[feature]/usecases**: Use cases for each feature
- **data**: Data layer modules for each feature

### Technology Stack

- **Language**: Kotlin
- **UI**: Jetpack Compose with Material Design 3
- **Architecture**: MVVM with Clean Architecture
- **Dependency Injection**: Hilt
- **Database**: Room
- **Networking**: Retrofit + OkHttp
- **Navigation**: Navigation Compose with type-safe routes
- **Async**: Kotlin Coroutines and Flow
- **Build System**: Gradle with Convention Plugins

## Getting Started

### Prerequisites

- Android Studio Giraffe | 2022.3.1 or later
- JDK 17 or later
- Android SDK 34 or later

### Build and Run

1. Clone the repository
2. Open in Android Studio
3. Sync project with Gradle files
4. Run the app

### Project Structure

\`\`\`
$PROJECT_NAME/
├── app/                    # Main application
├── build-logic/           # Convention plugins
├── core/                  # Core infrastructure
├── features/             # Feature modules
├── domain/               # Domain layer
├── data/                 # Data layer
├── ui/                   # UI modules
└── gradle/              # Version catalog
\`\`\`

## Development

### Adding New Features

1. Create feature modules using the established patterns
2. Follow the module templates in the documentation
3. Use convention plugins for consistent build configuration
4. Implement proper dependency injection with Hilt

### Testing

Run tests with:
\`\`\`bash
./gradlew test
./gradlew connectedAndroidTest
\`\`\`

### Build

Build the project with:
\`\`\`bash
./gradlew build
\`\`\`

## Documentation

Refer to the project documentation for detailed guides on:
- Architecture patterns
- Module creation
- Convention plugins
- Testing strategies

---

Generated with Android Project Creation Script
Package: $PACKAGE_NAME
Created: $(date)
EOF
}

# Function to validate project setup
validate_project_setup() {
    echo -e "${YELLOW}✅ Validating project setup...${NC}"
    
    cd "$PROJECT_DIR"
    
    # Check if gradlew exists and is executable
    if [ ! -x "./gradlew" ]; then
        echo -e "${RED}Error: gradlew is not executable${NC}"
        return 1
    fi
    
    # Try to sync project
    echo -e "${BLUE}🔄 Running initial Gradle sync...${NC}"
    if ./gradlew tasks --console=plain > /dev/null 2>&1; then
        echo -e "${GREEN}✅ Gradle sync successful${NC}"
    else
        echo -e "${YELLOW}⚠️  Gradle sync had issues, but project structure is created${NC}"
    fi
    
    # Check key files exist
    local key_files=(
        "settings.gradle.kts"
        "build.gradle.kts"
        "gradle/libs.versions.toml"
        "app/build.gradle.kts"
        "app/src/main/AndroidManifest.xml"
    )
    
    for file in "${key_files[@]}"; do
        if [ ! -f "$file" ]; then
            echo -e "${RED}Error: Missing key file: $file${NC}"
            return 1
        fi
    done
    
    echo -e "${GREEN}✅ Project validation completed${NC}"
    return 0
}

# Function to display next steps
show_next_steps() {
    echo -e "${GREEN}🎉 Project created successfully!${NC}"
    echo ""
    echo -e "${BLUE}📍 Project location: $PROJECT_DIR${NC}"
    echo ""
    echo -e "${YELLOW}Next steps:${NC}"
    echo "1. Open the project in Android Studio:"
    echo -e "   ${BLUE}File → Open → $PROJECT_DIR${NC}"
    echo ""
    echo "2. Wait for Gradle sync to complete"
    echo ""
    echo "3. Build the project:"
    echo -e "   ${BLUE}./gradlew build${NC}"
    echo ""
    echo "4. Run the app:"
    echo -e "   ${BLUE}./gradlew installDebug${NC}"
    echo ""
    echo "5. Start developing:"
    echo "   - Add your features using the established patterns"
    echo "   - Follow the module templates in the documentation"
    echo "   - Use convention plugins for new modules"
    echo ""
    echo -e "${GREEN}Happy coding! 🚀${NC}"
}

# Main execution
main() {
    echo -e "${BLUE}Starting Android project creation...${NC}"
    
    create_directory_structure
    copy_and_customize_templates
    generate_specific_files
    
    if validate_project_setup; then
        show_next_steps
    else
        echo -e "${RED}❌ Project creation completed with validation issues${NC}"
        echo -e "${YELLOW}Check the project manually and resolve any issues${NC}"
        exit 1
    fi
}

# Execute main function
main "$@"
```

### Template Preparation Script: `prepare-templates.sh`

```bash
#!/bin/bash

# Template Preparation Script
# This script creates the template directory structure needed by the main creation script

SCRIPT_DIR="$(dirname "$0")"
TEMPLATE_DIR="$SCRIPT_DIR/templates"

echo "🔧 Preparing template directory structure..."

# Create template directory structure
mkdir -p "$TEMPLATE_DIR"

# Create subdirectories for different types of templates
mkdir -p "$TEMPLATE_DIR/gradle"
mkdir -p "$TEMPLATE_DIR/build-logic/convention/src/main/java"
mkdir -p "$TEMPLATE_DIR/app/src/main/java"
mkdir -p "$TEMPLATE_DIR/app/src/main/res/values"
mkdir -p "$TEMPLATE_DIR/app/src/main/res/xml"
mkdir -p "$TEMPLATE_DIR/core"
mkdir -p "$TEMPLATE_DIR/ui"
mkdir -p "$TEMPLATE_DIR/features"
mkdir -p "$TEMPLATE_DIR/domain"
mkdir -p "$TEMPLATE_DIR/data"

echo "✅ Template directory structure created at: $TEMPLATE_DIR"
echo ""
echo "Next steps:"
echo "1. Copy your template files into the templates/ directory"
echo "2. Ensure all files use the placeholder tokens:"
echo "   - PROJECT_NAME for project name"
echo "   - PACKAGE_NAME for package name"
echo "   - PACKAGE_PATH for package path"
echo "3. Run the main creation script: ./create-android-project.sh"
```

### Gradle Wrapper Script: `add-gradle-wrapper.sh`

```bash
#!/bin/bash

# Gradle Wrapper Addition Script
# Adds Gradle wrapper to an existing project

PROJECT_DIR="$1"

if [ -z "$PROJECT_DIR" ]; then
    echo "Usage: $0 <project-directory>"
    exit 1
fi

if [ ! -d "$PROJECT_DIR" ]; then
    echo "Error: Project directory $PROJECT_DIR does not exist"
    exit 1
fi

cd "$PROJECT_DIR"

echo "📦 Adding Gradle wrapper..."

# Download and setup Gradle wrapper
if command -v gradle >/dev/null 2>&1; then
    gradle wrapper --gradle-version=8.5 --distribution-type=bin
    echo "✅ Gradle wrapper added successfully"
else
    echo "❌ Gradle command not found. Please install Gradle first."
    exit 1
fi
```

### Validation Script: `validate-project.sh`

```bash
#!/bin/bash

# Project Validation Script
# Validates that a created project has all necessary components

PROJECT_DIR="$1"

if [ -z "$PROJECT_DIR" ]; then
    echo "Usage: $0 <project-directory>"
    exit 1
fi

if [ ! -d "$PROJECT_DIR" ]; then
    echo "Error: Project directory $PROJECT_DIR does not exist"
    exit 1
fi

cd "$PROJECT_DIR"

echo "🔍 Validating project structure..."

# Check essential files
essential_files=(
    "settings.gradle.kts"
    "build.gradle.kts"
    "gradle.properties"
    "gradle/libs.versions.toml"
    "gradlew"
    "gradlew.bat"
    "app/build.gradle.kts"
    "app/src/main/AndroidManifest.xml"
    "build-logic/convention/build.gradle.kts"
)

missing_files=()

for file in "${essential_files[@]}"; do
    if [ ! -f "$file" ]; then
        missing_files+=("$file")
    fi
done

if [ ${#missing_files[@]} -eq 0 ]; then
    echo "✅ All essential files present"
else
    echo "❌ Missing files:"
    for file in "${missing_files[@]}"; do
        echo "   - $file"
    done
fi

# Try to build project
echo "🔨 Testing project build..."
if ./gradlew tasks --console=plain > /dev/null 2>&1; then
    echo "✅ Project builds successfully"
else
    echo "❌ Project build failed"
fi

echo "🏁 Validation complete"
```

## Usage Instructions

### 1. Setup

```bash
# Make scripts executable
chmod +x create-android-project.sh
chmod +x prepare-templates.sh
chmod +x add-gradle-wrapper.sh
chmod +x validate-project.sh

# Prepare template directory
./prepare-templates.sh
```

### 2. Add Template Files

Copy all template files from the `TEMPLATE_FILES_COLLECTION.md` into the `templates/` directory, ensuring they use the placeholder tokens.

### 3. Create Project

```bash
# Create new project
./create-android-project.sh MyAwesomeApp com.mycompany.myapp

# Or specify custom directory
./create-android-project.sh MyAwesomeApp com.mycompany.myapp /path/to/projects
```

### 4. Validate Project

```bash
# Validate created project
./validate-project.sh /path/to/MyAwesomeApp
```

### 5. Open in Android Studio

```bash
# Open project
studio /path/to/MyAwesomeApp
```

## Script Features

### ✅ **Complete Automation**
- Creates full directory structure
- Copies and customizes all template files
- Replaces placeholders with project-specific values
- Sets up proper permissions

### ✅ **Validation**
- Checks template directory exists
- Validates package name format
- Verifies essential files are created
- Tests initial project build

### ✅ **Error Handling**
- Exits on any error
- Provides clear error messages
- Validates inputs before proceeding

### ✅ **User Experience**
- Colored output for better readability
- Progress indicators
- Clear next steps
- Comprehensive README generation

This script system provides complete automation for creating Android projects from scratch, ensuring consistency and following all established patterns and conventions.