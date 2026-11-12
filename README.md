# totp-cli

A tiny command-line tool to generate TOTP (2FA) codes. Single Python file, no dependencies. Secrets are stored in your system keyring via `secret-tool` (libsecret), never in plain files.

## Requirements

- Python 3.8+
- `secret-tool` (`sudo apt install libsecret-tools`) and a running keyring (e.g. GNOME Keyring)
- Optional: `xclip` or `wl-copy` for `--copy`

## Install

```bash
ln -s "$PWD/totp" ~/.local/bin/totp
```

## Usage

```bash
totp add github            # prompts (hidden) for the base32 secret or an otpauth:// URI
totp get github            # prints the current 6-digit code
totp get -c github         # also copies it to the clipboard
totp list                  # lists stored names
totp rm github             # deletes an entry
```

`add` deliberately has no way to pass the secret as an argument: it is always read from a hidden prompt, so it never ends up in shell history or the process list.

## Using it with Claude Code

Add a skill at `~/.claude/skills/totp/SKILL.md` telling Claude to run only `totp list` and `totp get`, and enforce it with deny rules in `~/.claude/settings.json`:

```json
"permissions": {
  "deny": [
    "Bash(*secret-tool*)",
    "Bash(*gnome-keyring*)",
    "Bash(*seahorse*)",
    "Bash(*org.freedesktop.secrets*)",
    "Bash(*totp add*)",
    "Bash(*totp rm*)"
  ]
}
```

The leading and trailing `*` also catch full paths and `bash -c` wrappers. This is a guardrail, not a sandbox: code that talks to the keyring directly could still bypass it.

## How it works

Standard RFC 6238 TOTP: HMAC-SHA1, 30-second step, 6 digits. Entries are stored in the keyring with the attributes `app=totp-cli` and `name=<name>`.
