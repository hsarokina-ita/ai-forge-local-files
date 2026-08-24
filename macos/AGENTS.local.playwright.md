# Playwright standalone browser — this machine

Machine-specific procedure referenced by `AGENTS.local.md`. Platform: macOS, using the Playwright-managed Chromium build via `npx` with a dedicated persistent profile (no OS service manager involved).

For interactive AI Forge browser work, use the Playwright-managed Chromium build with the dedicated persistent profile configured in:

```text
.playwright-cli/ai-forge-chromium.config.json
```

Open the app with:

```bash
npx --yes @playwright/cli \
  --config=.playwright-cli/ai-forge-chromium.config.json \
  open http://localhost:8306/
```

- Use `npx`; a global `playwright-cli` installation is not required.
- Do not add `--browser=chrome`, because that selects branded Chrome rather than the managed Chromium build.
- Do not use the default in-memory profile for authenticated workflows. The configured profile preserves login after the browser is closed.
- The `AutomationControlled` command-line flag warning is expected in this automation-only browser and does not affect the user's normal Chrome profile.
