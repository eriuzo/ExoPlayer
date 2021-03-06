# ExoPlayer VP9 Extension #

## Description ##

The VP9 Extension is a [TrackRenderer][] implementation that helps you bundle libvpx (the VP9 decoding library) into your app and use it along with ExoPlayer to play VP9 video on Android devices.

[TrackRenderer]: https://google.github.io/ExoPlayer/doc/reference/com/google/android/exoplayer/TrackRenderer.html

## Build Instructions (Android Studio and Eclipse) ##

Building the VP9 Extension involves building libvpx and JNI bindings using the Android NDK and linking it into your app. The following steps will tell you how to do that using Android Studio or Eclipse.

* Checkout ExoPlayer along with Extensions

```
git clone https://github.com/google/ExoPlayer.git
```

* Set the following environment variables:

```
cd "<path to exoplayer checkout>"
EXOPLAYER_ROOT="$(pwd)"
VP9_EXT_PATH="${EXOPLAYER_ROOT}/extensions/vp9/src/main"
```

* Download the [Android NDK][] and set its location in an environment variable:

```
NDK_PATH="<path to Android NDK>"
```

* Fetch libvpx and libyuv

```
cd "${VP9_EXT_PATH}/jni" && \
git clone https://chromium.googlesource.com/webm/libvpx libvpx && \
git clone https://git.chromium.org/external/libyuv.git libyuv
```

* Run a script that generates necessary configuration files for libvpx

```
cd ${VP9_EXT_PATH}/jni && \
./generate_libvpx_android_configs.sh "${NDK_PATH}"
```

### Android Studio ###

For Android Studio, we build the native libraries from the command line and then Gradle will pick it up when building your app using Android Studio.

* Build the JNI native libraries

```
cd "${VP9_EXT_PATH}"/jni && \
${NDK_PATH}/ndk-build APP_ABI=all -j4
```

* In your project, you can add a dependency to the VP9 Extension by using a the following rule

```
// in settings.gradle
include ':..:ExoPlayer:library'
include ':..:ExoPlayer:vp9-extension'

// in build.gradle
dependencies {
    compile project(':..:ExoPlayer:library')
    compile project(':..:ExoPlayer:vp9-extension')
}
```

* Now, when you build your app, the VP9 extension will be built and the native libraries will be packaged along with the APK.

### Eclipse ###

* The following steps assume that you have installed Eclipse and configured it with the [Android SDK][] and [Android NDK ][]:
  * Navigate to File->Import->General->Existing Projects into Workspace
  * Select the root directory of the repository
  * Import the following projects:
    * ExoPlayerLib
    * ExoPlayerExt-VP9
    * If you are able to build ExoPlayerExt-VP9 project, then you're all set.
    * (Optional) To speed up the NDK build:
      * Right click on ExoPlayerExt-VP9 in the Project Explorer pane and choose Properties
      * Click on C/C++ Build
      * Uncheck `Use default build command`
      * In `Build Command` enter: `ndk-build -j4` (adjust 4 to a reasonable number depending on the number of cores in your computer)
      * Click Apply

You can now create your own Android App project and add ExoPlayerLib along with ExoPlayerExt-VP9 as a dependencies to use ExoPlayer along with the VP9 Extension.


[Android NDK]: https://developer.android.com/tools/sdk/ndk/index.html
<!---
Work around to point to two different links for the same text.
-->
[Android NDK ]: http://tools.android.com/recent/usingthendkplugin
[Android SDK]: http://developer.android.com/sdk/installing/index.html?pkg=tools

## Building for various Architectures ##

### Android Studio ###

The manual invocation of `ndk-build` will build the library for all architectures and the correct one will be picked up from the APK based on the device its running on.

### Eclipse  ###

libvpx is optimized for various architectures (like neon, x86, etc.). The `generate_libvpx_android_configs.sh` script generates Android configurations for the following architectures:

* armeabi (the default - does not include neon optimizations)
* armeabi-v7a (choose this to enable neon optimizations)
* mips
* x86
* arm64-v8a
* mips64
* x86_64
* all (will result in a larger binary but will cover all architectures)

You can build for a specific architecture in two ways:

* Method 1 (edit `Application.mk`)
  * Edit `${VP9_EXT_PATH}/jni/Application.mk` and add the following line `APP_ABI := <arch>` (where `<arch>` is one of the above 7 architectures)
* Method 2 (pass NDK build flag)
  * Right click on ExoPlayerExt-VP9 in the Project Explorer pane and choose Properties
  * Click on C/C++ Build
  * Uncheck `Use default build command`
  * In `Build Command` enter: `ndk-build APP_ABI=<arch>` (where `<arch>` is one of the above 7 architectures)
  * Click Apply

## Other Things to Note ##

* Every time there is a change to the libvpx checkout:
  * Android config scripts should be re-generated by running `generate_libvpx_android_configs.sh`
  * Clean and re-build the project.
* If you want to use your own version of libvpx or libyuv, place it in `${VP9_EXT_PATH}/jni/libvpx` or `${VP9_EXT_PATH}/jni/libyuv` respectively.

