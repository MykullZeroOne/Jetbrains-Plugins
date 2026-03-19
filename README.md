# JetBrains Plugin Repository

Custom plugin repository for internal JetBrains IDE plugins.

## For Users: Install Plugins

1. Open your JetBrains IDE (IntelliJ IDEA, WebStorm, etc.)
2. Go to **Settings > Plugins > Gear Icon > Manage Plugin Repositories**
3. Add this URL:
   ```
   https://MykullZeroOne.github.io/Jetbrains-Plugins/updatePlugins.xml
   ```
4. Go to the **Marketplace** tab and search for the plugin name
5. Click **Install**

Updates will be detected automatically.

## Available Plugins

| Plugin | Description |
|--------|-------------|
| **Keyscript IDE** | Keystone script development with JCEF preview, proxy, and CR completions |
| **Docs Plugin** | Documentation browser for JetBrains IDEs |

## For Plugin Developers: Add a New Plugin

### 1. Register the plugin

Create a JSON file in `plugins/` with your plugin metadata:

```json
{
  "id": "com.your.plugin.id",
  "repo": "MykullZeroOne/your-plugin-repo",
  "name": "Your Plugin Name",
  "description": "What it does.",
  "vendor": "Your Name",
  "sinceBuild": "253",
  "untilBuild": "253.*"
}
```

### 2. Add the release workflow to your plugin repo

In your plugin repo, create `.github/workflows/release.yml`:

```yaml
name: Release Plugin
on:
  push:
    tags: ['v*']

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
      - uses: gradle/actions/setup-gradle@v4

      - name: Build plugin
        run: ./gradlew buildPlugin

      - name: Create release
        run: |
          TAG="${GITHUB_REF#refs/tags/}"
          ZIP=$(ls build/distributions/*.zip | head -1)
          gh release create "$TAG" "$ZIP" --title "Release $TAG"
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Trigger registry update
        run: |
          gh api repos/MykullZeroOne/Jetbrains-Plugins/dispatches \
            -f event_type=plugin-released \
            -f client_payload='{"plugin":"${{ github.repository }}"}'
        env:
          GH_TOKEN: ${{ secrets.REGISTRY_TOKEN }}
```

### 3. Set up the REGISTRY_TOKEN secret

The plugin repo needs a `REGISTRY_TOKEN` secret (a GitHub PAT with `repo` scope) to trigger the registry rebuild. Create one at [GitHub Settings > Developer Settings > Personal Access Tokens](https://github.com/settings/tokens) and add it as a secret in your plugin repo.

### 4. Tag a release

```bash
git tag v1.0.0
git push origin v1.0.0
```

The workflow will build the plugin, create a GitHub Release with the ZIP, and trigger the central registry to regenerate `updatePlugins.xml`.
