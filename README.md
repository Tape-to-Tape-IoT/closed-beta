# Tape to tape and NHL LED Scoreboard v2 closed beta

# Project Overview

A lot of things have changed and some things are still missing. Please
read [PROJECT_OVERVIEW.md](PROJECT_OVERVIEW.md) to know more.

# What I'm working on
- Add Goal details scene
- Add Penalty scene
- Add Penalty details scene
- move scene configs into the config folder and create a default and user config for each of them.

## What you're testing

Two programs that run together on one Raspberry Pi:

* **Tape-to-Tape (the hub)** is the backend. It follows the NHL for your favourite teams: live scores, goals, penalties and standings. It saves everything to a local database (Valkey) for other devices to read.
* **NHL LED Scoreboard v2** is the client and reads from the hub and shows it on an LED panel. It shows your team's games, with a goal animation. Between games it rotates through the day's scores, the standings, your team's summary and a clock.

Only the hub talks to the NHL, so you install it first. Your favourite teams are set in the hub.

# Closed Beta Install Guide

## Before you start

**You need:**

* A Raspberry Pi Zero 2 W, 3, 4 or 5
* Raspberry Pi OS **Trixie, 64-bit** (Lite recommended). Bookworm and 32-bit won't work.
* A HUB75 LED panel (128x64, 128x32 or 64x32) with a HAT or bonnet. The defaults assume Adafruit's.
* A separate 5V power supply for the panel
* The GitHub account you gave us

**Accept both GitHub invites.** They expire after 7 days. If one has, ask us for a new one.

* [tape-to-tape](https://github.com/Tape-to-Tape-IoT/tape-to-tape)
* [nhl-led-scoreboard-v2](https://github.com/Tape-to-Tape-IoT/nhl-led-scoreboard-v2)

Everything below runs in an SSH session on the Pi. Check the OS first:

```bash
grep VERSION_CODENAME /etc/os-release   # must say trixie
uname -m                                # must say aarch64
```

## Step 1: Sign in to GitHub

```bash
sudo apt update && sudo apt install -y git gh
gh auth login
```

1. Answer the prompts: **GitHub.com**, **HTTPS**, **Yes**, **Login with a web browser**.
2. Note the code it shows (like `35C2-643E`) and press **Enter**.
3. It says it failed to open a browser. That's expected, so leave it running.
4. On your phone or computer, open [github.com/login/device](https://github.com/login/device), enter the code and approve.
5. The Pi then says `✓ Logged in as <you>`. The warning about credentials saved in plain text is normal on a Pi.

Don't press Ctrl+C while it waits. If you did, run `gh auth login` again.

## Step 2: Install the hub

```bash
cd ~
gh repo clone Tape-to-Tape-IoT/tape-to-tape
cd tape-to-tape
./setup.sh --service
```

This installs the hub and Valkey, and starts the hub at boot. It asks for your `sudo` password.

Then set your favourite teams:

```bash
nano config/user.toml
```

Uncomment the `[teams]` lines and list your teams, #1 first:

```toml
[teams]
favorites = ["MTL", "TOR"]
```

Your #1 team is the one the team summary shows. When two of your teams play at once, the scoreboard follows the higher one.

Restart the hub and check it:

```bash
sudo supervisorctl restart tape-to-tape
cd ~/tape-to-tape && .venv/bin/python ping_hub.py   # should print ✅ Hub is ONLINE
```

## Step 3: Install the scoreboard

```bash
cd ~
gh repo clone Tape-to-Tape-IoT/nhl-led-scoreboard-v2
cd nhl-led-scoreboard-v2
./setup.sh      # the app
./setup_pi.sh   # the panel driver (asks for sudo)
```

**Pi 4 and older:** if `setup_pi.sh` warns about `snd_bcm2835`, turn off the onboard sound and reboot:

```bash
echo "blacklist snd_bcm2835" | sudo tee /etc/modprobe.d/blacklist-rgb-matrix.conf
sudo update-initramfs -u
sudo reboot
```

## Step 4: Configure (optional)

Skip this if you have a 128x64 panel and want a 12-hour clock. Otherwise, from `~/nhl-led-scoreboard-v2`:

```bash
nano config/user.toml
```

* **Another panel size:** uncomment `width` and `height` under `[display]` and set them.
* **24-hour clock:** add this at the end:

  ```toml
  [clock]
  format = 24
  ```

There's no hub connection to set. The scoreboard finds the hub's Valkey on this Pi by itself.

If the panel looks wrong (swapped colours, blank, flicker), the top of this file lists what to change.

## Step 5: Test it

```bash
NLS_SOURCE=sim ./run.sh   # a demo game, no hub needed. Ctrl+C to stop
./run.sh                  # the real thing, from the hub
```

It asks for `sudo` because the panel driver needs root. Every start plays an intro animation. If the splash screen stays up after it, see Troubleshooting.

**No panel yet?** Run `NLS_MATRIX=emulator NLS_SOURCE=sim ./run.sh` and open `http://<pi-address>:8087` in a browser.

## Step 6: Start at boot

```bash
./setup_pi.sh --service
```

The hub and the scoreboard now both start at boot. Useful commands:

```bash
sudo supervisorctl restart nhl-led-scoreboard-v2   # after editing the scoreboard's config/user.toml
sudo supervisorctl restart tape-to-tape            # after editing the hub's config/user.toml
sudo supervisorctl tail -f nhl-led-scoreboard-v2   # follow the log (or tape-to-tape)
sudo supervisorctl stop nhl-led-scoreboard-v2      # before running ./run.sh by hand
```

You can also manage both at `http://<pi-address>:9001`. It has no password, so anyone on your network can use it.

## Troubleshooting

|Problem|Fix|
|-|-|
|`unable to find git executable in PATH`|Run `sudo apt install -y git`, then `gh auth login` again.|
|`repository not found`|Accept the invite for that repo. Check `gh auth status` shows the right account.|
|`destination path ... already exists`|You already cloned it. `cd` into it and run `git pull`.|
|`glibc ... is too old`|You're on Bookworm. Re-image with Trixie 64-bit.|
|`ImportError: libfontconfig.so.1`, or the browser preview starts instead of the panel|Re-run `./setup_pi.sh`.|
|`already running under Supervisor`|Run `sudo supervisorctl stop nhl-led-scoreboard-v2` first.|
|Splash screen never goes away|The hub isn't reachable. `sudo supervisorctl status tape-to-tape` should say `RUNNING`, and `valkey-cli ping` should print `PONG`.|
|Swapped colours, blank panel, flicker|See the top of the scoreboard's `config/user.toml`.|

## Updating

```bash
cd ~/tape-to-tape && git pull && ./setup.sh --service
cd ~/nhl-led-scoreboard-v2 && git pull && ./setup.sh && ./setup_pi.sh
sudo supervisorctl restart nhl-led-scoreboard-v2
```

Your `config/user.toml` and `.env` files are kept.

## Feedback

Open an issue at [github.com/Tape-to-Tape-IoT/nhl-led-scoreboard-v2/issues](https://github.com/Tape-to-Tape-IoT/nhl-led-scoreboard-v2/issues) with:

* Your Pi model and panel size
* Your versions: `git -C ~/tape-to-tape rev-parse --short HEAD` and `git -C ~/nhl-led-scoreboard-v2 rev-parse --short HEAD`
* What you saw (a photo helps)
* The log: the `./run.sh` output, or `sudo supervisorctl tail nhl-led-scoreboard-v2`

Never paste your `.env`. It holds passwords.
