# claude-acc

Switch [Claude Code](https://claude.com/claude-code) between several Claude accounts on macOS without logging in again, and see each account's usage before you pick one.

```
$ claude-acc
* personal     me@example.com            5h: 4% (reset 17/11 23:29)   week: 32% (reset 23/11 04:59)
  work         me@company.com            5h: 2% (reset 18/11 01:19)   week: 59% (reset 21/11 02:59)

$ claude-acc use work
Switched to work (reload the VSCode window to refresh the account shown there)
* work         me@company.com            5h: 2% (reset 18/11 01:19)   week: 59% (reset 21/11 02:59)
```

You log in to each account once. After that, switching is one command, and a Claude Code session that is already running (terminal or VSCode extension) keeps going on the new account with its conversation intact.

## Requirements

- macOS (Claude Code keeps its login in the Keychain)
- Claude Code logged in with a Claude.ai account (Pro/Max/Team), not an API key
- `jq` and `curl` (`brew install jq`)

## Install

```sh
mkdir -p ~/bin
curl -fsSL https://raw.githubusercontent.com/tdvietdev/claude-acc/main/claude-acc -o ~/bin/claude-acc
chmod +x ~/bin/claude-acc
```

Make sure `~/bin` is on your `PATH`, for example add this to `~/.zshrc`:

```sh
export PATH="$HOME/bin:$PATH"
```

## Set up your accounts

Do this once per account:

1. Run `claude`, type `/login`, and log in to the account in the browser.
2. Exit Claude Code and run `claude-acc save <name>`, e.g. `claude-acc save personal`.

Repeat for every account. `/login` replaces the current login, so save each account right after logging in to it.

## Commands

| Command | What it does |
|---|---|
| `claude-acc` | List saved accounts with email, 5-hour usage and weekly usage. `*` marks the active one. |
| `claude-acc save <name>` | Save the currently logged-in account as `<name>`. Saving again under the same name updates it. |
| `claude-acc use <name>` | Switch Claude Code to `<name>` and show its usage. |
| `claude-acc sync` | Make the account shown by Claude Code (`/status`, VSCode) match the active login. |
| `claude-acc help` | Show usage. |

## Switching in the middle of a session

- **Session still running** (terminal or VSCode): run `claude-acc use <name>` in another terminal and keep chatting. The next request goes out on the new account.
- **Session already closed**: run `claude-acc use <name>`, then `claude --continue` in the same project folder.
- The VSCode extension reads the displayed account only at startup. Requests use the new account right away, but the name it shows changes only after `Developer: Reload Window`.

The first message after a switch resends the whole conversation to the new account, without prompt cache. On a long session that is a lot of input tokens. Running `/compact` first keeps it small.

## How it works

- Claude Code stores its OAuth login in the Keychain item `Claude Code-credentials`. `save` copies that item to `claude-acc-<name>`, and `use` copies a saved one back.
- Before switching away, `use` saves the active login back into its slot, because Claude Code refreshes tokens while you work and the saved copy would otherwise go stale.
- If the active login does not belong to the slot `claude-acc` thinks is active (e.g. you ran `/login` by hand and did not `save`), `use` stops instead of overwriting that slot.
- Saved access tokens expire after a few hours. When a saved one has expired, `claude-acc` refreshes it the same way Claude Code does and stores the new token in the slot.
- Usage comes from the same endpoint Claude Code's `/usage` uses.
- `use` also updates the account details in `~/.claude.json` so Claude Code shows the right email. The first time it does this it keeps a backup at `~/.claude-accounts/claude.json.bak`.

State lives in the Keychain (`claude-acc-*` items) and `~/.claude-accounts/` (active slot name and each slot's email). Tokens are never written to disk or printed.

## Limitations

- Connectors you added on claude.ai (Google Drive, etc.) belong to each Claude account. Connect them once per account in claude.ai Settings → Connectors.
- Local MCP servers configured in Claude Code are not tied to an account and keep working after a switch.
- macOS only.

## Uninstall

```sh
rm ~/bin/claude-acc
rm -rf ~/.claude-accounts
for s in $(security dump-keychain | grep -o '"claude-acc-[^"]*"' | tr -d '"' | sort -u); do
  security delete-generic-password -s "$s"
done
```

Your current Claude Code login is not touched.

## Disclaimer

This is an unofficial tool and is not affiliated with Anthropic. It relies on how Claude Code stores its login and on internal endpoints, which can change in any release and break it. Use it with accounts you own, and follow the terms of each account's plan.

## License

MIT
