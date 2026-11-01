# GitHub Integration & Agent Automation Setup Guide

This guide details how to configure GitHub CLI (`gh`) or a Personal Access Token (PAT) on Windows so AI coding assistants can programmatically:
1. Search and read existing tasks/issues.
2. Create and prioritize issues on your GitHub Projects board.
3. Check out feature branches.
4. Open pull requests linked to tracking issues.
5. Review CI checks and squash-merge PRs upon completion.

---

## Step 1: Install GitHub CLI (`gh`)

On your Windows system, open PowerShell and install GitHub CLI via winget:

```powershell
winget install --id GitHub.cli
```
*(Or download the installer directly from [cli.github.com](https://cli.github.com/)).*

After installation, close and reopen your PowerShell terminal so `gh` is in your `PATH`.

---

## Step 2: Authenticate GitHub CLI

Run the interactive login command:

```powershell
gh auth login
```

Follow the prompts:
1. **What account do you want to log into?** -> `GitHub.com`
2. **What is your preferred protocol for Git operations?** -> `HTTPS`
3. **Authenticate Git with your GitHub credentials?** -> `Yes`
4. **How would you like to authenticate GitHub CLI?** -> `Login with a web browser`
5. Press Enter. It will display an **8-character one-time code** and open your default browser.
6. Enter the code in your browser and authorize GitHub CLI.

Verify authentication is successful:

```powershell
gh auth status
```

---

## Step 3: Headless / Environment Variable Option (Alternative)

If you prefer not using browser login, or want the AI agent to have standalone access:
1. Go to **GitHub Settings -> Developer Settings -> Personal Access Tokens -> Fine-grained tokens** ([github.com/settings/tokens?type=beta](https://github.com/settings/tokens?type=beta)).
2. Click **Generate new token**.
3. Select your target repository (e.g. `my-app`).
4. Set Repository permissions:
   - **Issues**: Read and write
   - **Pull requests**: Read and write
   - **Contents**: Read and write
   - **Projects**: Read and write
5. Copy the generated token (`github_pat_...`).
6. Set it as an environment variable in PowerShell:
   ```powershell
   [System.Environment]::SetEnvironmentVariable('GH_TOKEN', 'your_token_here', 'User')
   ```

---

## Step 4: Verification Test

Once authenticated, test creating a private repository or listing repositories:

```powershell
gh repo list --limit 5
```

Once connected, I will be able to run `gh issue list`, `gh issue create`, `gh pr create`, and `gh pr merge` directly on your repository!
