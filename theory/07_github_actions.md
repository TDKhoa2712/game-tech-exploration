# 07 · GitHub Actions — CI/CD Tự Động

> GitHub Actions là nền tảng CI/CD tích hợp sẵn trong GitHub. Tự động hóa: test, lint, build, deploy — mỗi khi có thay đổi code. Với Flutter, điều này nghĩa là không bao giờ push code lỗi lên main branch.

---

## 1. Khái Niệm Cơ Bản

```
Workflow  →  Job  →  Step  →  Action/Command

Workflow: file YAML trong .github/workflows/
Job: tập hợp steps chạy trên cùng runner (máy chủ)
Step: một lệnh hoặc action cụ thể
Runner: máy chủ GitHub (ubuntu-latest, macos-latest, windows-latest)
```

### Cấu trúc thư mục

```
.github/
└── workflows/
    ├── ci.yml          # Chạy khi có PR hoặc push lên bất kỳ branch
    └── cd.yml          # Chỉ chạy khi merge vào main → build release
```

---

## 2. CI Pipeline — Kiểm Tra Code Chất Lượng

```yaml
# .github/workflows/ci.yml
name: CI

# Trigger: khi nào workflow này chạy
on:
  push:
    branches: ['**']                # mọi branch
  pull_request:
    branches: [main, develop]       # PR vào main hoặc develop

jobs:
  test:
    name: Analyze & Test
    runs-on: ubuntu-latest

    steps:
      # Bước 1: Checkout code
      - name: Checkout repository
        uses: actions/checkout@v4

      # Bước 2: Setup Java (cần cho Android build)
      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      # Bước 3: Setup Flutter
      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.22.0'
          channel: 'stable'
          cache: true              # cache Flutter SDK — tăng tốc đáng kể

      # Bước 4: Cache pub packages
      - name: Cache pub packages
        uses: actions/cache@v4
        with:
          path: ~/.pub-cache
          key: ${{ runner.os }}-pub-${{ hashFiles('**/pubspec.lock') }}
          restore-keys: ${{ runner.os }}-pub-

      # Bước 5: Install dependencies
      - name: Install dependencies
        run: flutter pub get

      # Bước 6: Generate code (Riverpod, Hive adapters,...)
      - name: Generate code
        run: flutter pub run build_runner build --delete-conflicting-outputs

      # Bước 7: Analyze (lint, type check)
      - name: Analyze code
        run: flutter analyze --no-fatal-infos

      # Bước 8: Format check
      - name: Check formatting
        run: dart format --output=none --set-exit-if-changed .

      # Bước 9: Run tests
      - name: Run unit & widget tests
        run: flutter test --coverage

      # Bước 10: Upload coverage report
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v4
        with:
          file: coverage/lcov.info
          fail_ci_if_error: false   # không fail CI nếu Codecov lỗi
        env:
          CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

---

## 3. Build Debug — Kiểm Tra Build Thành Công

```yaml
# Thêm vào ci.yml (job thứ hai, chạy sau khi test pass)
  build-android:
    name: Build Android Debug
    runs-on: ubuntu-latest
    needs: test              # chỉ chạy nếu job "test" thành công

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.22.0'
          cache: true

      - run: flutter pub get
      - run: flutter pub run build_runner build --delete-conflicting-outputs

      - name: Build APK debug
        run: flutter build apk --debug

      # Upload APK để test thủ công
      - name: Upload APK artifact
        uses: actions/upload-artifact@v4
        with:
          name: debug-apk
          path: build/app/outputs/flutter-apk/app-debug.apk
          retention-days: 3   # giữ 3 ngày

  build-ios:
    name: Build iOS (no-codesign)
    runs-on: macos-latest    # iOS chỉ build được trên macOS!
    needs: test

    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.22.0'
          cache: true

      - run: flutter pub get
      - run: flutter pub run build_runner build --delete-conflicting-outputs

      - name: Build iOS (no codesign — chỉ kiểm tra build)
        run: flutter build ios --no-codesign --debug
```

---

## 4. CD Pipeline — Build Release & Deploy

```yaml
# .github/workflows/cd.yml
name: CD — Release Build

on:
  push:
    branches: [main]              # Chỉ chạy khi push/merge vào main
  release:
    types: [created]              # Hoặc khi tạo GitHub Release mới

