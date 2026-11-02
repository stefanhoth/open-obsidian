---
name: open-obsidian
description: Turn an Obsidian note into a clickable https link (https://open-obsidian.stefanhoth.com/?url=…) that opens the note in Obsidian. Use when sharing a note link in a place that doesn't accept obsidian:// links, such as Slack, Telegram, email, GitHub, todo apps or a chat reply. Works alongside the obsidian-cli skill, which looks up the vault name and note path.
---

# open-obsidian: https links to Obsidian notes

Many apps refuse to link `obsidian://` URIs. `https://open-obsidian.stefanhoth.com/?url=<encoded obsidian URI>` is a normal web link that immediately forwards to the `obsidian://` URI, so the note opens in Obsidian on whatever device the user clicks it.

## When to use what

| Goal | Use |
|------|-----|
| Read, create, edit, search notes yourself | **obsidian-cli** (`obsidian read`, `obsidian create`, …) |
| Open a note in the running Obsidian app right now | **obsidian-cli** (`obsidian open path=…`). No link needed. |
| Give the *user* a link to click later or in another app/device | **open-obsidian** (this skill) |

Never "open" a note for yourself through this website, because it only redirects a browser. Never put CLI syntax (`file="…"`) into a link; links use Obsidian URI parameters.

## Build a link

1. Build the Obsidian URI and percent-encode each parameter value:
   `obsidian://open?vault=<vault>&file=<path from vault root, .md optional>`
2. Percent-encode the **whole** URI once more and append it after `?url=`.
3. `url` must be the last query parameter: the page takes everything after `url=` and decodes it once.

```bash
python3 -c 'import sys,urllib.parse as u; q=lambda s:u.quote(s,safe=""); print("https://open-obsidian.stefanhoth.com/?url="+q(f"obsidian://open?vault={q(sys.argv[1])}&file={q(sys.argv[2])}"))' "My Vault" "Daily Notes/2026-03-10"
# → https://open-obsidian.stefanhoth.com/?url=obsidian%3A%2F%2Fopen%3Fvault%3DMy%2520Vault%26file%3DDaily%2520Notes%252F2026-03-10
```

```js
const q = encodeURIComponent;
const link = `https://open-obsidian.stefanhoth.com/?url=${q(`obsidian://open?vault=${q(vault)}&file=${q(file)}`)}`;
```

Notes:
- The note opens in a **new tab** by default (`paneType=tab` is added). To control this, set `paneType=tab|split|window` (or `newpane=…`) in the Obsidian URI yourself.
- The page does **not** close its tab by default, so first-time users can tick the browser's "always allow" checkbox. Only add `autoclose=1&` before `url=` (`…/?autoclose=1&url=…`) when the user asks for it or has already set up "always allow". It only works for tabs opened by a script.
- Headings and blocks: append `#Heading` or `#^block-id` to the `file` value *before* encoding (`Note#Heading` → `Note%23Heading`).
- Other URI actions work too, e.g. `obsidian://daily?vault=…` or `obsidian://search?vault=…&query=…`. The page only checks that the target starts with `obsidian://`.

## Combine with obsidian-cli

Get the exact values from the CLI instead of guessing them:

```bash
obsidian vault info=name                 # vault name → vault=
obsidian file file="Recipe"              # prints "path  Notes/Recipe.md" → file=
obsidian daily:path                      # path of today's daily note
obsidian vault="My Vault" file path="Projects/Plan.md"   # when several vaults exist
```

Typical flow: create or update the note with obsidian-cli (`create` doesn't open the note unless you pass `open`), read its `path` with `obsidian file`, then build the link as above and hand it to the user.

```bash
obsidian create path="000 - Inbox/Meeting notes.md" content="# Meeting notes"
obsidian file path="000 - Inbox/Meeting notes.md"   # confirm path
# link: obsidian://open?vault=<name>&file=000%20-%20Inbox%2FMeeting%20notes.md, encoded again after ?url=
```

## Pitfalls

- **Wrong encoding**: an unencoded `&` or space splits the URI. Encode the values, then encode the whole URI. Don't encode it a third time.
- **Vault name per device**: the link resolves on the device where it's clicked. The vault must have the same name there (e.g. on the phone). If unsure, ask the user.
- **Privacy**: vault name and note path are visible in the link. Don't post links to sensitive notes in shared channels.
- **Legacy URL**: `stefanhoth.github.io/open-obsidian/` only redirects to the new domain. Always generate links with `open-obsidian.stefanhoth.com`.
