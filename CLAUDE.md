# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Workspace Overview

This is a JetBrains IDE workspace containing four IntelliJ plugin projects. Navigate using the workspace configuration at `.idea/jb-workspace.xml`.

| Project | Path | Purpose |
|---------|------|---------|
| **teams-plugin** | `../teams-plugin` | Microsoft Teams integration for IDE |
| **Package Finder (UDM)** | `../ncies-Unified-Dependency-Manager-UDM-` | NuGet-style dependency manager UI |
| **ConSync** | `../ConSync` | Markdown-to-Confluence sync CLI |
| **docs-plugin** | `../docs-plugin` | Swimm.io-style code documentation |

## Build Systems

### Gradle Projects (teams-plugin, Package Finder, docs-plugin)

```bash
./gradlew build          # Build plugin
./gradlew runIde         # Launch sandboxed IDE with plugin
./gradlew test           # Run tests
./gradlew verifyPlugin   # Verify IDE compatibility
./gradlew buildPlugin    # Create distributable ZIP
```

### Maven Project (ConSync)

```bash
mvn clean package        # Build fat JAR
mvn test                 # Run tests
java -jar consync-cli/target/consync.jar --help  # Run CLI
```

## Technology Stack

All projects target **Java 21** with **Kotlin 2.1+**.

| Component | Technology |
|-----------|------------|
| Build (Gradle) | Kotlin DSL, IntelliJ Platform Gradle Plugin 2.10+ |
| UI Framework | Jetpack Compose for Desktop (Jewel) |
| HTTP | OkHttp 4.x |
| Serialization | Kotlinx Serialization |
| Async | Kotlin Coroutines |

## Project Architectures

### teams-plugin
Microsoft Teams IDE integration using OAuth 2.0 PKCE + Microsoft Graph API.

**Key directories:**
- `auth/` - MSAL4J OAuth client, token management
- `api/` - Graph API clients (Chat, Channel, User, Giphy)
- `services/` - IntelliJ services (Authentication, Chat, Presence, Notifications)
- `ui/components/` - Compose UI (23 components)
- `video/` - LiveKit/Jitsi pair programming

**Keyboard shortcuts:** `Ctrl+Alt+T` (Send to Teams), `Ctrl+Alt+P` (Pair Programming)

### Package Finder (UDM)
Unified dependency manager with NuGet-style UI for Maven/Gradle/NPM.

**Key directories:**
- `gradle/manager/ui/` - Main panels (UnifiedDependencyPanel, PackageListPanel, PackageDetailsPanel)
- `maven/` - Maven Central API client, dependency models
- `npm/` - NPM package search
- `setting/` - Plugin configuration

**Package ID:** `star.intellijplugin.pkgfinder`

### ConSync
Multi-module Maven CLI for Confluence documentation sync.

**Modules:**
- `consync-core` - Markdown parsing, Confluence API, hierarchy management
- `consync-cli` - Clikt CLI commands (sync, validate, status, diff)
- `consync-maven-plugin` - Build integration

**Key services:** `SyncService` (orchestration), `ConfluenceClient` (API), `MarkdownParser`

### docs-plugin
Code documentation system with smart token tracking.

**Key directories:**
- `services/DocsService.kt` - CRUD, indexing, search (622 lines)
- `model/CodeDoc.kt` - Document types (ProgramDoc, TodoDoc, DiscussionDoc)
- `model/SmartToken.kt` - Code reference tracking with change detection

**Storage:** `.docs/` directory with `.td.md` files (YAML frontmatter + Markdown)

**Keyboard shortcuts:** `Ctrl+Alt+D` (Create Doc), `Ctrl+Shift+D` (Find Doc)

## Plugin Configuration

All Gradle plugins define extensions in `src/main/resources/META-INF/plugin.xml`:
- Tool windows, services, actions, settings pages
- IDE version compatibility (`sinceBuild` constraint)
- Required/optional plugin dependencies

## Testing

```bash
# Gradle projects
./gradlew test                    # Unit tests
./gradlew runIdeForUiTests        # UI tests (Package Finder)

# Maven (ConSync)
mvn test
```

Test frameworks: JUnit 4/5, MockK 1.13.x, WireMock (API mocking)

## IDE Compatibility

- **teams-plugin, docs-plugin:** IntelliJ 2025.2.4+ (sinceBuild 252.25557)
- **Package Finder:** IntelliJ 2024.2+ (sinceBuild 242)
- **ConSync:** N/A (CLI tool, requires Java 21+)
