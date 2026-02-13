This is a step-by-step tutorial on how to patch [Port Authority](https://github.com/aaronjwood/PortAuthority) with pull request [#156](https://github.com/aaronjwood/PortAuthority/pull/156) and build an unsigned (debug) APK with version 2.4.5.

### Disclaimer
The steps have only been tested in Debian 12. I will try to make it as easy as possible for users with minimum to no development knowledge, but you need to at least know your way around Linux on a basic level. It's best to setup a quick Debian 12 VM for the sole purpose of building the APK and then disposing it. This way we can ensure a clean environment and reproducibility, otherwise some steps may fail. Experienced users can just copy what's needed.

Assuming you are going to use a VM and then dispose it, I will not provide cleanup steps after a successful build. It's best to use a VM so you don't end up with unnecessary packages and environment modifications on you main system after the fact.

### 1. Install necessary packages

```
sudo apt update
sudo apt install -y git curl unzip zip ca-certificates openjdk-17-jdk
```
### 2. Install and configure SDK (Command Line Tools)

Create the SDK parent directroy.

```
mkdir -p "$HOME/Android/Sdk"
cd "$HOME/Android/Sdk"
```

Download latest Command Line Tools and setup layout.

```
curl -L -o cmdline-tools.zip https://dl.google.com/android/repository/commandlinetools-linux-13114758_latest.zip

mkdir -p cmdline-tools
unzip -q cmdline-tools.zip -d cmdline-tools
mv cmdline-tools/cmdline-tools cmdline-tools/latest

cd ~
```

### 3. Configure environment variables in Bash

Open the rc file with `nano ~/.bashrc`. Then paste the following lines on the bottom of the file.

```
export ANDROID_SDK_ROOT="$HOME/Android/Sdk"
export PATH="$PATH:$ANDROID_SDK_ROOT/cmdline-tools/latest/bin:$ANDROID_SDK_ROOT/platform-tools"
```

To save the changes and exit do `Ctrl+S` and `Ctrl+X`.

Make the changes take effect by sourcing the file.

```
. ~/.bashrc
```

### 4. Install necessary SDK components.

```
yes | sdkmanager --licenses
sdkmanager "platform-tools" "build-tools;34.0.0" "platforms;android-34"
```

### 5. Clone the Port Authority repository and apply PR #156.

Clone and checkout branch.

```
git clone https://github.com/aaronjwood/PortAuthority.git
cd PortAuthority
git checkout development
```

Configure Git so it doesn't complain. No need to modify these values, just copy-paste as is.

```
git config user.name "foo"
git config user.email "foo@bar"
```

Apply the fix.

```
curl -L https://github.com/aaronjwood/PortAuthority/pull/156.patch | git am
```

### 6. Build the APK

```
./gradlew assembleDebug
```

Assuming the build was successful, your APK will be in `/app/build/outputs/apk/free/debug/`. Copy it in your home directory.

```
cp app/build/outputs/apk/free/debug/PortAuthority-2.4.5-free-debug.apk ~/
cd ~
```
### Done

You can now move the file to your phone and install it. This is a debug and not a signed release. It doesn't change something functionally for you but you will see a _Leaks_ app installed alongside and you are not gonna be able to uninstall it (technically not an individual app), just hide it from your drawer. This is the [LeakCanary](https://github.com/square/leakcanary) tool (library) and comes as a debug component, it detects memory leaks.

If you want to go through the steps of creating your own Keystore and signing a release, create an issue and I will create a tutorial for it.
