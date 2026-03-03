# awsp - AWS Profile Manager

A simple, portable CLI tool for managing AWS profiles with SSO support. Built for teams using AWS SSO with multiple environments (dev, uat, prod, etc.).

## Install

```bash
curl -sL https://raw.githubusercontent.com/funactuator/tools/main/awsp/awsp -o ~/.aws/awsp && chmod +x ~/.aws/awsp && ~/.aws/awsp install
```

> Tip: Inspect the script before running it:
> ```bash
> curl -sL https://raw.githubusercontent.com/funactuator/tools/main/awsp/awsp | cat
> ```

Restart your terminal (or `source ~/.zshrc`) after install.

## Usage

```
awsp <command>

  list    | ls     List all profiles (active one highlighted)
  use     | set    Switch active profile (interactive picker or pass name)
  status  | st     Show current profile, account, role & SSO state (default)
  login            Force SSO login for active profile
  env              Print export statements for credentials
  install          Set up PATH + alias in shell config
  help             Show usage
```

## Examples

```bash
awsp use                   # Interactive profile picker (fzf or numbered menu)
awsp use irame-app-dev     # Set profile directly
awsp status                # See what profile is active and if SSO is valid
awsp list                  # See all profiles with account & role info
awsp login                 # Trigger SSO login
awsenv                     # Load credentials into current shell session
```

## How it works

- Reads profiles dynamically from `~/.aws/config` — nothing is hardcoded
- Stores the active profile in `~/.aws/active-profile`
- Uses `aws configure export-credentials` under the hood
- Auto-triggers `aws sso login` if your session has expired
- Uses `fzf` for interactive selection if available, falls back to a numbered menu

## Security

- No hardcoded credentials or secrets
- Only uses standard AWS CLI commands (`aws configure`, `aws sso login`)
- Your credentials come from your local `~/.aws` setup — nothing leaves your machine

## Requirements

- AWS CLI v2
- Configured AWS SSO profiles in `~/.aws/config`
- `fzf` (optional, for fuzzy profile picker)
