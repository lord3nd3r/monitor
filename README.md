# monitor (Sopel plugin)

A small Sopel plugin that tracks how much each user talks per channel.

It records message counts and first/last-seen timestamps in a SQLite database, then provides IRC commands to display leaderboards and per-user stats.

## Requirements

- Sopel (tested against modern Sopel 8.x/9.x style plugin APIs)
- Python 3.9+ (whatever Sopel supports in your environment)
- SQLite (Python’s built-in `sqlite3` module)

No third-party Python dependencies.

## Install

### Option A: Sopel “scripts” directory

Copy `monitor.py` into your Sopel scripts directory.

Common locations:

- `~/.sopel/scripts/` (Sopel user install)
- A system-wide plugins/scripts directory (depends on how you deployed Sopel)

Example:

```bash
cp monitor.py ~/.sopel/scripts/monitor.py
```

### Option B: Sopel “plugins” directory

You can also place it under `~/.sopel/plugins/` if that’s how your bot is configured.

## Enable / Load

How Sopel loads plugins depends on your Sopel configuration and deployment. Typical approaches:

- Start/restart Sopel (it will load the plugin on startup)
- Use the `reload` plugin (if installed) to reload without restarting the bot

If you have Sopel’s `reload` plugin enabled:

- Reload this plugin:

```text
$reload monitor
```

The bot should report it reloaded the module from your scripts path.

## Configuration

This plugin uses the Sopel config section `[channelstats]`.

### Minimal config

```ini
[channelstats]
channels =
    #chatters
    #anotherchan
```

### Full config

```ini
[channelstats]
# List of channels that are eligible to be monitored.
# (Multi-line format is recommended; comma-separated is deprecated in Sopel 9.)
channels =
    #chatters
    #anotherchan

# Where to store the SQLite DB.
# - If absolute: used as-is
# - If relative: stored next to your Sopel config file
# Default: monitor.db
# db_path = monitor.db

# If True, eligible channels are monitored by default unless explicitly disabled.
# If False, you must enable monitoring with `.monitor on #channel`.
# Default: false
# default_enabled = false

# If True, admins can add a channel to the eligible list at runtime (no bot restart)
# via `.monitor on #channel`. The channel is persisted into the plugin DB.
# Default: false
# allow_admin_add = false
```

### Where is the database?

By default (`db_path = monitor.db`), the plugin stores its SQLite file **next to your Sopel config file**.

For example, if your Sopel config is:

- `/home/ender/.sopel/default.cfg`

Then the DB will be:

- `/home/ender/.sopel/monitor.db`

If you set an absolute `db_path`, that location is used.

## What gets tracked

- Only channel messages (`PRIVMSG` in channels)
- Per `(channel, nick)` counters
- `first_seen` and `last_seen` timestamps (UTC)

It ignores the bot’s own messages.

## Commands

### `.monitor` (admin-only, **PM-only**)

Controls per-channel monitoring.

- Enable monitoring for a channel:

```text
.monitor on #channel
```

- Disable monitoring:

```text
.monitor off #channel
```

- List currently enabled monitored channels (from the eligible list):

```text
.monitor list
```

Notes:

- The channel must be in the **eligible channel list** (`[channelstats] channels`), unless you enabled `allow_admin_add = true`.
- The bot must already be **in the channel** (safety check).

### `.channelstats` (channel-only)

Shows the talk leaderboard for the current channel.

```text
.channelstats
```

Outputs:

- Top 10 talkers
- Bottom 10 talkers (lowest message counts)

The top 3 talkers in the “Top 10” line are prefixed with medals (🥇🥈🥉).

### `.userstats` (channel-only)

Shows stats for a single user in the current channel.

- Your own stats:

```text
.userstats
```

- Specific nick:

```text
.userstats SomeNick
```

Outputs:

- message count
- first seen (UTC)
- last seen (UTC)

## Operational notes

### Reloading vs restarting

- Reloading the plugin (e.g. `$reload monitor`) updates the code immediately.
- Sopel generally does **not** re-read the main config file just because a plugin is reloaded.
  - If you change `[channelstats] channels` in the config file, you may need to restart Sopel to ensure the new config values are loaded.
  - Alternatively, use `allow_admin_add = true` to add channels at runtime with `.monitor on #channel`.

### Performance

The plugin does a small SQLite write per message (with WAL enabled). It is designed to be lightweight, but extremely busy channels could still generate a lot of writes.

## Troubleshooting

### “#channel is not in the eligible channel list in config”

Fix one of these:

1. Add the channel to `[channelstats] channels` and restart Sopel (recommended if you want a strict allowlist), or
2. Set `allow_admin_add = true` and then run:

```text
.monitor on #channel
```

### “I’m not currently in #channel”

The plugin refuses to enable monitoring for a channel the bot isn’t in. Join the channel first, then re-run `.monitor on #channel`.

### No stats appear

- Ensure monitoring is enabled for that channel: `.monitor list` (in PM)
- Ensure messages are being sent in the channel
- Check the DB file exists and grows: `ls -lh /path/to/monitor.db`

## License

No license is specified yet. Add one if you intend others to reuse/redistribute.
