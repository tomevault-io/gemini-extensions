## calls-sdk-flutter

> This repository contains the CometChat Calls SDK sample apps for Flutter. When working with this codebase, use the agent skills in the `skills/` directory for SDK-specific guidance.

# CometChat Calls SDK v5 — Flutter

This repository contains the CometChat Calls SDK sample apps for Flutter. When working with this codebase, use the agent skills in the `skills/` directory for SDK-specific guidance.

## Skills

Load the relevant skill based on the task:

### Core
- `skills/setup/SKILL.md` — SDK dependency from Cloudsmith, CallAppSettings, CometChatCalls.init, permissions
- `skills/join-session/SKILL.md` — CometChatCalls.joinSession, SessionSettingsBuilder, Widget container
- `skills/ringing-integration/SKILL.md` — Dual SDK (Chat + Calls), initiateCall, accept/reject/cancel
- `skills/session-settings/SKILL.md` — All SessionSettingsBuilder options: layouts, session type, audio mode, hide buttons
- `skills/event-listeners/SKILL.md` — SessionStatus, Participant, Media, ButtonClick, Layout listeners
- `skills/call-logs/SKILL.md` — CallLogRequest, fetching and displaying call history

### Migration
- `skills/migration-v4-to-v5/SKILL.md` — Upgrading from v4 to v5, deprecated API mapping

### Advanced
- `skills/recording/SKILL.md` — Auto-start recording, recording events
- `skills/screen-sharing/SKILL.md` — Screen share viewing, presenter status
- `skills/picture-in-picture/SKILL.md` — PiP mode configuration
- `skills/background-handling/SKILL.md` — OngoingCallService, lifecycle management
- `skills/voip-calling/SKILL.md` — VoIP push notifications, CallKit (iOS), ConnectionService (Android)
- `skills/audio-controls/SKILL.md` — Mute/unmute, audio mode switching
- `skills/video-controls/SKILL.md` — Camera on/off, switch camera
- `skills/participant-management/SKILL.md` — Participant list, mute/kick, raise hand
- `skills/custom-ui/SKILL.md` — Custom control panel, hide default UI, overlay controls
- `skills/in-call-chat/SKILL.md` — In-call messaging during active session

## Key Rules

- SDK hosted on Cloudsmith: `https://dart.cloudsmith.io/cometchat/cometchat/`
- `SessionType.audio` / `SessionType.video`
- `LayoutType.tile` / `LayoutType.sidebar` / `LayoutType.spotlight`
- `AudioMode.speaker` / `AudioMode.earpiece` / `AudioMode.bluetooth` / `AudioMode.headphones`
- `CameraFacing.front` / `CameraFacing.back`
- `joinSession` returns a `Widget?` — embed in widget tree
- `CallSession.getInstance()` for session controls after joining
- Language: Dart 3.0+, Flutter 3.10+
- Android minSdk 26, iOS 13.0+
- Documentation: https://www.cometchat.com/docs/calls/flutter/overview

---
> Source: [cometchat/calls-sdk-flutter](https://github.com/cometchat/calls-sdk-flutter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-08 -->
