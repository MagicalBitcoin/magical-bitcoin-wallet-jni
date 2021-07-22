# bdk-jni

<a href="https://github.com/bitcoindevkit/bdk-jni/actions?query=workflow%3ACI"><img alt="CI Status" src="https://github.com/bitcoindevkit/bdk-jni/workflows/CI/badge.svg"></a>

Install Open JDK 14 or later if not already installed and make sure your `JAVA_HOME` env variable is
set and `java --version` is found and displays the the correct version.

If you haven't installed rust android targets first add those to your environment using [rustup](https://www.rust-lang.org/learn/get-started)
```sh
rustup target add x86_64-apple-darwin x86_64-unknown-linux-gnu x86_64-linux-android aarch64-linux-android armv7-linux-androideabi i686-linux-android
```

Then make sure that you have an Android NDK installed ([our Github CI is using 21.4.7075529](https://github.com/actions/virtual-environments/blob/main/images/macos/macos-10.15-Readme.md), 
and that you have an `ANDROID_NDK_HOME` env variable set before you start building the library. 
Usually, if installed through the `sdkmanager`, your `ANDROID_NDK_HOME` will look more or less like 
this: `/home/<user>/Android/Sdk/ndk/<version>/`.

```
export ANDROID_NDK_HOME=/home/<user>/Android/Sdk/ndk/<NDK version, ie. 21.4.7075529>
```

Build and unit test jvm .jar and build android .aar libraries with:
```sh
./gradlew build
```

Test android .aar library connected unit tests with:
```sh
./gradlew connectedDebugAndroidTest
```

Gradle will build automatically the native library with rust for all 4 platforms using NDK. You can 
choose to build only for a specific platform by setting the env variable `BUILD_TARGETS` to a 
comma-separated list containing one or more of the following items:

* `aarch64`
* `armv7`
* `x86_64`
* `i686`

The output aar library is available at `./android/build/outputs/aar`.

To run the tests first launch a local android emulator from the Android Studio IDE or via the 
command line. If starting from the command line you will also need to set the `ANDROID_SDK_ROOT` 
env variable.
```
export ANDROID_SDK_ROOT=</home/<user>/Android/Sdk or where ever your Sdk is installed>
```

#### Build just the native library

If you only want to build the native library, maybe for one single platform, you can do so with something like:
```sh
CARGO_TARGET_AARCH64_LINUX_ANDROID_LINKER="aarch64-linux-android21-clang" CC="aarch64-linux-android21-clang" cargo build --target=aarch64-linux-android
```

Make sure that the compiler from the NDK is in your PATH

If the library is built in debug mode, there should already be a symlink from 
./target/debug/<target>/libbdk\_jni.so to the `jniLibs` directory, otherwise manually copy the 
shared object.

#### Publish .aar files to local maven repository

Use the below command to publish the jvm .jar and android .aar files generated by this project to 
your local maven2 repository (~/.m2/repository/org/bitcoindevkit/bdkjni/bdk-<jvm or android>):
```sh
./gradlew publishToMavenLocal
```

Once published you will see two new artifacts in your local maven repository:
```sh
~/.m2/repository/org/bitcoindevkit/bdkjni/bdk-jvm/<version>/bdk-jvm-<version>.jar
~/.m2/repository/org/bitcoindevkit/bdkjni/bdk-android/<version>/bdk-android-<version>.aar
~/.m2/repository/org/bitcoindevkit/bdkjni/bdk-android-debug/<version>/bdk-android-debug-<version>.aar
```

When you include the `bdk-android.aar` or `bdk-android-debug.aar` library in a local Android project
the `bdk-jvm.jar` will also be included as a transitive dependency. To include dependencies from your
local maven repository add `mavenLocal()` to the `allprojects` section of your project 
`build.gradle` file:
```gradle
allprojects {
    repositories {
        mavenLocal()
        ...
    }
}
```

At the moment we recommend using the `bdk-jni-debug` version of the library so it can be debugged 
and used locally in the IDE. Because you added your local Maven repository to your `build.gradle` 
file, you can now simply add `bdk-jni` or `bdk-jni-debug` to your `app/build.gradle` dependencies:
```gradle
dependencies {
    implementation 'org.bitcoindevkit.bdkjni:bdk-jni-debug:0.2.0'
    ...
}
```

Note: Once regular release builds are published to a public maven repository these steps will only 
be needed to test local development builds of bdk-jni.
