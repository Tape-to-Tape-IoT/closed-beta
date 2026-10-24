# Project Overview

What changed since V1, and where things are now. The [README](README.md) covers installing and running.

The docs here are the bare minimum: what you need to set up and run the project. That said, there are a LOT of comments in the code, and each repository has detailed docs:

- **Tape-to-Tape:** [CONFIG_GUIDE.md](https://github.com/Tape-to-Tape-IoT/tape-to-tape/blob/master/CONFIG_GUIDE.md), [TOPICS_REFERENCE.md](https://github.com/Tape-to-Tape-IoT/tape-to-tape/blob/master/TOPICS_REFERENCE.md) and [docs/ARCHITECTURE.md](https://github.com/Tape-to-Tape-IoT/tape-to-tape/blob/master/docs/ARCHITECTURE.md)
- **NHL LED Scoreboard v2:** [docs/](https://github.com/Tape-to-Tape-IoT/nhl-led-scoreboard-v2/tree/main/docs). [docs/t2t](https://github.com/Tape-to-Tape-IoT/nhl-led-scoreboard-v2/tree/main/docs/t2t) also describes the hub's data from the scoreboard's side.

## Coming from V1

- Your V1 `config.json` doesn't carry over. Set things again in each repo's `config/user.toml`.
- V2 needs Raspberry Pi OS Trixie 64-bit, so most of you will start on a fresh SD card.
- What we used to call **boards** are now called **scenes**.
- The rgbmatrix flags are gone. See [Display settings](#display-settings).
- The "new day" setting is now `cleanup.time`. See [End of day](#end-of-day).

## NHL LED Scoreboard v2

It shows the splash scene until it connects to Tape-to-Tape.

### Config

The configs are under `config/`. Setup creates `config/user.toml`.

- `config/default/default.toml` lists every setting with its default. Don't edit it: it's overwritten every time you update.
- Put your changes in `config/user.toml`. Updating never touches it.

### Display settings

We don't use the rgbmatrix flags anymore. All the rgbmatrix settings are under `[display]` in the config.

Most keys are the old flag name without `--led-`, with underscores instead of dashes. For example, `--led-gpio-mapping` is `gpio_mapping` and `--led-slowdown-gpio` is `slowdown_gpio`. Two exceptions:

| V1 flag | V2 `[display]` key |
|-|-|
| `--led-cols` / `--led-rows` | `width` / `height` |
| `--led-no-hardware-pulse` | `disable_hardware_pulsing = true` |

`width` and `height` are still the size of one panel. `chain` and `parallel` tile panels like before.

### Panel sizes and layouts

Anything from 64x32 to 128x64 is supported. I've divided the layout designs into three tiers, picked from the panel's total size (after chaining):

| Tier | Rule | Typical panel |
|-|-|-|
| **Narrow** | 96 px wide or less | 64x32, 64x64 |
| **Wide** | wider than 96 px, under 48 px tall | 128x32 |
| **XL** | wider than 96 px, 48 px tall or more | 128x64 |

I haven't tested any square displays, so keep that in mind. A 64x64 panel gets the Narrow layout. [docs/scenes.md](https://github.com/Tape-to-Tape-IoT/nhl-led-scoreboard-v2/blob/main/docs/scenes.md#responsive-tiers) has the details.

### Scenes

These are the scenes you'll see:

- **Scoreboard:** your team's game
- **Score ticker:** the day's scores
- **Standings**
- **Team summary:** your #1 team
- **Clock**
- **Goal animation**

There's only a goal animation for now. Goal details, the penalty animation and penalty details are coming soon.

These V1 features aren't in V2 right now, so don't report them as missing: weather and weather alerts, the series ticker, Stanley Cup champions, the Christmas board, the pushbutton, the dimmer, the screensaver, and plugins.

You can change some things in scenes with `scenes/scene_config.toml`, the base shared by every panel. Each tier also has an overlay: `scene_config.narrow.toml`, `scene_config.wide.toml` and `scene_config.xl.toml`. Values in your tier's overlay win over the base. [docs/scenes.md](https://github.com/Tape-to-Tape-IoT/nhl-led-scoreboard-v2/blob/main/docs/scenes.md#scene-config-scenecfg) explains the settings.

## Tape-to-Tape

Tape-to-Tape doesn't only feed data to the scoreboard. It's a standalone application your own apps can connect to through Valkey (Redis) and MQTT.

### Config

It uses the same config setup as the scoreboard. Look at `config/default/default.toml` to see what's available, and put what you want to change in `config/user.toml`.

### Connecting your own apps

- **Valkey** is on port 6379 (`valkey_host` and `valkey_port` under `[server]`).
- **MQTT** is off by default. To turn it on, set `enabled = true` under `[mqtt]` in `config/user.toml`. Put the broker's host, port, username and password in `.env` (see `.env.example`).

[TOPICS_REFERENCE.md](https://github.com/Tape-to-Tape-IoT/tape-to-tape/blob/master/TOPICS_REFERENCE.md) lists every Valkey key, pub/sub channel and MQTT topic, with their payloads.

### End of day

The "new day" setting from V1 is now `cleanup.time`, in the hub's `config/user.toml`:

```toml
[cleanup] # end of day
time = "5:00"  # Daily time (HH:MM) the previous day's game data is purged from Valkey
```

It's the Pi's local time. Until then, the hub keeps the previous day's games, so late games stay up past midnight.
