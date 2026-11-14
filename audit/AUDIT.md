# Moblin audit (2026-11-13)

Source: https://github.com/eerimoq/moblin, tag `ios-40.1.0-497` (ba625a7df).
Branch `main` = tag + `audit/0001-relay-off-by-default.patch`.

## Findings

- **No analytics, telemetry or crash-reporting libraries.** None in Package.resolved or in the Swift code.
- **Only one connection starts by itself:** the Remote Control relay
  (`wss://moblin.mys-lang.org/moblin-remote-control-relay/bridge/control/<random uuid>`).
  - `SettingsRemoteControlServerRelay.enabled` defaults to `true` on a fresh install.
  - `Model.reloadConnections()` calls `reloadRemoteControlRelay()` at launch.
  - It only listens: it sends no data, and the developer's server sees your public IP and a random ID.
  - If a "connect" message arrives, it bridges to the local assistant at `ws://localhost:<port>`.
    That assistant runs only if Remote Control > Assistant is enabled and a password is set.
  - This is the `moblin.mys-lang.org` entry in the iPhone App Privacy Report.
- **Off unless you configure them:** Moblin Cloud (enabled=false), NTP (timecodes off),
  chats (Twitch/Kick/YouTube), TTS (ElevenLabs/TTS Monster), AI chat (Gemini).
- **Video** goes only to the stream URL you enter. SRT supports AES with `passphrase` and `pbkeylen` in the URL query.
- `NSAllowsArbitraryLoads` = true in Info.plist, because plain RTMP and WS to user servers need it.
- 25 Swift packages; 10 of them are eerimoq forks or repos.
  Packages were not audited line by line.

## App Store version (without rebuilding)

Remote Control > Assistant > Relay > Enabled = OFF.

## Own build

```bash
git clone https://github.com/eerimoq/moblin && cd moblin
git checkout -b local-audit ios-40.1.0-497
git apply 0001-relay-off-by-default.patch  
cp User.template.xcconfig User.xcconfig      # check the README for what to fill in
open Moblin.xcodeproj
```
In Xcode: Signing & Capabilities > your Team, unique bundle ID, select the iPhone, then Run.
With a free Apple ID the app expires after 7 days, so rebuild weekly.

## Stream settings

URL: `srt://<receiver-ip>:9000?passphrase=<secret>&pbkeylen=32`, HEVC, 4K30 or 1080p60, 30–40 Mbps.
