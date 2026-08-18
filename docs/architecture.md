# DeepSeekHarness (Android) — architecture

> 安卓端使用 — Android client for DeepSeek Harness (`dsh`).

## Overview

A phone app that uses an **SSH tunnel as a secure bridge** to bring the server's dsh
web service to the phone, then uses a **full-screen WebView** to use it with all
features — like a remote desktop for your dsh service, but far lighter.

```
phone (display)  ──SSH tunnel──►  server (host)  ──HTTPS/JSON──►  DeepSeek cloud
```

## The four pieces

### 1. Android app (the shell)

- Built-in SSH client (JSch library): phone → server encrypted connection.
- Local port forwarding: everything sent to the phone's `127.0.0.1:6080` is forwarded
  to the server's `127.0.0.1:3080` (the dsh web service).
- A full-screen WebView opens `127.0.0.1:6080` — the port forwarding makes it see the
  real dsh web UI on the server.
- Auto-reconnect on tunnel drop; live status refresh (`DshApi`: events / models /
  presets).

### 2. SSH tunnel (the secure wire)

- Server address / username / password / private key are only used to build this tunnel.
- Fully encrypted end-to-end; a transport-security layer around dsh.
- Auto-reconnects with live status monitoring in the app.

### 3. Server dsh (the brain)

- dsh is a plugin agent platform (cordis architecture + typert RPC), running on the
  server's port 3080.
- The web UI is only the front end; all capabilities (agent loop, model switching,
  plugins, approvals) execute on the server.
- **Model API keys live only on the server** — the phone never touches them, which is
  why the app is safe.

### 4. DeepSeek cloud

- dsh calls the DeepSeek model API for conversation, tool calls and agent reasoning.

## Companion keep-alive

- PC: `login.bat` → hidden-window watchdog → auto-reconnect every 5 s + auto-start on
  boot (Run key).
- Server: `systemd` service (`dsh-web.service`) → start on boot + crash auto-restart.

## Implementation notes (from the v3.4.1 APK)

- Application id: `com.dshtunnel.app` — main activity `MainActivity`.
- Core classes: `SshTunnel` (JSch SSH + port forwarding), `DshApi` (status polling with
  `EventItem` / `ModelInfo` / `PresetInfo` payloads).
- Local forward target string: `127.0.0.1:3080` (server side of the tunnel).
- Libraries: JSch, JZlib, jBCrypt (see THIRD_PARTY_NOTICES.md).

## Pair with the UI adaptation plugin

Install [dsh-client-ui-android](https://github.com/LeMonXi-i/dsh-client-ui-android) on
the server — the web UI then detects the Android device and switches to a phone
layout (drawer sidebar, full-screen settings, single-line composer toolbar), so the
WebView experience is touch-first.
