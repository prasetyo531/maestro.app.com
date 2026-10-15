# Maestro UI test starter

A small, app-agnostic starter suite for end-to-end UI tests with [Maestro](https://docs.maestro.dev/).
The repository starts without an application binary, so replace the example app ID and screen labels
before running the flows against your app.

## What's included

```text
.maestro/
├── config.yaml             # Shared app ID and flow discovery
├── flows/
│   └── smoke.yaml          # Launch the app and verify the first screen
└── examples/
    └── login.yaml          # Copyable login journey; not included in suite runs
```

The smoke flow is tagged `smoke`. The login example is kept outside the configured `flows/**` path,
so it won't accidentally run before its selectors are adapted to the app.

> **macOS Finder:** `.maestro` starts with a dot, so Finder hides it by default. Press **⌘ + Shift + .**
> while viewing the repository to show or hide dotfiles.

## Install Maestro and prepare a device (macOS)

The CLI is the primary way to run this repository's suite. These steps assume macOS and zsh.
For Windows or Linux, use the official [CLI installation guide](https://docs.maestro.dev/maestro-cli/how-to-install-maestro-cli).

### 1. Install Java 17+

Maestro CLI requires Java 17 or newer. Install a JDK such as [Eclipse Temurin](https://adoptium.net/temurin/releases/),
then check that Java is available:

```sh
java -version
```

If `JAVA_HOME` is not set, add this line to `~/.zshrc` (use `21` if that is the JDK version you installed):

```sh
export JAVA_HOME=$(/usr/libexec/java_home -v 17)
```

Reload the shell with `source ~/.zshrc`, then verify `echo "$JAVA_HOME"` points to the JDK.

### 2. Install the Maestro CLI

Run the official installer:

```sh
curl -fsSL "https://get.maestro.mobile.dev" | bash
```

Open a new terminal if needed, then verify the CLI is on your `PATH`:

```sh
maestro --help
```

### 3. Set up an Android device (if testing Android)

1. Install [Android Studio](https://developer.android.com/studio).
2. In Android Studio, install the Android SDK and **Android SDK Platform-Tools** from **Settings → Languages & Frameworks → Android SDK**.
3. In **Device Manager**, create and start an emulator. A physical device with USB debugging enabled also works.
4. If `adb` is not found, add the SDK platform-tools directory to your `~/.zshrc`. The default SDK path on macOS is:

   ```sh
   export ANDROID_HOME="$HOME/Library/Android/sdk"
   export PATH="$PATH:$ANDROID_HOME/platform-tools"
   ```

5. Reload the shell and check that the device is listed:

   ```sh
   source ~/.zshrc
   adb devices
   ```

### 4. Set up an iOS simulator (if testing iOS)

1. Install and open [Xcode](https://apps.apple.com/app/xcode/id497799835), then complete its first-launch setup.
2. Install Xcode Command Line Tools if needed:

   ```sh
   xcode-select --install
   ```

3. In Xcode, open **Window → Devices and Simulators**, choose **Simulators**, and boot a simulator.
4. Verify Xcode and the simulator are available:

   ```sh
   xcodebuild -version
   xcrun simctl list devices available
   ```

Maestro's documented iOS flow uses simulators. Install or build the app for the simulator before running its test.

### 5. Install the app under test

Maestro automates an installed app. Build a test version of your app, then install it on the running device. For example:

```sh
# Android APK
adb install path/to/app.apk

# iOS Simulator .app bundle
xcrun simctl install booted path/to/App.app
```

Then set its Android package name or iOS bundle ID in `.maestro/config.yaml` and update the `Welcome` assertion in
the smoke flow to text visible on the first screen.

Make sure the app exposes useful accessibility labels or stable resource IDs for its controls.

### Optional: install Maestro Studio

Studio is useful for exploring selectors and debugging a flow visually. Download [MaestroStudio.dmg](https://studio.maestro.dev/MaestroStudio.dmg),
open it, and drag Maestro Studio into **Applications**. Start an emulator or simulator, open the project folder as a
workspace, and select the running device. Use the CLI commands below to run the configured suite: Studio currently
doesn't support the workspace `config.yaml` file.

## Configure the app

Update the `APP_ID` values in `package.json` to the debug and production package names/bundle IDs. Each flow
uses `${APP_ID}`, so the scripts select the matching app without duplicating flows. The corresponding build
must already be installed on the device. Also update the `Welcome` assertion in `.maestro/flows/smoke.yaml`
to text visible on the initial screen. Keep assertions user-facing and stable; prefer accessibility labels
or IDs over screen coordinates.

If Android and iOS also use different IDs, set the matching ID in the npm script you use for that
platform, or add separate platform-specific scripts.

## Run the tests

With Maestro CLI installed, the matching app build installed, and an emulator or simulator running, run the
debug smoke test:

```sh
npm run test:smoke
```

`test:smoke` defaults to the debug app. To select a build explicitly:

```sh
npm run test:smoke:debug
npm run test:smoke:prod
```

Run the full suite against either app:

```sh
npm run test:debug
npm run test:prod
```

`npm test` runs the full suite against debug. These npm scripts are shortcuts for the Maestro CLI; no `npm install`
is needed because the repo has no npm dependencies. To run a single flow directly, pass its app ID with `-e APP_ID=...`.

### Android default Contacts app

The separate Contacts example targets `com.google.android.contacts`, which is available on standard Google Android
emulator images. Start an Android emulator with the app installed, then run:

```sh
npm run test:android:contacts
```

The flow creates a uniquely named synthetic contact and verifies its name and phone number. It leaves that contact
on the device, so use an emulator or remove the test contact afterward. This example is kept outside `flows/**` and
does not run as part of the debug/prod app suites.

## Add a login flow

After changing the button and field labels to match your app, run `.maestro/examples/login.yaml` directly
or copy it into `.maestro/flows/` when it's ready to join the full suite. It uses `APP_ID` plus runtime
parameters for credentials, so secrets don't need to live in the YAML.

Create a local `.env` file from the checked-in template and put your test account credentials in it:

```sh
cp .env.example .env
```

Load those values into the current shell, then pass them to Maestro with `-e`:

```sh
set -a
source .env
set +a

maestro test \
  -e APP_ID=com.yourcompany.yourapp.debug \
  -e TEST_EMAIL="$TEST_EMAIL" \
  -e TEST_PASSWORD="$TEST_PASSWORD" \
  .maestro/examples/login.yaml
```

`.env` is ignored by Git; `.env.example` is safe to commit because it contains only dummy values. Use dedicated
test accounts and store credentials in your CI secret manager. Don't commit real credentials.

## CI and reports

For CI, provide a built app artifact and run the same flows on a suitable simulator/emulator. To run on
hosted devices, Maestro Cloud accepts an app file and a flow path:

```sh
maestro cloud --app-file path/to/app.apk --flows .maestro/flows
```

Use the corresponding iOS Simulator app build for iOS runs. Configure the Maestro Cloud API key as a CI
secret. Maestro can also emit JUnit or HTML reports; see the
[CLI options](https://docs.maestro.dev/maestro-cli/maestro-cli-commands-and-options) for report flags.

## Writing reliable flows

- Give each flow one clear user journey and make it runnable from a clean app state.
- Prefer text, accessibility labels, and resource IDs; use coordinates only when the UI has no stable selector.
- Use assertions after meaningful actions so failures identify the broken step.
- Keep credentials and environment-specific data out of committed flow files; pass them with `-e KEY=value`.
- Add tags such as `smoke`, `auth`, or `checkout` to control which flows run in CI.

See the [Maestro command reference](https://docs.maestro.dev/reference/commands-available) for available
actions and assertions.
