# Updates

Newest first. The commands are in [Updating](README.md#updating).

## October 29, 2026

Update both. The new scoreboard needs the new hub, so do the hub first.

### Before you update

**The scoreboard's config moved.** `./setup.sh` moves your `config/user.toml` to `config/app/user.toml` for you. Run it after `git pull`, like the [Updating](README.md#updating) steps say.

**If you edited a scene config file**, like `scenes/scene_config.xl.toml`, `git pull` stops with `Your local changes to the following files would be overwritten by merge`. Your changes now go in `config/scene/user.toml`, which updating never touches:

1. Copy what you changed somewhere safe, like a note on your computer. `git diff` shows it if you don't remember.
2. Undo your changes and update:

   ```bash
   cd ~/nhl-led-scoreboard-v2
   git checkout -- .
   git pull && ./setup.sh && ./setup_pi.sh
   ```

3. Put your changes in `config/scene/user.toml`, under the same `[table]` names, then run `sudo supervisorctl restart nhl-led-scoreboard-v2`.

That one file covers every panel size, so name only the settings you changed.

**If your scoreboard config has an `[alerts]` section** (like `goal = "main_event"`), it's now ignored and the log warns about it. Each alert type has its own table now:

```toml
[alerts.goal]
show = "main_event"
```

### New

* **Goal details:** after the goal animation, a card shows the scorer and the assists. It plays for either team's goal.
* **Penalty details:** a card shows the player, the infraction and the length of every penalty, for either team.
* **Team summary:** shows the next game's start time ([#4](https://github.com/Tape-to-Tape-IoT/nhl-led-scoreboard-v2/issues/4)).
* **Alerts play one at a time.** A goal landing during a penalty card waits for it to finish, and two goals close together play back to back.

The power play animation isn't there yet.

### Changed

* **The goal animation now plays for both teams' goals,** not only your team's. To keep it to your teams, add this to `config/app/user.toml`:

  ```toml
  [alerts.goal]
  teams = "favorite"
  ```

* When two of your teams play each other, the higher one on your list counts as your team.
* The scoreboard switches to a higher team's game at puck drop, not during its pregame.
* `limit_refresh` now defaults to `0`, no cap on the panel's refresh rate. If you set it yourself, your value is kept.
* Scene tweaks go in `config/scene/user.toml`. See [Scenes](PROJECT_OVERVIEW.md#scenes).

### Fixed

* A Pi switched on in the afternoon no longer shows the previous day's scores ([#9](https://github.com/Tape-to-Tape-IoT/nhl-led-scoreboard-v2/issues/9)).
* On 64x32, the standings no longer cut off "DIVISION STANDINGS". The line under the title now says only "DIVISION" ([#8](https://github.com/Tape-to-Tape-IoT/nhl-led-scoreboard-v2/issues/8)).
* Running the hub's `./setup.sh --service` no longer stops the scoreboard ([closed-beta#1](https://github.com/Tape-to-Tape-IoT/closed-beta/issues/1)). The scoreboard's `./setup_pi.sh --service` leaves the hub alone too.

### If your own apps read the hub

Skip this if you only run the scoreboard.

* `alerts/goal` now fires for either team's goal. Check its new `is_favorite` field if you only want your team's.
* One `powerplay` alert for either team replaces `powerplay_start` and `penalty`. It's sent once per power play, after the NHL settles the call. `tapetotape/alerts/penalty` is no longer sent. `penalty_details` still covers every penalty.
* `goal_details` now has the scorer's `sweater_number`.

[TOPICS_REFERENCE.md](https://github.com/Tape-to-Tape-IoT/tape-to-tape/blob/master/TOPICS_REFERENCE.md) has the payloads.
