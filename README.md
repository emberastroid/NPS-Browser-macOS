# NPS Browser for macOS

> A Swift 5 macOS client for browsing, downloading, and managing PS Vita content packages.
> Tested and working on **macOS 10.14+ — Intel and Apple Silicon**.

![Main Screenshot](/Screenshots/main.png?raw=true)

---

## Features

- **Multi-select downloads** — Cmd+click or Shift+click to queue multiple items, download all at once
- **Resumable downloads** — pause, stop, and resume at any point; survives app restarts
- **Bookmark support** — save titles via the Favourites checkbox in the details panel; downloads can be started directly from the bookmark list
- **Compatibility pack support** — FW 3.61+ compat packs supported
- **Always-latest game updates** — update URLs are resolved to the most recent version automatically
- **Game artwork** — cover art displayed in the details panel
- **System notifications** — notified when a download starts and when a full batch completes
- **Simplified Chinese localisation**

---

## Usage

1. Open **Preferences** to set your NPS database URLs and extraction path
2. Select **Database › Reload** (or press **⌘R**) to fetch the latest listings
3. Compatibility pack URLs must point to a raw `.txt` file

---

## Building

> **Note:** Due to Xcode 26 dropping support for old deployment targets, the standard `carthage bootstrap` will fail without patching. Follow the steps below exactly.

### Prerequisites

- Xcode 14 or later (tested up to Xcode 26.4)
- [Carthage](https://github.com/Carthage/Carthage) installed (`brew install carthage`)

### Step 1 — Check out dependencies

```bash
carthage bootstrap --platform macOS --no-use-binaries --cache-builds
```

This will fail partway through on Xcode 15+. That's expected — proceed to Step 2.

### Step 2 — Patch deployment targets

Xcode 15+ removed `libarclite` for old deployment targets. Bump all dependencies to supported minimums:

```bash
find Carthage/Checkouts -name "project.pbxproj" | while read f; do
  sed -i '' 's/IPHONEOS_DEPLOYMENT_TARGET = [0-9.]*/IPHONEOS_DEPLOYMENT_TARGET = 12.0/g' "$f"
  sed -i '' 's/MACOSX_DEPLOYMENT_TARGET = 10\.[0-9]*/MACOSX_DEPLOYMENT_TARGET = 10.13/g' "$f"
done
```

### Step 3 — Build dependencies

```bash
carthage build --platform macOS --no-use-binaries
```

### Step 4 — Fix iOS-only framework slices

Several dependencies (Fuzi, Promises, FBLPromises, SwiftyBeaver, SwiftyUserDefaults, RealmSwift) build iOS-only slices when invoked through Carthage. Build them directly with the macOS SDK:

```bash
cd Carthage/Checkouts

xcodebuild -project Fuzi/Fuzi.xcodeproj \
  -scheme Fuzi -configuration Release -sdk macosx \
  ONLY_ACTIVE_ARCH=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY= build

xcodebuild -project promises/Promises.xcodeproj \
  -scheme Promises -configuration Release -sdk macosx \
  ONLY_ACTIVE_ARCH=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY= build

xcodebuild -project promises/Promises.xcodeproj \
  -scheme FBLPromises -configuration Release -sdk macosx \
  ONLY_ACTIVE_ARCH=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY= build

xcodebuild -project SwiftyBeaver/SwiftyBeaver.xcodeproj \
  -scheme SwiftyBeaver-Package -configuration Release -sdk macosx \
  ONLY_ACTIVE_ARCH=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY= build

xcodebuild -project SwiftyUserDefaults/SwiftyUserDefaults.xcodeproj \
  -scheme SwiftyUserDefaults -configuration Release -sdk macosx \
  ONLY_ACTIVE_ARCH=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY= build

xcodebuild -workspace realm-swift/Carthage/Realm.xcworkspace \
  -scheme RealmSwift -configuration Release -sdk macosx \
  ONLY_ACTIVE_ARCH=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY= build

xcodebuild -project Alamofire/Alamofire.xcodeproj \
  -scheme "Alamofire macOS" -configuration Release -sdk macosx \
  ONLY_ACTIVE_ARCH=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY= build
```

Then copy the correctly-built macOS frameworks into `Carthage/Build/Mac`:

```bash
cd ../..  # back to project root

for dep in Fuzi Promises FBLPromises SwiftyBeaver SwiftyUserDefaults RealmSwift Realm Alamofire; do
  src=$(find ~/Library/Developer/Xcode/DerivedData -name "$dep.framework" \
    -path "*/Products/Release/*" ! -path "*ios*" ! -path "*simulator*" 2>/dev/null | head -1)
  if [ -n "$src" ]; then
    echo "Copying $dep"
    rm -rf Carthage/Build/Mac/$dep.framework
    cp -R "$src" Carthage/Build/Mac/
  else
    echo "NOT FOUND: $dep"
  fi
done
```

### Step 5 — Copy minizip headers

The Zip framework requires minizip headers to be present manually:

```bash
mkdir -p Carthage/Build/Mac/Zip.framework/Headers/minizip
cp Carthage/Checkouts/Zip/Zip/minizip/*.h \
   Carthage/Build/Mac/Zip.framework/Headers/minizip/
cp Carthage/Checkouts/Zip/Zip/minizip/module.modulemap \
   Carthage/Build/Mac/Zip.framework/Headers/minizip/
```

### Step 6 — Build the app

```bash
xcodebuild \
  -project "NPS Browser.xcodeproj" \
  -scheme "NPS Browser" \
  -configuration Debug \
  -destination 'platform=macOS' \
  build
```

Or open `NPS Browser.xcodeproj` in Xcode and press **⌘B**.

### Archiving for distribution

```bash
xcodebuild \
  -project "NPS Browser.xcodeproj" \
  -scheme "NPS Browser" \
  -configuration Release \
  -destination 'platform=macOS' \
  archive \
  -archivePath build/NPS-Browser.xcarchive
```

Or in Xcode: **Product › Archive › Export**.

---

## Removal

After moving the app to Trash, run the following to remove all app data:

```bash
rm -r ~/Library/Application\ Support/JK3Y.NPS-Browser/
rm -r ~/Library/Caches/JK3Y.NPS-Browser
rm -r ~/Library/Caches/NPS\ Browser
defaults delete JK3Y.NPS-Browser
```

---

## App Icon

Current icon by **iigiovanni**, sourced from [macOSicons](https://macosicons.com).
Original icon by **Ann0ying**.

---

## Credits

- [Luro02](https://github.com/Luro02/pkg2zip) — pkg2zip fork
- [devnoname120](https://github.com/devnoname120/vitanpupdatelinks) — vitanpupdatelinks
- **L1cardo** — Simplified Chinese translation

---

[Changelog](CHANGELOG.md) · [Carthage](https://github.com/Carthage/Carthage) · [pkg2zip](https://github.com/Luro02/pkg2zip) · [vitanpupdatelinks](https://github.com/devnoname120/vitanpupdatelinks)
