# omarchy-antigravity

Keep an eye on your **Google Antigravity** subscription without leaving the bar: the
5-hour and weekly quota for each model group, with the live reset countdowns, as a
native tab in the Omarchy agents panel.

No extra tooling, no API key to paste, no background daemon. The plugin asks the
Antigravity CLI you are already signed into for its own `/usage`, and writes one record
the panel picks up.

## What you get

- **Two limit groups** — Gemini models and Claude/GPT models, each with a **5-hour** and
  a **weekly** window, matching what `agy` shows under `/usage`.
- **Percentage used** per window, with a meter and the time until it resets.
- **Subscription-aware** — a signed-in Google account with **Google AI Pro** (or Ultra)
  gets higher limits; the plugin reports whatever the CLI reports.

## Requirements

- Omarchy, with the `omarchy.agents` bar widget in your layout.
- The Antigravity CLI (`agy`), signed in — the same login the CLI uses. Install it
  following Google's official instructions at
  <https://antigravity.google/docs/cli/install>. The plugin never downloads or executes
  an installer itself.
- `python3` (already present on Omarchy).

## Install

```bash
omarchy plugin add https://github.com/fsavoia/omarchy-antigravity.git --enable
```

That is the whole setup. Sign in to Antigravity once with the official CLI and the tab
appears; if it is not signed in, the plugin writes nothing and the panel simply shows no
Antigravity tab.

## Authentication

Nothing is issued, and nothing is written back. The plugin shells out to
`agy -p "/usage" --output-format json` and maps the result; the CLI owns the credential.

## How it works

- `bin/antigravity-collect` runs `agy -p "/usage" --output-format json`, reads
  `command.data.groups[].buckets[]` (`remaining_fraction`, `reset_time`), and prints one
  agents-panel record: `limits[]` where `percent = 1 - remaining_fraction`.
- `--write` writes it to `~/.local/state/omarchy/agents/usage/antigravity.json`
  atomically.
- `ui/main.qml` runs the collector every 5 minutes and at startup.
- Nothing outside `~/.local/state/omarchy/agents/usage/` is touched.

## Manual test

```bash
~/.config/omarchy/plugins/fsavoia.antigravity-usage/bin/antigravity-collect          # print the record
~/.config/omarchy/plugins/fsavoia.antigravity-usage/bin/antigravity-collect --write
jq . ~/.local/state/omarchy/agents/usage/antigravity.json
omarchy plugin validate ~/.config/omarchy/plugins/fsavoia.antigravity-usage
omarchy-shell shell rescanPlugins
```

## Uninstall

```bash
omarchy plugin remove fsavoia.antigravity-usage
rm ~/.local/state/omarchy/agents/usage/antigravity.json
```

Removing the plugin stops the collector; deleting the record drops the tab.

## License

MIT
