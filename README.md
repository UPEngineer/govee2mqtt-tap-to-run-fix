# govee2mqtt with the Tap-to-Run fix (Home Assistant add-on)

A temporary Home Assistant app (add-on) for [govee2mqtt](https://github.com/wez/govee2mqtt) with a fix for
[wez/govee2mqtt#737](https://github.com/wez/govee2mqtt/issues/737): Tap-to-Run / one-click scenes fail with
`authorization token is invalid` and new Tap-to-Runs don't appear after Purge Caches.

Since early November 2026, Govee rejects the community-api token on the Tap-to-Run endpoint. This patch
fetches Tap-to-Runs with the main account token instead, the same fix homebridge-govee made.

**Use this only until an official govee2mqtt release includes the fix**, then switch back to the official app.

## How it works

There's no prebuilt image. When you install it, Home Assistant downloads the govee2mqtt source (pinned
to commit `6f5a844`), applies `tap-to-run-account-token.patch` and compiles it on your own machine.
You can read everything it does in [`govee2mqtt_patched/Dockerfile`](govee2mqtt_patched/Dockerfile).

- Supports **amd64** (Intel/AMD) and **aarch64** (Raspberry Pi 4/5 and other 64-bit ARM).
- The first install compiles Rust: about **10–20 minutes on a PC**, and an hour or more on a Raspberry Pi.
  The install page shows a spinner the whole time. That's normal.

## Install

1. In Home Assistant, go to **Settings → Apps → App store**.
2. Click **⋮ → Repositories**, add `https://github.com/YOUR-GITHUB-USERNAME/govee2mqtt-tap-to-run-fix`, and click **Add**.
3. Find **Govee to MQTT Bridge (Tap-to-Run fix)** in the store and click **Install**. Wait for the build.
4. On its **Configuration** tab, enter the same values as your official Govee to MQTT Bridge app
   (Govee email, password, API key, and MQTT settings if you set them there).
5. **Stop the official Govee to MQTT Bridge app** and turn off its *Start on boot*. Running both
   publishes the same devices twice.
6. Start this app and turn on *Start on boot*. Your entities keep their names, since it uses the same MQTT topics.

Check the **Log** tab: the `exec-plat/home ... 401 authorization token is invalid` lines should be gone,
and your `scene.govee_to_mqtt_one_click_*` scenes should be available. After adding a new Tap-to-Run in the
Govee app, press **Purge Caches** on the Govee2MQTT device in Home Assistant.

## Going back to the official version

When a govee2mqtt release includes the fix: stop and uninstall this app, then update the official
Govee to MQTT Bridge app, turn *Start on boot* back on and start it.
