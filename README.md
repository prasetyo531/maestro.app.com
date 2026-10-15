# Maestro UI tests

Starter UI smoke test suite for Android and iOS apps using [Maestro](https://docs.maestro.dev/).

## First flow

`.maestro/flows/smoke.yaml` launches the app and checks that the home screen shows `Welcome`.
Before running it, replace `com.example.app` with the app's Android package name or iOS bundle ID,
and replace `Welcome` with text that is visible on the app's initial screen.

## Run

Install the [Maestro CLI](https://docs.maestro.dev/get-started/installing-maestro), connect an
Android device/emulator or boot an iOS simulator, then run:

```sh
maestro test .maestro/flows/smoke.yaml
```

To run every flow in this repository:

```sh
maestro test .maestro/flows
```