jobs:
  release-android:
    name: Build Android Release
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.22.0'
          cache: true

      - run: flutter pub get
      - run: flutter pub run build_runner build --delete-conflicting-outputs

      # Decode keystore từ secret (base64)
      - name: Decode keystore
        run: echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 --decode > android/app/keystore.jks

      # Tạo key.properties
      - name: Create key.properties
        run: |
          cat > android/key.properties << EOF
          storePassword=${{ secrets.KEYSTORE_PASSWORD }}
          keyPassword=${{ secrets.KEY_PASSWORD }}
          keyAlias=${{ secrets.KEY_ALIAS }}
          storeFile=keystore.jks
          EOF

      # Build APK release (đã ký)
      - name: Build release APK
        run: flutter build apk --release

      # Build App Bundle (cho Google Play)
      - name: Build App Bundle
        run: flutter build appbundle --release

      # Upload lên GitHub Release
      - name: Upload to GitHub Release
        if: github.event_name == 'release'
        uses: softprops/action-gh-release@v2
        with:
          files: |
            build/app/outputs/flutter-apk/app-release.apk
            build/app/outputs/bundle/release/app-release.aab
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## 5. Secrets — Bảo Mật Thông Tin Nhạy Cảm

Không bao giờ hard-code API key, keystore password trong code. Dùng GitHub Secrets.

```
GitHub Repository → Settings → Secrets and variables → Actions → New secret
```

### Encode keystore thành base64 (chạy local)

```bash
# macOS/Linux
base64 -i android/app/keystore.jks | pbcopy  # copy vào clipboard

# Windows (PowerShell)
[Convert]::ToBase64String([IO.File]::ReadAllBytes("android\app\keystore.jks")) | Set-Clipboard
```

### Các secret thường dùng

| Secret name | Nội dung |
|-------------|----------|
| `KEYSTORE_BASE64` | Keystore file encode base64 |
| `KEYSTORE_PASSWORD` | Mật khẩu keystore |
| `KEY_PASSWORD` | Mật khẩu key alias |
| `KEY_ALIAS` | Tên key alias |
| `CODECOV_TOKEN` | Token từ codecov.io |
| `GOOGLE_PLAY_JSON` | Service account JSON cho Play Store |

### Dùng secret trong workflow

```yaml
- name: Bước có secret
  env:
    MY_API_KEY: ${{ secrets.MY_API_KEY }}
  run: echo "Key bắt đầu bằng ${MY_API_KEY:0:4}..."  # không bao giờ print toàn bộ!
```

---

## 6. Matrix Strategy — Test Nhiều Phiên Bản

```yaml
jobs:
  test:
    strategy:
      matrix:
        flutter-version: ['3.19.0', '3.22.0']
        os: [ubuntu-latest, macos-latest]
      fail-fast: false    # tiếp tục test các matrix khác dù một cái fail

    runs-on: ${{ matrix.os }}

    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ matrix.flutter-version }}
      - run: flutter test
```

---

## 7. Workflow Thực Tế Cho Dự Án Flutter Game

### Chiến lược branch

```
main        ← CD deploy, chỉ nhận merge từ develop
develop     ← CI test + build debug
feature/*   ← CI test mỗi lần push
```

### CI đầy đủ (thực tế)

```yaml
name: CI

on:
  push:
    branches: ['**']
    paths:                           # chỉ chạy khi có thay đổi liên quan
      - 'lib/**'
      - 'test/**'
      - 'assets/**'
      - 'pubspec.yaml'
      - 'pubspec.lock'
      - '.github/workflows/**'
  pull_request:
    branches: [main, develop]

concurrency:                         # hủy run cũ khi có run mới hơn
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 20              # fail nếu chạy quá 20 phút

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.22.0'
          channel: 'stable'
          cache: true

      - name: Restore pub cache
        uses: actions/cache@v4
        with:
          path: ~/.pub-cache
          key: pub-${{ hashFiles('pubspec.lock') }}

      - run: flutter pub get
      - run: flutter pub run build_runner build --delete-conflicting-outputs
      - run: flutter analyze
      - run: flutter test --coverage --reporter=github

      - name: Check for TODO/FIXME in lib/
        run: |
          if grep -r "TODO\|FIXME" lib/ --include="*.dart"; then
            echo "⚠️ Tìm thấy TODO/FIXME — xem xét trước khi merge"
          fi

  build:
    needs: test
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.22.0'
          cache: true

      - run: flutter pub get
      - run: flutter pub run build_runner build --delete-conflicting-outputs
      - run: flutter build apk --debug

      - uses: actions/upload-artifact@v4
        with:
          name: debug-apk-${{ github.sha }}
          path: build/app/outputs/flutter-apk/app-debug.apk
          retention-days: 7
```

