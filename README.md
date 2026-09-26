<div align="center">

# nexium-discord

**A Discord bot library for Nexium: the gateway, the REST calls a bot makes, and events to `match` on.**<br>
Written in Nexium over `std.http` and `std.websocket`, with TLS from [nxtls](https://github.com/Londopy/nxtls), or the system's own where there are no roots for nxtls to read.

[![CI](https://github.com/Londopy/nexium-discord/actions/workflows/ci.yml/badge.svg)](https://github.com/Londopy/nexium-discord/actions/workflows/ci.yml)
[![Written in Nexium](https://img.shields.io/badge/written%20in-Nexium-7C3AED)](https://github.com/Londopy/nexium)
[![Status: early](https://img.shields.io/badge/status-early-orange)](#status)
[![License: MIT](https://img.shields.io/github/license/Londopy/nexium-discord?color=blue)](LICENSE)

[Use](#use) · [Modules](#modules) · [Tests](#tests) · [Status](#status)

</div>

---

A bot is a loop over `bot.next()`: the events it wants come back as values
to `match` on, and everything the gateway needs to stay connected
(identifying, heartbeats, resuming a dropped session at the URL Discord
gives, reconnecting with growing waits) happens inside. Answers go out
through the REST calls on the same `bot`, with Discord's rate limits
waited out. The gateway's state machine and the message, interaction and
command code come from a Discord bot that has run on them since September
2026.

> [!NOTE]
> **Needs [Nexium](https://github.com/Londopy/nexium) 1.4.0** or later.
> Runs on Linux, macOS, the BSDs and Windows. The TLS is nxtls, over the
> system's trusted roots; Windows keeps those in a store rather than a
> file, so a bot there uses the system's own TLS (SChannel), unless
> `$SSL_CERT_FILE` names a PEM bundle for nxtls.

## Use

In your bot's `nexium.toml`, then `nx fetch`:

```toml
[dependencies]
discord = { git = "https://github.com/Londopy/nexium-discord", tag = "v0.2.0" }
```

A bot that answers `/ping` ([examples/pingbot](examples/pingbot/main.nx)
has it whole, and `!ping` too):

```nexium
import std.json
import discord
import discord.command
import discord.interaction

fn main() -> !void {
    var tls = try discord.layers()                             // the system's trusted roots
    var bot = try discord.from_env(&mut tls, discord.GUILDS)   // the token from $DISCORD_TOKEN
    while true {
        let ev = bot.next() catch |e| {
            eprintln("{}", .{bot.problem})                     // what Discord said
            return e
        }
        match ev {
            .Ready(r) => {
                var defs = List(json.Json).new()
                defs.append(command.command("ping", "Is the bot there?"))
                try bot.register("", &defs)
            }
            .InteractionCreate(ix) => {
                if ix.command == "ping" {
                    let reply = interaction.reply_text("pong")
                    try bot.respond(&ix, reply[..])
                }
            }
            else => { }
        }
    }
}
```

The token is the bot's, from the [Developer
Portal](https://discord.com/developers/applications) (Bot tab); it is sent
only to Discord, and never appears in `bot.problem`.

## Modules

| module | what |
| --- | --- |
| `discord` | `Bot`: `next` (the next event; heartbeats, resumes and reconnects inside), `send` and `post` (messages to a channel), `edit`, `delete`, `react`, `respond` (an interaction's answer, within its three seconds), `followup` and `edit_reply` (after a deferred answer), `register` (slash commands, everywhere or in one server), and `api` for any other REST call, a 429 waited out as long as Discord asks and a 5xx tried again |
| `discord.event` | `Event`, what `next` returns: `Ready`, `Resumed`, `MessageCreate`, `InteractionCreate`, `GuildCreate`, `ReactionAdd`, and `Other` with the name and JSON of any other dispatch |
| `discord.message` | messages from parts: text, embeds with fields and images, rows of buttons, select menus, forms (modals); pings no one unless a user or role is named; Discord's length and count limits applied once, when built |
| `discord.interaction` | slash commands (with subcommands and options), buttons, menu choices, submitted forms and autocomplete as they arrive, and the answers: `reply`, `reply_text`, `deferred`, `update`, `modal`, `choices` |
| `discord.command` | slash command definitions: options of every type, fixed choices, subcommands and groups, autocomplete, commands only for servers, commands hidden from members without a permission |
| `discord.gateway` | the Gateway protocol (v10, JSON) as a state machine with no socket in it: identify, heartbeats with the sequence, resume at `resume_gateway_url`, Reconnect and Invalid Session, a connection that stops acknowledging heartbeats, the close codes no retry fixes; every intent Discord has |
| `discord.layer` | TLS behind `std.http`'s `Transport`: nxtls over the system's trusted roots (`$SSL_CERT_FILE`, or the usual bundles), or the platform's own where there is no bundle (Windows); `discord.system_layers()` takes the platform's everywhere |

## Tests

```sh
nx fetch
for m in js gateway message interaction command event layer lib; do nx test src/$m.nx; done
```

A scripted Discord plays through a transport of its own: a gateway
session from Hello to a dispatch, the connection dropped and the session
resumed at the URL READY named, a refused token, a 429 waited out, a
refusal's words in `problem`, slash commands registered. The gateway's
state machine runs whole sessions on made-up clocks (a heartbeat never
acknowledged, Invalid Session, every close code), and the messages,
interactions and commands are read and written as Discord's JSON has
them. Checked live from Linux and Windows, with no token that works:
`GET /gateway`, and the gateway's handshake, Hello and Identify, ending in
Discord's close 4004 as `bot.problem` says it; on Windows through both
SChannel and nxtls.

## Status

0.2.0 is a bot's everyday: messages, slash commands and their answers,
buttons, menus, forms, reactions. Not yet: sharding (a bot in more than
2,500 servers), voice, the gateway's zlib compression, uploading files,
and the rest of the REST API beyond `api`.

## License

MIT, as [LICENSE](LICENSE) says.
