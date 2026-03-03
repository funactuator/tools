# iaws - AWS Profile Manager

A simple, portable CLI tool for managing AWS profiles with SSO support. Built for teams using AWS SSO with multiple environments (dev, uat, prod, etc.).

## Install

```bash
curl -sL https://raw.githubusercontent.com/funactuator/tools/main/iaws/iaws -o ~/.aws/iaws && chmod +x ~/.aws/iaws && ~/.aws/iaws install
```

Restart your terminal (or `source ~/.zshrc`) after install.

> **Tip:** Inspect the script before running it:
> ```bash
> curl -sL https://raw.githubusercontent.com/funactuator/tools/main/iaws/iaws | cat
> ```

## Usage

```
iaws <command>

  list    | ls        List all profiles (active one highlighted)
  use     | set       Switch active profile (interactive picker or pass name)
  status  | st        Show current profile, account, role & SSO state (default)
  login               Force SSO login for active profile
  load                Load credentials into current shell session
  env                 Print credential export statements
  upgrade | update    Upgrade iaws to latest version from GitHub
  uninstall | remove  Remove iaws and clean up shell config
  install             Set up PATH + shell function in shell config
  help                Show usage
```

## Examples

```bash
iaws use                   # Interactive profile picker (fzf or numbered menu)
iaws use irame-app-dev     # Set profile directly
iaws status                # See what profile is active and if SSO is valid
iaws list                  # See all profiles with account & role info
iaws login                 # Trigger SSO login
iaws load                  # Load credentials into current shell session
iaws upgrade               # Upgrade to latest version
iaws uninstall             # Remove iaws completely
```

## How it works

- Reads profiles dynamically from `~/.aws/config` — nothing is hardcoded
- Stores the active profile in `~/.aws/active-profile`
- Uses `aws configure export-credentials` under the hood
- Auto-triggers `aws sso login` if your session has expired
- Uses `fzf` for interactive selection if available, falls back to a numbered menu
- `iaws load` works via a shell function installed in your rc file — this is how it can export credentials into your current shell session

## Upgrade

```bash
iaws upgrade
# or
iaws update
```

Downloads the latest version from GitHub, validates it, and replaces the existing script. Shows a version diff before updating.

## Uninstall

```bash
iaws uninstall
# or
iaws remove
```

Asks for confirmation, then removes the script, cleans up your shell config, and deletes the active profile state file.

## Security

- No hardcoded credentials or secrets
- Only uses standard AWS CLI commands (`aws configure`, `aws sso login`)
- Your credentials come from your local `~/.aws` setup — nothing leaves your machine
- `upgrade` validates the downloaded file before replacing the existing script

## Requirements

- AWS CLI v2
- Configured AWS SSO profiles in `~/.aws/config`
- `curl` (for install/upgrade)
- `fzf` (optional, for fuzzy profile picker)
