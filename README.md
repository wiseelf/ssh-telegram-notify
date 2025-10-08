# SSH Telegram Notify

This Ansible role configures SSH login notifications via Telegram bot. When a user logs in via SSH, a notification is sent to a specified Telegram chat.

## Requirements

- Ansible 2.10 or higher
- Debian or Ubuntu target systems
- `curl` package (automatically installed)
- `libpam-modules` package (automatically installed)
- A Telegram bot token and chat ID

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
# Telegram bot configuration (REQUIRED)
telegram_bot_token: ""
telegram_chat_id: ""

# Script configuration
pam_script_directory: "/etc/pam.scripts"
pam_script_name: "login.sh"
pam_script_path: "{{ pam_script_directory }}/{{ pam_script_name }}"

# PAM configuration
pam_sshd_config_path: "/etc/pam.d/sshd"
pam_exec_line: "session    optional     pam_exec.so {{ pam_script_path }}"
```

## Dependencies

None.

## Setup Telegram Bot

1. Create a Telegram bot by messaging [@BotFather](https://t.me/botfather)
2. Send `/newbot` and follow the instructions
3. Save the bot token provided
4. Add the bot to your chat or group
5. Get your chat ID by sending a message to the bot and visiting:
   `https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates`

## Example Playbook

```yaml
- hosts: servers
  roles:
    - role: ssh-telegram-notify
      telegram_bot_token: "YOUR_BOT_TOKEN_HERE"
      telegram_chat_id: "YOUR_CHAT_ID_HERE"
```

Or with variables file:

```yaml
- hosts: servers
  vars_files:
    - vars/telegram.yml
  roles:
    - ssh-telegram-notify
```

Where `vars/telegram.yml` contains:

```yaml
telegram_bot_token: "YOUR_BOT_TOKEN_HERE"
telegram_chat_id: "YOUR_CHAT_ID_HERE"
```

## What it does

1. Validates that Telegram configuration variables are set
2. Installs required packages (`curl`, `libpam-modules`)
3. Creates `/etc/pam.scripts/` directory with proper permissions
4. Deploys the notification script to `/etc/pam.scripts/login.sh`
5. Adds PAM configuration to `/etc/pam.d/sshd`:
   ```
   session    optional     pam_exec.so /etc/pam.scripts/login.sh
   ```
6. Verifies PAM configuration syntax using `pamtester`
7. Restarts SSH service if configuration changes

## Notification Format

The Telegram notification is sent as a single-line message with:

- 🟢/🔴/🔵 Color-coded action indicators (Login/Logout/Other)
- Action description (SSH Login, SSH Logout, etc.)
- Username and hostname
- Source IP address
- Timestamp with timezone information

Example notification:

```
🟢 SSH Login: pi@raspberrypi.local from 192.168.1.100 at ⏰ 14:30:25 on 07 Oct 2025 (UTC)
```

The script automatically detects different PAM actions:

- 🟢 **SSH Login** (open_session)
- 🔴 **SSH Logout** (close_session)
- 🔵 **Other SSH Actions** (any other PAM type)

## Development & Contributing

### Commit Convention

This project uses [Conventional Commits](https://www.conventionalcommits.org/) for automated semantic versioning. Please format your commit messages as follows:

- `feat: add new feature` (minor version bump)
- `fix: resolve bug` (patch version bump)
- `docs: update documentation` (patch version bump)
- `chore: maintenance tasks` (no version bump)
- `refactor: code improvements` (patch version bump)

### Automated Releases

- **Semantic Release**: Automatic versioning based on commit messages
- **GitHub Actions**: Automated testing, linting, and releases
- **Ansible Galaxy**: Automatic publishing of new versions

## Compatibility

This role is tested and supported on:

- **Debian**: all versions
- **Ubuntu**: all versions

## License

BSD/MIT

## Author Information

**Author:** wiseelf
**Description:** Telegram notification for SSH login on Debian/Ubuntu systems
Created for infrastructure management and security monitoring.