---

## 8. Badges — Hiển Thị Trạng Thái Trên README

```markdown
<!-- README.md -->
# My Flutter Game

![CI](https://github.com/username/repo/actions/workflows/ci.yml/badge.svg)
![CD](https://github.com/username/repo/actions/workflows/cd.yml/badge.svg)
[![codecov](https://codecov.io/gh/username/repo/branch/main/graph/badge.svg)](https://codecov.io/gh/username/repo)
```

---

## 9. Tối Ưu Tốc Độ

```yaml
# 1. Cache Flutter SDK (quan trọng nhất — tiết kiệm ~2 phút)
- uses: subosito/flutter-action@v2
  with:
    cache: true

# 2. Cache pub packages
- uses: actions/cache@v4
  with:
    path: ~/.pub-cache
    key: pub-${{ hashFiles('pubspec.lock') }}

# 3. Chỉ chạy khi file liên quan thay đổi
on:
  push:
    paths:
      - 'lib/**'
      - 'test/**'
      - 'pubspec.yaml'

# 4. Cancel run cũ
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

# 5. Dùng ubuntu-latest cho jobs không cần macOS/Windows
# (macOS runner đắt tiền hơn ~10x)
```

---

## 10. Debugging Workflow

```yaml
# In thông tin môi trường để debug
- name: Print environment
  run: |
    flutter --version
    dart --version
    java -version
    echo "OS: ${{ runner.os }}"
    echo "Runner: ${{ runner.name }}"
    echo "Branch: ${{ github.ref_name }}"
    echo "SHA: ${{ github.sha }}"

# Enable debug logging khi chạy
# Trong GitHub → Settings → Secrets → New secret:
# ACTIONS_STEP_DEBUG = true
# ACTIONS_RUNNER_DEBUG = true
```

### Các lỗi thường gặp

| Lỗi | Nguyên nhân | Fix |
|-----|-------------|-----|
| `pubspec.lock not found` | Quên commit lock file | `git add pubspec.lock` |
| `build_runner not found` | Thiếu bước generate code | Thêm step `build_runner build` |
| `Keystore file not found` | Path sai hoặc decode fail | Kiểm tra base64 decode + path |
| `Timeout` | Job chạy quá lâu | Tăng `timeout-minutes` hoặc tối ưu build |
| `macOS minute limit` | Dùng macOS runner quá nhiều | Dùng ubuntu khi có thể |

---

## Bài Tập Thực Hành

**Bài 1 — CI cơ bản:** Tạo workflow chạy `flutter analyze` và `flutter test` mỗi khi push lên bất kỳ branch nào.

**Bài 2 — Build artifact:** Thêm job build APK debug, upload artifact. Download và cài thử trên điện thoại thật.

**Bài 3 — Branch protection:** Cấu hình GitHub để branch `main` chỉ nhận merge khi CI pass (Settings → Branches → Branch protection rules).

**Bài 4 — CD pipeline:** Tạo workflow CD chạy khi tạo GitHub Release mới, build APK release và attach vào release.

---

## Tóm Tắt Toàn Bộ Tech Stack

```
Dart          →  Ngôn ngữ: type-safe, async, null-safety
Flutter       →  UI framework: widgets, animations, themes
Flame         →  Game engine: components, game loop, collision
Riverpod      →  State management: providers, notifiers, testing
Hive          →  Local storage: NoSQL key-value, type-safe
flame_audio   →  Âm thanh: BGM loop, SFX preload, volume control
Rive          →  Animation: state machine, real-time, interactive
GitHub Actions→  CI/CD: test, lint, build, deploy tự động
```

---

*Đây là tài liệu cuối trong series. Bắt đầu build thôi! 🚀*
