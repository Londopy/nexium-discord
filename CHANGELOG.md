# Changelog

All notable changes to this package are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and versions
follow [Semantic Versioning](https://semver.org/).

## [Unreleased]

## [0.2.0] - 2026-09-26

### Added

- Windows. nxtls 0.5.0 reads its randomness from Nexium 1.4's
  `random.secure`, and where the system keeps no bundle of trusted roots
  for nxtls to read (Windows, unless `$SSL_CERT_FILE` names one),
  `discord.layers()` gives the platform's own TLS instead: `std.http`'s
  `SystemTls`, SChannel on Windows, which checks certificates against the
  system's store. Checked live on Windows through both.
- `discord.system_layers()`: layers over the platform's TLS on any system
  (SChannel, Security.framework, OpenSSL's libssl).

### Changed

- Needs Nexium 1.4.0, the release; CI runs on it, on Linux, Windows and
  macOS, where it built a commit of Nexium's main branch before.
- nxtls 0.5.0, from 0.4.0.

## [0.1.0] - 2026-09-25

### Added

- `Bot`: the gateway session behind `next`, which returns the next event
  and keeps the connection alive (identify, heartbeats with the sequence,
  resume at `resume_gateway_url`, reconnects with waits growing to a
  minute), and stops with `error.Refused` and a sentence in `problem` on a
  close no retry fixes. REST calls: `send`, `post`, `edit`, `delete`,
  `react`, `respond`, `followup`, `edit_reply`, `register`, and `api` for
  any other, a 429 waited out as long as Discord asks and a 5xx or a
  failed connection tried again.
- `discord.event`: `Ready`, `Resumed`, `MessageCreate`,
  `InteractionCreate`, `GuildCreate`, `ReactionAdd`, and `Other`.
- `discord.message`, `discord.interaction`, `discord.command`: messages
  with embeds, buttons, menus and forms; interactions read and answered;
  slash command definitions.
- `discord.gateway`: the Gateway protocol (v10, JSON) as a state machine,
  and every intent.
- `discord.layer`: nxtls in `std.http`'s TLS slot, over the system's
  trusted roots.
