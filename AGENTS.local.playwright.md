# Playwright standalone browser — this machine

Machine-specific procedure referenced by `AGENTS.local.md`. Platform: Linux with Cinnamon, systemd user services, and Chrome for Testing from the Playwright cache.

Use the installed Chrome for Testing as a separate Cinnamon window.

Before launching Chrome, check whether the existing browser's CDP endpoint is available:

```bash
curl --fail --silent --show-error http://127.0.0.1:9224/json/version
```

- If the command succeeds, the test browser is already running. Reuse it and do not launch another browser.
- If it fails, check for both a matching browser process and any active Forge browser unit (the unit may have a numeric suffix):

```bash
pgrep -af 'chrome.*--remote-debugging-port=9224'
systemctl --user list-units --type=service --state=active \
  'forge-playwright-browser*.service'
```

- If either check finds a browser, it is still starting or its CDP endpoint is unhealthy. Do not launch another instance. Inspect matching units with `systemctl --user status 'forge-playwright-browser*.service'`, then retry the CDP check.
- Launch Chrome only when the CDP check fails, no matching browser process exists, and no matching Forge browser unit is active.

```bash
systemd-run --user --unit=forge-playwright-browser \
  --setenv=DISPLAY=:0 \
  --setenv=XAUTHORITY=/home/ann40a/.Xauthority \
  /home/ann40a/.cache/ms-playwright/chromium-1232/chrome-linux64/chrome \
  --user-data-dir=/tmp/forge-playwright-profile \
  --remote-debugging-port=9224 \
  --class=ForgePlaywright \
  --new-window \
  --no-first-run \
  --no-default-browser-check \
  http://localhost:8306
```

Attach Playwright to the same window and keep using the named session:

```bash
playwright-cli -s=forge-test attach --cdp=http://127.0.0.1:9224
playwright-cli -s=forge-test snapshot
```

Run the browser launch and CDP interaction commands with desktop/network sandbox escalation when required.
