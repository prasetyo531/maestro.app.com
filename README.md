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

## Prerequisites

- Install the [Maestro CLI](https://docs.maestro.dev/maestro-cli/how-to-install-maestro-cli).
- Install the app on a connected Android device/emulator, or boot an iOS simulator with the app installed.
- Make sure the app exposes useful accessibility labels or stable resource IDs for its controls.

## Configure the app

Edit `.maestro/config.yaml` and replace `com.example.app` with the Android package name or iOS bundle ID.
Then update the `Welcome` assertion in `.maestro/flows/smoke.yaml` to text that is visible on the initial
screen. Keep assertions user-facing and stable; prefer accessibility labels or IDs over screen coordinates.

If Android and iOS use different app IDs, put `appId` in the relevant flow files or keep separate
config files for each platform.

## Run the tests

Run the smoke test:

```sh
maestro test .maestro --include-tags=smoke
```

Run all flows discovered under `.maestro/flows`:

```sh
maestro test .maestro
```

Run a single flow directly:

```sh
maestro test --config .maestro/config.yaml .maestro/flows/smoke.yaml
```

## Add a login flow

Copy `.maestro/examples/login.yaml` into `.maestro/flows/` after changing the button and field labels to
match your app. It uses runtime parameters for credentials, so secrets don't need to live in the YAML:

```sh
maestro test \
  -e TEST_EMAIL="$TEST_EMAIL" \
  -e TEST_PASSWORD="$TEST_PASSWORD" \
  .maestro/flows/login.yaml
```

Use dedicated test accounts and store credentials in your CI secret manager. Don't commit real credentials.

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
