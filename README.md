# Maestro UI tests

This repository contains a reusable app smoke test that can run against either the debug or release-ready build, plus a separate test for the default Google Contacts app on Android.

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
3. In **Device Manager**, create and start an emulator. For the Contacts test, choose a Google Android image with the Google Contacts app installed.
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

Install the app build you want to test on the emulator. The app-specific smoke flow uses the visible text `Welcome` as a sample assertion; update it in `.maestro/flows/smoke.yaml` to match your app.

## Choose the app build

Update the app IDs in `package.json` to your Android package names. Both app builds must already be installed on the device you use.

```json
"test:debug": "maestro test -e APP_ID=com.yourcompany.yourapp.debug .maestro",
"test:prod": "maestro test -e APP_ID=com.yourcompany.yourapp .maestro"
```

The `APP_ID` is passed to `.maestro/config.yaml` and used by the smoke flow. Change the sample IDs to your debug and release-ready package IDs.

## Run the app-specific test

No `npm install` is needed; `package.json` only provides shortcuts to the Maestro CLI.

```sh
npm test                 # debug build
npm run test:debug       # debug build
npm run test:prod        # release-ready / production build
npm run test:release     # alias for test:prod
npm run test:smoke       # smoke test on debug
npm run test:smoke:prod  # smoke test on release-ready / production
```

The smoke flow is a starter test under `.maestro/flows/`; add app-specific flows there when ready. Both build variants use the same flows, selected by the `APP_ID` passed in the npm script.

## Run the Android Contacts test

The separate test targets `com.google.android.contacts` and is outside `.maestro/flows/`, so it is not included in the debug or release-ready app suite. Start an emulator with Google Contacts installed, then run:

```sh
npm run test:android:contacts
```

The flow generates a unique first name, last name, and phone number, creates the contact, then checks that those values appear. It leaves the contact on the emulator, so use a disposable emulator or delete the generated contact afterward.

## Run in CI

Install Java, Maestro CLI, and Android SDK/platform-tools in the CI job. Install the app artifact matching the selected package ID, start an Android emulator, then run `npm run test:debug` or `npm run test:prod`. The Contacts test requires a Google Android emulator image with Google Contacts installed. No `.env` file or npm dependencies are required.
