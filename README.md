
# AndroidPdfViewer (BlueCodeSystems fork)

Android view for displaying PDFs rendered via Pdfium, with gestures, zoom, and double‑tap support.
This fork modernizes the build, publishing, and dependencies for Android SDK 35 and Maven Central.

- Min SDK 28, target/compile SDK 35
- Gradle 8.7, AGP 8.6.0
- Published to Maven Central under `io.github.bluecodesystems`

## Installation

Add repositories and dependency:

```groovy
repositories { mavenCentral() }

dependencies {
  implementation 'io.github.bluecodesystems:AndroidPdfViewer:2.0.0'
}
```

JitPack (alternative):

```groovy
repositories { maven { url 'https://jitpack.io' } }
dependencies {
  implementation 'com.github.BlueCodeSystems:AndroidPdfViewer:2.0.0'
}
```

---
### Original description
# Android PdfViewer

Library for displaying PDF documents on Android, with `animations`, `gestures`, `zoom` and `double tap` support.
It is based on Pdfium for decoding PDF files. Licensed under Apache License 2.0.

## What's new in 3.2.0-beta.1?
* Merge PR #714 with optimized page load
* Merge PR #776 with fix for max & min zoom level
* Merge PR #722 with fix for showing right position when view size changed
* Merge PR #703 with fix for too many threads
* Merge PR #702 with fix for memory leak
* Merge PR #689 with possibility to disable long click
* Merge PR #628 with fix for hiding scroll handle
* Merge PR #627 with `fitEachPage` option
* Merge PR #638 and #406 with fixed NPE
* Merge PR #780 with README fix
* Update compile SDK and support library to 28
* Update Gradle and Gradle Plugin

## Changes in 3.0 API
* Replaced `Contants.PRELOAD_COUNT` with `PRELOAD_OFFSET`
* Removed `PDFView#fitToWidth()` (variant without arguments)
* Removed `Configurator#invalidPageColor(int)` method as invalid pages are not rendered
* Removed page size parameters from `OnRenderListener#onInitiallyRendered(int)` method, as document may have different page sizes
* Removed `PDFView#setSwipeVertical()` method

## Quickstart

Add the view to your layout:

```xml
<com.github.barteksc.pdfviewer.PDFView
    android:id="@+id/pdfView"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>
```

Load a PDF from assets (or file/uri/stream):

```kotlin
val pdfView = findViewById<PDFView>(R.id.pdfView)
pdfView.fromAsset("sample.pdf")
  .enableSwipe(true)
  .swipeHorizontal(false)
  .enableDoubletap(true)
  .defaultPage(0)
  .enableAnnotationRendering(false)
  .spacing(0)
  .pageFitPolicy(FitPolicy.WIDTH)
  .load()
```

See the `sample` module for a working example.

## Requirements

- JDK 17+
- Android SDK 35, Build Tools 35.0.0
- NDK 26.2.11394342 (via `gradle.properties` as `android.ndkVersion`)

## Pdfium dependency

By default this fork depends on `com.github.BlueCodeSystems:PdfiumAndroid:v2.0.0`.
You can override it at build time:

```bash
./gradlew :android-pdf-viewer:assembleRelease \
  -PPDFIUM_GROUP=com.github.BlueCodeSystems \
  -PPDFIUM_ARTIFACT=PdfiumAndroid \
  -PPDFIUM_VERSION=v2.0.0
```

Or set in `gradle.properties`:

```
PDFIUM_GROUP=com.github.BlueCodeSystems
PDFIUM_ARTIFACT=PdfiumAndroid
PDFIUM_VERSION=v2.0.0
```

## Build

- Clean + build: `./gradlew clean build`
- Library AAR (release): `./gradlew :android-pdf-viewer:assembleRelease`
  - Output: `android-pdf-viewer/build/outputs/aar/android-pdf-viewer-release.aar`
- Sample app (debug): `./gradlew :sample:assembleDebug`

## Publish

Local Maven:

```bash
./gradlew :android-pdf-viewer:publishToMavenLocal
```

Maven Central (bundle zip for manual upload):

```bash
./gradlew -PcentralBundle=true -PuseGpgCmd=true \
  :android-pdf-viewer:publishMavenPublicationToCentralBundleRepository \
  :android-pdf-viewer:generateCentralBundleChecksums \
  :android-pdf-viewer:zipCentralBundle
```

Sonatype (staging → release):

```bash
export ORG_GRADLE_PROJECT_sonatypeUsername='TOKEN_USER'
export ORG_GRADLE_PROJECT_sonatypePassword='TOKEN_PASS'
./gradlew :android-pdf-viewer:publish -PuseGpgCmd=true -PcentralRelease=true
./gradlew closeAndReleaseRepository
```

## Notes for AGP 8+

- `namespace` is configured in Gradle. `package` attribute in AndroidManifest is removed and must not be added.

## Changelog

See CHANGELOG.md for release notes. Current version: 2.0.0

## License

Apache License 2.0

## Possible questions
### Why resulting apk is so big?
Android PdfViewer depends on PdfiumAndroid, which is set of native libraries (almost 16 MB) for many architectures.
Apk must contain all this libraries to run on every device available on market.
Fortunately, Google Play allows us to upload multiple apks, e.g. one per every architecture.
There is good article on automatically splitting your application into multiple apks,
available [here](http://ph0b.com/android-studio-gradle-and-ndk-integration/).
Most important section is _Improving multiple APKs creation and versionCode handling with APK Splits_, but whole article is worth reading.
You only need to do this in your application, no need for forking PdfiumAndroid or so.

### Why I cannot open PDF from URL?
Downloading files is long running process which must be aware of Activity lifecycle, must support some configuration, 
data cleanup and caching, so creating such module will probably end up as new library.

### How can I show last opened page after configuration change?
You have to store current page number and then set it with `pdfView.defaultPage(page)`, refer to sample app

### How can I fit document to screen width (eg. on orientation change)?
Use `FitPolicy.WIDTH` policy or add following snippet when you want to fit desired page in document with different page sizes:
``` java
Configurator.onRender(new OnRenderListener() {
    @Override
    public void onInitiallyRendered(int pages, float pageWidth, float pageHeight) {
        pdfView.fitToWidth(pageIndex);
    }
});
```

### How can I scroll through single pages like a ViewPager?
You can use a combination of the following settings to get scroll and fling behaviour similar to a ViewPager:
``` java
    .swipeHorizontal(true)
    .pageSnap(true)
    .autoSpacing(true)
    .pageFling(true)
```

## One more thing
If you have any suggestions on making this lib better, write me, create issue or write some code and send pull request.

## License

Created with the help of android-pdfview by [Joan Zapata](http://joanzapata.com/)
```
Copyright 2017 Bartosz Schiller

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
