# Maestro Android Contacts test

One Maestro UI test that creates a contact in the default Google Contacts app on an Android emulator and verifies the saved name and phone number.

> **macOS Finder:** `.maestro` starts with a dot, so Finder hides it by default. Press **⌘ + Shift + .** while viewing the repository to show or hide dotfiles.

## Install and prepare

These steps assume macOS and zsh. For Windows or Linux, follow the official [Maestro CLI installation guide](https://docs.maestro.dev/maestro-cli/how-to-install-maestro-cli).

### 1. Install Java 17 or newer

Install a JDK such as [Eclipse Temurin](https://adoptium.net/temurin/releases/) and verify it:

```sh
java -version
```

If `JAVA_HOME` is not set, add this to `~/.zshrc` (use the version you installed):

```sh
export JAVA_HOME=$(/usr/libexec/java_home -v 17)
```

Then run `source ~/.zshrc` and check `echo "$JAVA_HOME"`.

### 2. Install Maestro CLI

Run the official installer, open a new terminal if needed, and verify the CLI:

```sh
curl -fsSL "https://get.maestro.mobile.dev" | bash
maestro --help
```

### 3. Install Android Studio and start an emulator

1. Install [Android Studio](https://developer.android.com/studio).
2. In **Settings → Languages & Frameworks → Android SDK**, install the Android SDK and **Android SDK Platform-Tools**.
3. In **Device Manager**, create and start a Google Android emulator. The test needs the Google Contacts app (`com.google.android.contacts`) installed; choose an emulator image that includes Google apps.
4. If `adb` is not found, add the SDK tools to `~/.zshrc`:

   ```sh
   export ANDROID_HOME="$HOME/Library/Android/sdk"
   export PATH="$PATH:$ANDROID_HOME/platform-tools"
   ```

5. Reload the shell and confirm the emulator is listed:

   ```sh
   source ~/.zshrc
   adb devices
   ```

## Run the test

No `npm install` is needed; `package.json` only provides shortcuts to the Maestro CLI.

```sh
npm test
```

Or run the single flow directly:

```sh
npm run test:android:contacts
```

The flow generates a unique first name, last name, and phone number, creates the contact, then checks that those values appear. It leaves the contact on the emulator, so use a disposable emulator or delete the generated contact afterward.

## Run in CI

Install Java, Maestro CLI, and Android SDK/platform-tools in the CI job, start an Android emulator image with Google apps and Contacts installed, then run `npm test`. No `.env` file or npm dependencies are required for this test.
