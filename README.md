# DeepSeekHarness (Android)

> **安卓端使用** — An Android client for [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) (`dsh`).

A thin, secure Android shell that turns your phone into a remote display for your own `dsh` server:

- **SSH tunnel** (JSch) carries an encrypted connection from the phone to your server,
- a **full-screen WebView** renders the *real* dsh web GUI — 100% of the features (agent loop, model switching, plugins, approvals, trajectories, settings),
- the model API keys **never leave the server**.

One sentence: **phone = display, server = host, SSH = the secure wire.** It wraps the connection, not the content.

English | [中文](README.zh.md)

## Why this architecture

| Approach | Problem |
| --- | --- |
| Native chat re-write (v2.x) | dozens of features (approvals, plugins, trajectories, model settings…) can never be fully re-implemented |
| WebView → direct connection (v3.x) | ✅ 100% of the web features; the app only owns connection + shell |

## How it works

```
┌───────────────┐   SSH tunnel (encrypted)   ┌──────────────────┐   HTTPS/JSON   ┌────────────────┐
│ Android App   │ ─────────────────────────► │ Server: dsh web  │ ─────────────► │ DeepSeek API   │
│ (JSch +       │   phone 127.0.0.1:6080     │ (127.0.0.1:3080) │   (models,     │ (cloud)        │
│  WebView)     │   ──► server 127.0.0.1:3080│  [cordis + typert]│   tools, etc.) │                │
└───────────────┘                            └──────────────────┘                └────────────────┘
```

1. The app (package `com.dshtunnel.app`) opens an SSH connection to your server with
   JSch and forwards the phone's `127.0.0.1:6080` to the server's `127.0.0.1:3080`.
2. A full-screen WebView opens `127.0.0.1:6080` — it is looking at your real dsh web UI.
3. The tunnel auto-reconnects; the app watches the dsh API (`DshApi`: events, models,
   presets) to refresh the connection status live.

### Companion keep-alive

- **PC side**: `login.bat` → hidden-window watchdog → auto-reconnect every 5 s + auto-start on boot (Run key).
- **Server side**: `systemd` service (`dsh-web.service`) → start on boot + crash auto-restart.

### Security

- Everything is end-to-end encrypted by the SSH tunnel.
- Model API keys are stored **only on the server**; the phone never touches them.
- Add the [dsh-client-ui-android](https://github.com/LeMonXi-i/dsh-client-ui-android)
  plugin on the server and the web UI adapts to a phone layout automatically
  (drawer sidebar, full-screen settings, single-line composer toolbar).

## Download

Grab the latest APK from [Releases](../../releases) (or `releases/` in this repo):

- `releases/DeepSeekHarness-v3.4.1.apk` — Android 7+ (API 24+), ~1.9 MB.

> Requires an Android device. Not available for iOS — use any browser on iOS to open
> the web UI directly.

## Usage

1. Install the APK on your Android phone.
2. Enter your server address, SSH port, username and password / private key.
3. Tap connect — the tunnel is established and the dsh GUI loads in the full-screen
   WebView.
4. If the tunnel drops, the app reconnects automatically.

## Build from source

> Source code will be published in a follow-up commit (the current release ships the
> compiled APK). The app is a standard Android Studio project: Kotlin, JSch for SSH,
> WebView for rendering, and `DshApi` for status polling.

Third-party libraries: [JSch](http://www.jcraft.com/jsch/) (BSD), JZlib (BSD),
jBCrypt (ISC) — see [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

## Related

- [dsh-client-ui-android](https://github.com/LeMonXi-i/dsh-client-ui-android) — the
  Android/mobile adaptation plugin for the dsh web GUI (use together).

## License

MIT — see [LICENSE](LICENSE).
