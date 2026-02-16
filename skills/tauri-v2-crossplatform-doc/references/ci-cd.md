# Tauri V2 CI/CD Reference

## Table of Contents
- [GitHub Actions Workflow](#github-actions-workflow)
- [Platform Matrix](#platform-matrix)
- [Code Signing in CI](#code-signing-in-ci)
- [Release Configuration](#release-configuration)
- [Cross-Compilation](#cross-compilation)

---

## GitHub Actions Workflow

### Complete Multi-Platform Workflow

```yaml
name: 'publish'

on:
  workflow_dispatch:
  push:
    branches:
      - release

jobs:
  publish-tauri:
    permissions:
      contents: write
    strategy:
      fail-fast: false
      matrix:
        include:
          # macOS Apple Silicon
          - platform: 'macos-latest'
            args: '--target aarch64-apple-darwin'
          # macOS Intel
          - platform: 'macos-latest'
            args: '--target x86_64-apple-darwin'
          # Linux x64
          - platform: 'ubuntu-22.04'
            args: ''
          # Linux ARM64 (public repos only)
          - platform: 'ubuntu-22.04-arm'
            args: ''
          # Windows x64
          - platform: 'windows-latest'
            args: ''

    runs-on: ${{ matrix.platform }}
    steps:
      - uses: actions/checkout@v4

      # Linux dependencies
      - name: Install dependencies (Ubuntu)
        if: matrix.platform == 'ubuntu-22.04' || matrix.platform == 'ubuntu-22.04-arm'
        run: |
          sudo apt-get update
          sudo apt-get install -y libwebkit2gtk-4.1-dev \
            libappindicator3-dev \
            librsvg2-dev \
            patchelf

      # Node.js setup
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: lts/*
          cache: 'npm'  # or 'yarn' or 'pnpm'

      # Rust setup
      - name: Install Rust stable
        uses: dtolnay/rust-toolchain@stable
        with:
          targets: ${{ matrix.platform == 'macos-latest' && 'aarch64-apple-darwin,x86_64-apple-darwin' || '' }}

      # Rust cache
      - name: Rust cache
        uses: swatinem/rust-cache@v2
        with:
          workspaces: './src-tauri -> target'

      # Install frontend dependencies
      - name: Install frontend dependencies
        run: npm install  # or yarn install / pnpm install

      # Build and release
      - uses: tauri-apps/tauri-action@v0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tagName: app-v__VERSION__
          releaseName: 'App v__VERSION__'
          releaseBody: 'See the assets to download this version and install.'
          releaseDraft: true
          prerelease: false
          args: ${{ matrix.args }}
```

---

## Platform Matrix

### All Platforms

```yaml
matrix:
  include:
    - platform: 'macos-latest'
      args: '--target aarch64-apple-darwin'
    - platform: 'macos-latest'
      args: '--target x86_64-apple-darwin'
    - platform: 'ubuntu-22.04'
      args: ''
    - platform: 'ubuntu-22.04-arm'
      args: ''
    - platform: 'windows-latest'
      args: ''
```

### Simple Matrix

```yaml
matrix:
  platform: [macos-latest, ubuntu-22.04, windows-latest]
```

### Target Architectures

| Platform | Target | Architecture |
|----------|--------|--------------|
| macOS | `aarch64-apple-darwin` | Apple Silicon (M1+) |
| macOS | `x86_64-apple-darwin` | Intel |
| Linux | (default) | x64 |
| Linux | `ubuntu-22.04-arm` | ARM64 |
| Windows | (default) | x64 |

---

## Code Signing in CI

### macOS Signing

```yaml
jobs:
  build-macos:
    runs-on: macos-latest
    env:
      APPLE_ID: ${{ secrets.APPLE_ID }}
      APPLE_PASSWORD: ${{ secrets.APPLE_PASSWORD }}
    steps:
      - name: Import Apple Developer Certificate
        env:
          APPLE_CERTIFICATE: ${{ secrets.APPLE_CERTIFICATE }}
          APPLE_CERTIFICATE_PASSWORD: ${{ secrets.APPLE_CERTIFICATE_PASSWORD }}
          KEYCHAIN_PASSWORD: ${{ secrets.KEYCHAIN_PASSWORD }}
        run: |
          echo $APPLE_CERTIFICATE | base64 --decode > certificate.p12
          security create-keychain -p "$KEYCHAIN_PASSWORD" build.keychain
          security default-keychain -s build.keychain
          security unlock-keychain -p "$KEYCHAIN_PASSWORD" build.keychain
          security set-keychain-settings -t 3600 -u build.keychain
          security import certificate.p12 -k build.keychain \
            -P "$APPLE_CERTIFICATE_PASSWORD" -T /usr/bin/codesign
          security set-key-partition-list -S apple-tool:,apple:,codesign: \
            -s -k "$KEYCHAIN_PASSWORD" build.keychain
          security find-identity -v -p codesigning build.keychain

      - name: Get Certificate ID
        run: |
          CERT_INFO=$(security find-identity -v -p codesigning build.keychain | grep "Apple Development")
          CERT_ID=$(echo "$CERT_INFO" | awk -F'"' '{print $2}')
          echo "CERT_ID=$CERT_ID" >> $GITHUB_ENV

      - uses: tauri-apps/tauri-action@v0
        env:
          APPLE_SIGNING_IDENTITY: ${{ env.CERT_ID }}
```

#### Required Secrets for macOS

| Secret | Description |
|--------|-------------|
| `APPLE_CERTIFICATE` | Base64-encoded .p12 certificate |
| `APPLE_CERTIFICATE_PASSWORD` | Certificate password |
| `APPLE_ID` | Apple ID email |
| `APPLE_PASSWORD` | App-specific password |
| `KEYCHAIN_PASSWORD` | Temporary keychain password |

### Windows Signing

```yaml
- uses: tauri-apps/tauri-action@v0
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    TAURI_SIGNING_PRIVATE_KEY: ${{ secrets.TAURI_SIGNING_PRIVATE_KEY }}
    TAURI_SIGNING_PRIVATE_KEY_PASSWORD: ${{ secrets.TAURI_SIGNING_PRIVATE_KEY_PASSWORD }}
```

Or with Azure Code Signing in `tauri.conf.json`:

```json
{
  "bundle": {
    "windows": {
      "signCommand": "trusted-signing-cli -e $AZURE_ENDPOINT -a $AZURE_ACCOUNT -c $AZURE_PROFILE %1"
    }
  }
}
```

---

## Release Configuration

### tauri-action Options

```yaml
- uses: tauri-apps/tauri-action@v0
  with:
    # Tag format (__VERSION__ replaced with app version)
    tagName: app-v__VERSION__

    # Release title
    releaseName: 'App v__VERSION__'

    # Release body/description
    releaseBody: 'See the assets to download this version and install.'

    # Create as draft (requires manual publish)
    releaseDraft: true

    # Mark as prerelease
    prerelease: false

    # Build arguments
    args: '--target aarch64-apple-darwin'

    # Tauri CLI version (optional)
    tauriVersion: 'v2'

    # Working directory
    projectPath: './frontend'

    # Skip building frontend
    buildFrontend: false

    # Include updater artifacts
    includeUpdaterJson: true
```

### Updater Artifacts

```yaml
- uses: tauri-apps/tauri-action@v0
  with:
    includeUpdaterJson: true
  env:
    TAURI_SIGNING_PRIVATE_KEY: ${{ secrets.TAURI_SIGNING_PRIVATE_KEY }}
```

---

## Cross-Compilation

### Windows from Linux/macOS

```bash
# Install cargo-xwin
cargo install cargo-xwin

# Build
npm run tauri build -- --runner cargo-xwin --target x86_64-pc-windows-msvc
```

### In GitHub Actions

```yaml
- name: Build Windows (from Linux)
  run: |
    cargo install cargo-xwin
    npm run tauri build -- --runner cargo-xwin --target x86_64-pc-windows-msvc
```

### Linux ARM64 (Cross-compile)

```yaml
- name: Install ARM64 dependencies
  run: |
    sudo dpkg --add-architecture arm64
    sudo apt-get update
    sudo apt-get install -y libwebkit2gtk-4.1-dev:arm64
```

---

## Complete CI/CD Example

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    permissions:
      contents: write
    strategy:
      fail-fast: false
      matrix:
        include:
          - platform: macos-latest
            args: '--target aarch64-apple-darwin'
          - platform: macos-latest
            args: '--target x86_64-apple-darwin'
          - platform: ubuntu-22.04
            args: ''
          - platform: windows-latest
            args: ''

    runs-on: ${{ matrix.platform }}
    steps:
      - uses: actions/checkout@v4

      - name: Install Linux deps
        if: matrix.platform == 'ubuntu-22.04'
        run: |
          sudo apt-get update
          sudo apt-get install -y libwebkit2gtk-4.1-dev libappindicator3-dev librsvg2-dev patchelf

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - uses: dtolnay/rust-toolchain@stable
        with:
          targets: ${{ matrix.platform == 'macos-latest' && 'aarch64-apple-darwin,x86_64-apple-darwin' || '' }}

      - uses: swatinem/rust-cache@v2
        with:
          workspaces: './src-tauri -> target'

      - run: npm ci
      - run: npm run build

      - uses: tauri-apps/tauri-action@v0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          APPLE_CERTIFICATE: ${{ secrets.APPLE_CERTIFICATE }}
          APPLE_CERTIFICATE_PASSWORD: ${{ secrets.APPLE_CERTIFICATE_PASSWORD }}
          APPLE_SIGNING_IDENTITY: ${{ secrets.APPLE_SIGNING_IDENTITY }}
          APPLE_ID: ${{ secrets.APPLE_ID }}
          APPLE_PASSWORD: ${{ secrets.APPLE_PASSWORD }}
          APPLE_TEAM_ID: ${{ secrets.APPLE_TEAM_ID }}
        with:
          tagName: ${{ github.ref_name }}
          releaseName: 'Release ${{ github.ref_name }}'
          releaseBody: 'See the assets to download and install.'
          releaseDraft: true
          prerelease: false
          args: ${{ matrix.args }}
```
