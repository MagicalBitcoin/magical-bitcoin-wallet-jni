# bdk-jni

![CI](https://github.com/bitcoindevkit/bdk-jni/workflows/CI/badge.svg)

Install [Open JDK 15](https://jdk.java.net/15/) if not already installed.

   ```
   sudo mv openjdk-15_osx-x64_bin.tar.gz /Library/Java/JavaVirtualMachines/
   cd /Library/Java/JavaVirtualMachines/
   sudo tar -xzf openjdk-15_osx-x64_bin.tar.gz
   sudo rm openjdk-15_osx-x64_bin.tar.gz
   ```

   Use `/usr/libexec/java_home` to set JAVA_HOME and PATH environment variables in `.bash_profile` (or where appropriate).

If you haven't installed rust android targets first add those to your environment using [rustup](https://www.rust-lang.org/learn/get-started)

   ```
   rustup target add x86_64-apple-darwin x86_64-unknown-linux-gnu x86_64-linux-android aarch64-linux-android armv7-linux-androideabi i686-linux-android
   ```

Then make sure that you have an Android NDK installed (preferably the latest one), and that you have an `NDK_HOME` env variable set before you start building the library. Usually, if installed through the `sdkmanager`,
your `NDK_HOME` will look more or less like this: `/home/user/Android/Sdk/ndk/<version>/`.

Build android library in `.aar` format with:
```
./gradlew build
```
Gradle will build automatically the native library with rust for all 4 platforms using NDK. You can choose to build only for a specific platform by setting the env variable `BUILD_TARGETS` to a comma-separated list
containing one or more of the following items:

* `aarch64`
* `armv7`
* `x86_64`
* `i686`

The output aar library is available at `./library/build/outputs/aar`.

Then you can run the tests with:

   ```script
   ./gradlew connectedDebugAndroidTest
   ```

#### Build just the native library

If you only want to build the native library, maybe for one single platform, you can do so with something like:

```
CARGO_TARGET_AARCH64_LINUX_ANDROID_LINKER="aarch64-linux-android21-clang" CC="aarch64-linux-android21-clang" cargo build --target=aarch64-linux-android
```

Make sure that the compiler from the NDK is in your PATH

If the library is built in debug mode, there should already be a symlink from ./target/debug/<target>/libbdk\_jni.so to the `jniLibs` directory, otherwise manually copy the shared object.

#### Publish .aar files to local maven repository

Use the below command to publish the .aar files generated by this project to your local maven2
repository (~/.r2/repository):

    ```script
    ./gradlew publishToMavenLocal
    ```

Once published you will see two new artifacts in your local maven repository:

   ```
   ~/.m2/repository/org/bitcoindevkit/bdkjni/bdk-jni-debug/<version>/bdk-jni-debug-<version>.aar
   ~/.m2/repository/org/bitcoindevkit/bdkjni/bdk-jni/<version>/bdk-jni-<version>.aar
   ```

You can then include the bdk.aar or bdk-debug.aar library in a local Android project by first
adding `mavenLocal()` to your `build.gradle` repositories section:

    ```
    repositories {

       mavenLocal()
       ...
    }
    ```


And then adding `bdk-jni` or `bdk-jni-debug` to your `app/build.gradle` dependencies:

    ```
    dependencies {
       implementation 'org.bitcoindevkit.bdkjni:bdk-jni:0.1.0-beta.1'
       ...
    }
    ```

Note: Once regular release builds are published to a public maven repository these steps will only
be needed to test local development builds of bdk-jni.
