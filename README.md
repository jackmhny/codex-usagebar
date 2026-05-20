# codex-usagebar

Tiny status widget for showing Codex `/status` usage limits in `i3status`.

[HTML docs](https://jackmhny.github.io/codex-usagebar/README.html)

<img src="assets/codex-usagebar-bar.png" alt="codex-usagebar output cropped to the i3status bar" width="980">

<!-- bar text: ████████████░░ 92% 05:04 (colors in terminal) -->

## About

`codex-usagebar` reads the ChatGPT access token that Codex stores in
`~/.codex/auth.json`, calls the ChatGPT usage endpoint used by Codex status
requests, formats the remaining primary window as a color-coded block bar, and atomically writes
the result to:

```text
~/.cache/i3status/codex-usage
```

The token is only used as a bearer token for the request. It is never printed
and never cached.

## Dependencies

- `sh`
- `curl`
- `jq`
- GNU `date`
- `mktemp`

## Install

Copy the script somewhere stable and make it executable:

```sh
install -Dm755 codex-usagebar ~/.local/bin/codex-usagebar
```

Run it once:

```sh
~/.local/bin/codex-usagebar
cat ~/.cache/i3status/codex-usage
```

## Timer

Run it from a systemd user timer. The service is oneshot because the script exits
after each update.

```sh
mkdir -p ~/.config/systemd/user
cat > ~/.config/systemd/user/codex-usagebar.service <<'EOF_SERVICE'
[Unit]
Description=Update Codex usage status for i3status

[Service]
Type=oneshot
ExecStart=%h/.local/bin/codex-usagebar
EOF_SERVICE

cat > ~/.config/systemd/user/codex-usagebar.timer <<'EOF_TIMER'
[Unit]
Description=Refresh Codex usage status for i3status

[Timer]
OnBootSec=30s
OnUnitActiveSec=60s
AccuracySec=10s
Unit=codex-usagebar.service

[Install]
WantedBy=timers.target
EOF_TIMER

systemctl --user daemon-reload
systemctl --user enable --now codex-usagebar.timer
```

## i3status

Use `read_file` and point it at the cache path.

```i3status
order += "read_file codex_usage"

read_file codex_usage {
        path = "~/.cache/i3status/codex-usage"
}
```

## Customize

Edit the constants at the top of `codex-usagebar`.

```sh
auth=${HOME}/.codex/auth.json
out=${HOME}/.cache/i3status/codex-usage
url=https://chatgpt.com/backend-api/wham/usage
icon=''

bar_width=14
block='█'
esc=$(printf '\\033')
color_red="${esc}[38;5;203m"
color_yellow="${esc}[38;5;221m"
color_green="${esc}[38;5;120m"
color_bg="${esc}[48;5;236m"
color_reset="${esc}[0m"
```

Change the output path, icon, endpoint, bar styling, or formatting directly in the
file.

## Output format

The normal line is:

```text
████████████░░  92% 05:04
```

(Blocks are colorized in your status bar: green/yellow/red for filled usage and gray background for the unfilled tail.)

That means:

- The block bar shows the remaining primary-window quota at a glance.
- `92%` is remaining usage in the primary window.
- `05:04` is the reset time (today or short weekday+time if not today).
- `LIMITED` appears as a prefix if the endpoint reports `limit_reached`.
