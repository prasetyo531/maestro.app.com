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

Install the app build you want to test on the emulator. The app-specific smoke flow uses the visible text `Welcome` as a sample assertion; update it in `.maestro/flows/app/smoke.yaml` to match your app.

## Choose the app build

Copy the example environment files, then set each app ID to the matching Android package name. Both app builds must already be installed on the device you use.

```sh
cp .env.debug.example .env.debug
cp .env.prod.example .env.prod
cp .env.contacts.example .env.contacts
```

Edit `MAESTRO_APP_ID` in `.env.debug` and `.env.prod` to your debug and release-ready package IDs. `.env.contacts` points to the Google Contacts app. These local files are ignored by Git; the `.example` files are safe templates. Both app-specific flows use `${MAESTRO_APP_ID}`. `config.yaml` discovers `.maestro/flows/**`; the debug and release scripts exclude the `android-default` tag so the Contacts flow stays separate.

## Run the app-specific test

Install the project helper dependency once:

```sh
npm install
```

This installs `dotenv-cli`, which loads the selected `.env` file before starting Maestro. The Maestro CLI itself must also be installed separately.

```sh
npm test                 # debug build
npm run test:debug       # debug build
npm run test:prod        # release-ready / production build
npm run test:release     # alias for test:prod
npm run test:smoke       # smoke test on debug
npm run test:smoke:prod  # smoke test on release-ready / production
```

The smoke flow is a starter test under `.maestro/flows/app/`; add app-specific flows there when ready. Both build variants use the same flows, selected by `MAESTRO_APP_ID` in `.env.debug` or `.env.prod`.

## Run the Android Contacts test

The Contacts test lives under `.maestro/flows/android-default/` and reads its package ID from `.env.contacts`. Start an emulator with Google Contacts installed, then run:

```sh
npm run test:android:contacts
```

The flow generates a unique first name, last name, and phone number, creates the contact, then checks the name and that a mobile number with Indonesia's `+62` country code was saved. Contacts may reformat the number for display. It leaves the contact on the emulator, so use a disposable emulator or delete the generated contact afterward.

## Troubleshooting

### CLI cannot connect to the Android device

If Maestro CLI reports a driver or gRPC connection error, close Maestro Studio and retry the CLI command. Studio and the CLI can conflict when they try to control the same Android device at the same time. Check that the emulator or device is connected with `adb devices`.

### `Package undefined is not installed`

The flow could not read the package ID. Check that the selected `.env` file exists, contains `MAESTRO_APP_ID=...`, and that dependencies are installed. `dotenv-cli` loads the selected file, and Maestro exposes shell variables with the `MAESTRO_` prefix to flows.

Do not hardcode app IDs in flow files or put them in `.maestro/config.yaml`; use the selected env file instead. If you add another env file, pass it with `dotenv -e <file> -- maestro ...` in `package.json`.

### Save cannot be found after selecting `+62`

Tapping the country code opens the country picker. The picker covers the form, including Save. The Contacts app already selects a country code based on the device, so enter the number in the phone field without opening the picker. The flow taps the number-entry area to the right of `+62`.

### `Phone` cannot be found

After entering the name fields, the keyboard may cover the phone field. Hide it before continuing. The Contacts UI may label the field `Phone (Mobile)`, so selectors must match the label shown by the installed Contacts version.

### Phone number assertion fails even though the contact was created

Contacts formats phone numbers for display, adding spaces or hyphens. An assertion against the raw generated digits may fail. Check the saved contact screen and assert against stable visible details, such as the `+62` country code and `Mobile` label, instead of requiring the raw digits to appear unchanged.

## Run in CI

Install Java, Maestro CLI, Android SDK/platform-tools, and Node.js in the CI job. Run `npm ci`, create the required `.env.debug`, `.env.prod`, or `.env.contacts` file from CI variables, install the matching app artifact, and start an Android emulator before running the test script. The Contacts test requires a Google Android emulator image with Google Contacts installed.
