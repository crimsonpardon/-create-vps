<div align="center">

# -create-vps

**A Discord bot that deploys and manages Docker-based VPS containers with slash commands.**

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![discord.py](https://img.shields.io/badge/discord.py-2.x-5865F2)
![Docker](https://img.shields.io/badge/Docker-required-2496ED)

</div>

---

## Overview

IL8 lets Discord users create Ubuntu 22.04 or Debian 11 containers, then control them from chat. Every VPS gets a browser/terminal access link through [sshx](https://sshx.io), sent to the user by DM.

## Features

- Create, start, stop, restart, reinstall and delete VPS containers
- Per-user VPS limit and a global running-instance limit
- RAM and CPU limits per container
- sshx access link sent by DM (`/ssh` makes a new one)
- Live info: uptime, CPU and memory usage, network I/O, logs
- Admin tools: create for a user, manage, suspend, ban, stats, stop all
- SQLite database for users, VPS records and bans
- Live bot status showing the number of active VPS
- `/about` command with developer info and links (footer: **Crimson**)

## Requirements

- Linux host with **Docker** installed and running
- Python 3.10+
- A Discord bot token

## Installation

```bash
git clone https://github.com/crimsonpardon/-create-vps.git
cd -create-vps
pip install -r requirements.txt
```

## Setup

1. Create an application in the [Discord Developer Portal](https://discord.com/developers/applications) and add a bot.
2. Enable **Message Content Intent** (the code requests it; login fails without it).
3. Invite the bot with the `bot` and `applications.commands` scopes.
4. Copy `il8_env.txt` to `.env` and fill in your values (see below).
5. Make sure the user running the bot can use Docker (member of the `docker` group, or root).
6. Start the bot:

```bash
python bot.py
```

Slash commands sync automatically on startup.

## Configuration

### `.env`

| Variable | Description | Default |
|---|---|---|
| `TOKEN` | Discord bot token | required |
| `ADMIN_ID` | Discord user ID of the admin | `0` |
| `BOT_STATUS_NAME` | Name shown in the bot status | `IL8` |
| `WATERMARK` | Footer text on most embeds | `Made by INFINITE` |
| `VPS_HOSTNAME` | Hostname prefix (user ID is appended) | `infinite-vps` |
| `TOTAL_SERVER_LIMIT` | Max running VPS across all users | `50` |
| `DATABASE_FILE` | SQLite file path | `bot.db` |

### VPS limits (top of `bot.py`)

Edit these values directly in `bot.py`. They are **not** read from `.env`.

```python
SERVER_LIMIT = 1       # Max VPS each user can create with /create
DEFAULT_RAM = '2g'     # RAM per VPS, e.g. '1g', '2g', '4g', '512m'
DEFAULT_CPU = '1'      # CPU cores per VPS, e.g. '1', '2', '0.5'
DEFAULT_DISK = '10G'   # Disk label only, not enforced by Docker
```

> CPU and RAM must fit within what the host machine actually has, or `/create` is refused.

## Commands

### User

| Command | Description |
|---|---|
| `/create <os>` | Deploy a new VPS with the default resources |
| `/list` | List your VPS instances |
| `/vps-info [vps_id]` | Full details: usage, uptime, SSH link |
| `/start <vps_id>` | Start a VPS |
| `/stop <vps_id>` | Stop a VPS |
| `/restart <vps_id>` | Restart a VPS |
| `/ssh [vps_id]` | Generate a new sshx link |
| `/reinstall <vps_id> [os]` | Wipe and reinstall with a chosen OS |
| `/delete <vps_id>` | Remove a VPS |
| `/logs <vps_id> [lines]` | Recent container logs |
| `/ping` | Bot latency |
| `/help` | Command list |
| `/about` | Bot and developer info |

`vps_id` can be the container ID or part of the container name.

### Admin (only the user set in `ADMIN_ID`)

| Command | Description |
|---|---|
| `/admin-create <user> <os> [ram] [cpu] [disk]` | Create a VPS for a user |
| `/admin-manage <user> <vps> <action>` | start, stop, restart, delete, suspend, unsuspend |
| `/admin-list` | All VPS instances (first 25) |
| `/admin-users` | Users with VPS counts |
| `/admin-stats` | Totals and allocated resources |
| `/admin-vps-info <user> <vps>` | Full details for a user's VPS |
| `/admin-logs <user> <vps> [lines]` | Logs for a user's VPS |
| `/admin-del-user <user>` | Delete all VPS of a user |
| `/admin-ban <user>` / `/admin-unban <user>` | Block or allow VPS creation |
| `/admin-stop-all` | Stop every running VPS |

## How it works

1. `/create` checks ban status, per-user and global limits, and that the CPU/RAM fit the host.
2. It runs `docker run -d` with the chosen image, memory and CPU limits and a hostname.
3. It installs `curl`, `wget`, `sudo`, `openssh-client` and sshx inside the container.
4. It starts sshx, captures the session URL, saves it to SQLite and DMs it to the user.

Database tables: `users`, `vps`, `bans`. Logs are written to `bot.log` and the console.

## Project structure

```
bot.py            # the bot
requirements.txt  # Python dependencies
il8_env.txt       # example environment file (copy to .env)
bot.db            # created on first run
bot.log           # created on first run
```

## Security notes

- Containers run with `--privileged --cap-add=ALL`. That gives users near-host-level power and is a real container-escape risk. Only give access to people you trust.
- The bot needs access to the Docker socket, which is effectively root on the host.
- sshx is installed with `curl ... | sh` inside each container.
- **Never commit your `.env`, token, `bot.db` or `bot.log` to this public repo.** Add them to a `.gitignore`:

```gitignore
.env
bot.db
bot.log
__pycache__/
```

## Credits

Made by **INFINITE** · [YouTube](https://www.youtube.com/@infinite8labs) · [GitHub](https://github.com/nxtinfinite481-png) · [Discord](https://discord.gg/pG22dSmAZD)

Crimson
