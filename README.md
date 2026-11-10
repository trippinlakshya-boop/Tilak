# Scam Report Bot — Render deployment

This package contains the uploaded Telegram bot prepared for Render. Its existing bot features and handlers are retained; deployment secrets are read from environment variables.

## Before deploying

1. The bot token in the original uploaded file was exposed. Revoke it with @BotFather and create a new token.
2. Make sure `CHANNEL_USERNAME` is the public username of your report channel (without `@`).
3. Add the bot as an administrator in that channel with permission to post messages. It also checks whether reporters joined the channel.
4. Collect the numeric Telegram user IDs of the people who should be prime admins. Put them in `PRIME_ADMIN_IDS`, separated by commas.

## Render setup (recommended: Background Worker)

A Telegram polling bot should run as a Background Worker, not as a website. The included `render.yaml` defines a paid Background Worker and a 1 GB persistent disk mounted at `/var/data`, where SQLite stores reports and settings.

1. Extract this ZIP.
2. Upload the extracted files to a new GitHub repository. Keep `.gitignore`; do not upload a real `.env` file or secrets.
3. In Render, choose **New → Blueprint** and connect that GitHub repository.
4. Render detects `render.yaml`. Review the service: `scam-report-bot`, type **Background Worker**, region **Singapore**.
5. When prompted, enter:
   - `BOT_TOKEN`: the newly generated BotFather token
   - `PRIME_ADMIN_IDS`: numeric IDs separated by commas, e.g. `123456789,987654321`
6. Confirm the paid worker and persistent disk, then apply/deploy the Blueprint.
7. Open the service's **Logs**. A successful startup should show `Bot starting...` and no traceback.
8. Open the bot in Telegram and send `/start`. Test a report flow and admin notification.

## Environment variables

| Variable | Required | Value |
|---|---:|---|
| `BOT_TOKEN` | Yes | New token from @BotFather |
| `CHANNEL_USERNAME` | Yes | Channel username without `@` (default in Blueprint: `worksafer`) |
| `PRIME_ADMIN_IDS` | Yes | Comma-separated numeric admin IDs |
| `DB_PATH` | Yes | `/var/data/scam_bot.db` (set by Blueprint) |
| `TG_API_ID` | No | `0` unless enabling optional Telethon lookup |
| `TG_API_HASH` | No | Optional Telethon API hash |
| `TG_SESSION` | No | Optional Telethon StringSession |

## Notes

- `requirements.txt` installs python-telegram-bot 21.6, Pillow (for image cards), and Telethon (optional account-based lookup).
- Do not run two copies of the bot with the same token at the same time; Telegram polling conflicts if multiple instances poll the same bot.
- Do not remove the persistent disk or change `DB_PATH` if you want to keep the existing database.
- Render's free web services can sleep and have ephemeral local storage. This package uses a Background Worker with persistent storage for continuous polling and database retention.
- The bot's Telegram profile photo and About text are configured separately in @BotFather.
