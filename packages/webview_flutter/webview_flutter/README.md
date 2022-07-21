# Flutter WebView Pro

[![pub package](https://img.shields.io/badge/pub-v3.0.1+3-orange)](https://pub.dartlang.org/packages/webview_flutter)

A Flutter plugin that provides a WebView widget.

On iOS the WebView widget is backed by a [WKWebView](https://developer.apple.com/documentation/webkit/wkwebview);
On Android the WebView widget is backed by a [WebView](https://developer.android.com/reference/android/webkit/WebView).

A Flutter plugin that provides a WebView widget who Support photo upload/take camera and Geolocation.
The official flutter plugin `webview_flutter` Android does not support H5 file upload, that is, it does not support the H5 code below.

```dart
<input type="file">
```

This caused us a lot of inconvenience, so this plugin adds support for file upload and geolocation on the android side on the basis of the official plugin.

## Usage

Add `webview_flutter` as a [dependency in your pubspec.yaml file](https://flutter.dev/docs/development/platform-integration/platform-channels). If you are targeting Android, make sure to read the _Android Platform Views_ section below to choose the platform view mode that best suits your needs.

You can now include a WebView widget in your widget tree. See the
[WebView](https://pub.dev/documentation/webview_flutter/latest/webview_flutter/WebView-class.html)
widget's Dartdoc for more details on how to use the widget.

### 1.Installing

Add this to your package's `pubspec.yaml` file:

```yaml
dependencies:
  # if you Flutter >=2.5 and <2.8, depend this
  webview_flutter: ^3.0.1+3

  # if you Flutter >=2.8 , depend this
  webview_flutter:
    git:
      url: https://github.com/wenzhiming/flutter-plugins.git
      ref: dev-3.0.4
      path: packages/webview_flutter/webview_flutter
```

### 2.Import

```dart
import 'package:webview_flutter/webview_flutter.dart';
```

### 3.How to use

```dart
      body: Builder(builder: (BuildContext context) {
        return WebView(
          initialUrl: 'https://www.xxxxxxx',
          javascriptMode: JavascriptMode.unrestricted,
          onWebViewCreated: (WebViewController webViewController) {
            _controller.complete(webViewController);
          },
          onProgress: (int progress) {
            print("WebView is loading (progress : $progress%)");
          },
          javascriptChannels: <JavascriptChannel>{
            _toasterJavascriptChannel(context),
          },
          navigationDelegate: (NavigationRequest request) {
            if (request.url.startsWith('https://www.youtube.com/')) {
              print('blocking navigation to $request}');
              return NavigationDecision.prevent;
            }
            print('allowing navigation to $request');
            return NavigationDecision.navigate;
          },
          onPageStarted: (String url) {
            print('Page started loading: $url');
          },
          onPageFinished: (String url) {
            print('Page finished loading: $url');
          },
          gestureNavigationEnabled: true,
          geolocationEnabled: false,//support geolocation or not
        );
      }),
```

## Android Platform Views

This plugin uses
[Platform Views](https://flutter.dev/docs/development/platform-integration/platform-views) to embed
the Android’s webview within the Flutter app. It supports two modes:
_hybrid composition_ (the current default) and _virtual display_.

Here are some points to consider when choosing between the two:

- _Hybrid composition_ has built-in keyboard support while _virtual display_ has multiple
  [keyboard issues](https://github.com/flutter/flutter/issues?q=is%3Aopen+label%3Avd-only+label%3A%22p%3A+webview-keyboard%22).
- _Hybrid composition_ requires Android SDK 19+ while _virtual display_ requires Android SDK 20+.
- _Hybrid composition_ and _virtual display_ have different
  [performance tradeoffs](https://flutter.dev/docs/development/platform-integration/platform-views#performance).

### Using Hybrid Composition

The mode is currently enabled by default. You should however make sure to set the correct `minSdkVersion` in `android/app/build.gradle` if it was previously lower than 19:

```groovy
android {
    defaultConfig {
        minSdkVersion 19
    }
}
```

### Using Virtual displays

1. Set the correct `minSdkVersion` in `android/app/build.gradle` (if it was previously lower than 20):

   ```groovy
   android {
       defaultConfig {
           minSdkVersion 20
       }
   }
   ```

2. Set `WebView.platform = AndroidWebView();` in `initState()`.
   For example:

   ```dart
   import 'dart:io';

   import 'package:webview_flutter/webview_flutter.dart';

   class WebViewExample extends StatefulWidget {
     @override
     WebViewExampleState createState() => WebViewExampleState();
   }

   class WebViewExampleState extends State<WebViewExample> {
     @override
     void initState() {
       super.initState();
       // Enable virtual display.
       if (Platform.isAndroid) WebView.platform = AndroidWebView();
     }

     @override
     Widget build(BuildContext context) {
       return WebView(
         initialUrl: 'https://flutter.dev',
       );
     }
   }
   ```

### Enable Material Components for Android

To use Material Components when the user interacts with input elements in the WebView,
follow the steps described in the [Enabling Material Components instructions](https://flutter.dev/docs/deployment/android#enabling-material-components).

### Setting custom headers on POST requests

Currently, setting custom headers when making a post request with the WebViewController's `loadRequest` method is not supported on Android.
If you require this functionality, a workaround is to make the request manually, and then load the response data using `loadHTMLString` instead.
