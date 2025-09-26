# RDP GitHub Actions Workflow with Tailscale and Management Bot

This project provides a GitHub Actions workflow to deploy a temporary Windows-based Remote Desktop Protocol (RDP) server on a GitHub-hosted runner, secured via Tailscale VPN. It includes automated file downloads from a repository-hosted list, automatic execution of specific software (e.g., unMiner.exe), periodic backups sent to Telegram, and a Telegram bot for managing multiple instances (e.g., checking status, requesting instant backups, screenshots, or system resource usage).

**Note:** This setup is intended for educational or testing purposes. Running resource-intensive tasks like mining on GitHub Actions may violate GitHub's Terms of Service. Ensure compliance with all platform policies.

## Features
- **Secure RDP Access:** Enables RDP over a custom port via Tailscale for encrypted, peer-to-peer connections.
- **Automated Downloads:** Downloads files from URLs listed in `links.txt` to a temporary directory.
- **Software Auto-Execution:** Automatically runs `unMiner.exe` if downloaded.
- **Periodic Backups:** Zips and sends a specified folder (default: `C:\MyBackupFolder`) to Telegram every hour.
- **Telegram Bot Management:** PHP-based backend with Python clients for controlling multiple workflow instances:
  - List online/offline instances.
  - Request instant backups, screenshots, or system status (CPU/RAM usage) for specific instances.
- **Multi-Instance Support:** Manages multiple simultaneous workflow runs with identical code.

## Prerequisites
- A GitHub repository with this workflow.
- GitHub secrets configured (see below).
- A Tailscale account with an auth key.
- A Telegram bot token (create via [@BotFather](https://t.me/botfather)) and chat ID (for backups and bot interactions).
- A PHP-compatible server (e.g., shared hosting, VPS with Apache/Nginx) to host the bot backend.
- Basic knowledge of GitHub Actions, PowerShell, Python, and PHP.

## Required GitHub Secrets
Add these to your repository settings under **Settings > Secrets and variables > Actions**:

| Secret Name          | Description                                                                 |
|----------------------|-----------------------------------------------------------------------------|
| `RDP_PORT`           | Custom RDP listening port (e.g., 3390).                                     |
| `RDP_PASSWORD`       | Password for RDP users (runneradmin and installer).                         |
| `TAILSCALE_AUTH_KEY` | Tailscale authentication key (generate from Tailscale admin console).       |
| `TELEGRAM_BOT_TOKEN` | Telegram bot token for backups and management.                              |
| `TELEGRAM_CHAT_ID`   | Telegram chat ID where backups and responses are sent (e.g., your user ID). |

## Setup Instructions

### 1. Repository Setup
- Create a new GitHub repository.
- Add the workflow file: Copy the provided `workflow.yml` to `.github/workflows/rdp.yml`.
- Add `links.txt`: Create this file in the repo root with one download URL per line (e.g., for unMiner.exe).
- Add `client.py`: Copy the provided Python code to the repo root.
- Commit and push changes.

### 2. PHP Backend Setup
- Host `api.php` and `bot.php` on your PHP server (ensure the directory is writable for JSON files).
- Replace `YOUR_BOT_TOKEN` in `bot.php` with your Telegram bot token.
- Run `bot.php` persistently (e.g., `php bot.php &` or use a process manager like Supervisor).
- Update `SERVER_URL` in `client.py` to point to your hosted `api.php` (e.g., `https://yourdomain.com/api.php`).

### 3. Running the Workflow
- Go to your repo's **Actions** tab.
- Select the "RDP" workflow.
- Click **Run workflow** (manual dispatch).
- The workflow will:
  - Checkout the repo.
  - Download files from `links.txt`.
  - Execute `unMiner.exe` if present.
  - Set up RDP and Tailscale.
  - Start the Python client for bot management.
  - Run periodic backups.
  - Display connection info in the logs (Tailscale IP, port, credentials).
- Connect via RDP: Use an RDP client (e.g., Microsoft Remote Desktop) with the Tailscale IP and port. Join the same Tailscale network on your client machine.

### 4. Using the Telegram Bot
- Start chatting with your Telegram bot.
- Commands:
  - `/list`: Lists all instances with online/offline status (based on heartbeats; offline if no heartbeat in 5 minutes).
  - `/backup <instance_id>`: Queues an instant backup for the specified instance (zips and sends to Telegram).
  - `/screenshot <instance_id>`: Queues a screenshot send.
  - `/status <instance_id>`: Queues CPU/RAM usage report.
- Instance IDs are GitHub run IDs (visible in workflow logs or `/list` output).
- Responses (backups, screenshots, status) are sent directly to the configured Telegram chat.

## Customization
- **Backup Folder:** Change `$backupFolder` in the workflow or `BACKUP_FOLDER` in `client.py`.
- **Download Issues:** If downloads fail, check URLs in `links.txt` or add `-UserAgent "Mozilla/5.0"` to `Invoke-WebRequest` for compatibility.
- **Bot Security:** Restrict bot access by checking `$chat_id` in `processMessage` against allowed IDs.
- **Python Dependencies:** Already installed via workflow; add more via `pip install` if needed.
- **Multiple Instances:** Trigger the workflow multiple times; each registers uniquely via run ID.

## Troubleshooting
- **Downloads Fail:** Verify URLs; test with `curl` locally. GitHub runners may block certain domains—use proxies if needed.
- **Backups Not Sending:** Check file size (<50MB), Telegram API limits, or secrets. View workflow logs for errors.
- **Bot Not Responding:** Ensure `bot.php` is running and server is accessible. Check PHP error logs.
- **Client Errors:** Workflow logs show Python issues; ensure `SERVER_URL` is correct.
- **RDP Connection Fails:** Confirm Tailscale network join, port forwarding, and firewall rules.

## Limitations
- Workflow timeouts at 6 hours (GitHub limit).
- Telegram file limit: 50MB; larger backups are skipped.
- No persistent storage; runners are ephemeral.
- For production, consider a dedicated VPS instead of GitHub Actions.

## License
MIT License. Use at your own risk.