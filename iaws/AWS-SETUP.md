# AWS Prerequisites Setup Guide

This guide covers everything you need before using `iaws`. If `iaws` pointed you here, work through each section in order.

---

## 1. Install AWS CLI v2

`iaws` requires the AWS CLI v2. v1 is not supported.

**macOS (Homebrew)**
```bash
brew install awscli
```

**macOS (official installer)**
```bash
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o AWSCLIV2.pkg
sudo installer -pkg AWSCLIV2.pkg -target /
```

**Linux**
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
unzip awscliv2.zip
sudo ./aws/install
```

**Verify the install**
```bash
aws --version
# Expected: aws-cli/2.x.x ...
```

---

## 2. Configure AWS SSO

AWS SSO (IAM Identity Center) lets you log in once via your browser and get temporary credentials for all your accounts and roles.

### 2a. Find your SSO details

You need three things from your AWS admin or the IAM Identity Center console:

| Field | Example | Where to find it |
|---|---|---|
| SSO start URL | `https://mycompany.awsapps.com/start` | IAM Identity Center → Settings → User portal URL |
| SSO region | `us-east-1` | The region where IAM Identity Center is deployed |
| Account ID | `123456789012` | AWS console → top-right account menu |
| Role/permission set name | `AdministratorAccess` | IAM Identity Center → Permission sets |

### 2b. Add a profile to `~/.aws/config`

Open `~/.aws/config` in any editor and add a block like this for each account/role combination:

```ini
[profile my-dev]
sso_start_url = https://mycompany.awsapps.com/start
sso_region    = us-east-1
sso_account_id = 111111111111
sso_role_name  = DeveloperAccess
region         = ap-south-1
output         = json

[profile my-prod]
sso_start_url = https://mycompany.awsapps.com/start
sso_region    = us-east-1
sso_account_id = 222222222222
sso_role_name  = ReadOnlyAccess
region         = ap-south-1
output         = json
```

> **Note:** The `[profile ...]` prefix is required for all non-default profiles. The section header is literally `[profile my-dev]`, not `[my-dev]`.

### 2c. Log in to SSO

```bash
aws sso login --profile my-dev
```

This opens a browser tab. Approve the request. After approval, the CLI confirms login and stores a session token in `~/.aws/sso/cache/`.

### 2d. Verify it works

```bash
aws sts get-caller-identity --profile my-dev
```

Expected output:
```json
{
    "UserId": "AROA...:your.name@company.com",
    "Account": "111111111111",
    "Arn": "arn:aws:sts::111111111111:assumed-role/DeveloperAccess/your.name@company.com"
}
```

---

## 3. Using `aws configure sso` (alternative wizard)

If you prefer a guided setup instead of editing the file manually:

```bash
aws configure sso
```

The wizard prompts for your SSO URL, region, account, and role, then writes the `~/.aws/config` block for you. It also triggers the browser login flow automatically.

---

## 4. Multiple profiles — recommended structure

For teams using multiple environments, a clean naming convention helps:

```ini
[profile acme-dev]
sso_start_url  = https://acme.awsapps.com/start
sso_region     = us-east-1
sso_account_id = 111111111111
sso_role_name  = DeveloperAccess
region         = ap-south-1

[profile acme-uat]
sso_start_url  = https://acme.awsapps.com/start
sso_region     = us-east-1
sso_account_id = 222222222222
sso_role_name  = DeveloperAccess
region         = ap-south-1

[profile acme-prod]
sso_start_url  = https://acme.awsapps.com/start
sso_region     = us-east-1
sso_account_id = 333333333333
sso_role_name  = ReadOnlyAccess
region         = ap-south-1
```

All profiles that share the same `sso_start_url` share a single SSO login session — logging in with one logs you in to all of them until the session expires.

---

## 5. Session expiry and re-login

SSO sessions expire (typically after 8–12 hours, set by your admin). When expired:

```bash
iaws login          # triggers aws sso login for the active profile
# or
aws sso login --profile my-dev
```

`iaws load` and `iaws env` auto-trigger login if the session has expired.

---

## 6. Verify your config before using `iaws`

```bash
# List all configured profiles
cat ~/.aws/config | grep '^\[profile'

# Check a specific profile's details
aws configure list --profile my-dev

# Confirm SSO credentials work
aws sts get-caller-identity --profile my-dev
```

Once at least one profile exists and `aws sts get-caller-identity` works, you're ready for `iaws`.

---

## Common errors

| Error | Cause | Fix |
|---|---|---|
| `command not found: aws` | AWS CLI not installed | See [Section 1](#1-install-aws-cli-v2) |
| `No profiles found in ~/.aws/config` | Config file missing or empty | See [Section 2b](#2b-add-a-profile-to-awsconfig) |
| `Error loading SSO Token` | SSO session expired | Run `iaws login` |
| `InvalidGrantException` | SSO re-auth required | Run `aws sso login --profile <name>` |
| `Profile not found` | Typo in profile name or missing `[profile ...]` block | Check `~/.aws/config` headers |

---

## References

- [AWS CLI v2 install docs](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- [Configuring AWS CLI with SSO](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html)
- [IAM Identity Center (SSO) user guide](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html)
