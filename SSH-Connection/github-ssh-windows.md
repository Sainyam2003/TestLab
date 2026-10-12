# Set up GitHub SSH on Windows

This configures Git to connect to GitHub over SSH using Windows PowerShell. Your GitHub account already exists; this guide does not create another account.

## 1. Open PowerShell and check for an existing key

```powershell
Get-ChildItem "$env:USERPROFILE\.ssh" -ErrorAction SilentlyContinue
```

If you see both `id_ed25519` and `id_ed25519.pub`, you can reuse that key: skip to **Add the key to GitHub**. If those files are absent, create a key:

```powershell
ssh-keygen -t ed25519 -C "your_github_email@example.com"
```

Replace the email with the email associated with your GitHub account. Press Enter to accept the default file location (`C:\Users\<you>\.ssh\id_ed25519`). Set a passphrase when prompted if you want added protection; remember it.

## 2. Start the Windows SSH agent and load the key

In PowerShell **Run as administrator**, enable and start the agent service:

```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```

Close the elevated window. In your regular PowerShell window, add the key to the agent:

```powershell
ssh-add "$env:USERPROFILE\.ssh\id_ed25519"
```

If you used a different key filename, substitute it above. Enter the key passphrase if prompted.

## 3. Add the public key to your existing GitHub account

Copy the public key (this is safe to share with GitHub; **never share the private file without `.pub`**):

```powershell
Get-Content "$env:USERPROFILE\.ssh\id_ed25519.pub" | Set-Clipboard
```

Open [GitHub SSH key settings](https://github.com/settings/ssh/new), sign in to your existing account, enter a title such as `Windows PC`, leave key type as **Authentication Key**, paste the clipboard contents, and select **Add SSH key**.

If GitHub CLI (`gh`) is installed and already authenticated to the right account, you can add the key from PowerShell instead:

```powershell
gh auth status
gh ssh-key add "$env:USERPROFILE\.ssh\id_ed25519.pub" --type authentication --title "Windows PC"
```

## 4. Test the connection

```powershell
ssh -T git@github.com
```

On the first connection, verify the displayed GitHub host fingerprint against [GitHub's published SSH key fingerprints](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints), then type `yes` if it matches. Success looks like `Hi USERNAME! You've successfully authenticated, but GitHub does not provide shell access.` The command may return exit code 1 despite this success message.

## 5. Use SSH for a repository

For a new clone, copy the repository's **SSH** URL from its GitHub **Code** menu, then run it in PowerShell:

```powershell
git clone git@github.com:OWNER/REPOSITORY.git
```

For an existing local repository, change its `origin` URL (run this from that repository's folder):

```powershell
git remote set-url origin git@github.com:OWNER/REPOSITORY.git
git remote -v
```

Replace `OWNER/REPOSITORY` with the repository path shown on GitHub. SSH authentication grants access only to repositories your GitHub account is already allowed to use.

## Troubleshooting

- **`Permission denied (publickey)`**: confirm the `.pub` key was added to the correct GitHub account, then run `ssh-add -l` to check that the agent has a key loaded. If needed, run `ssh-add "$env:USERPROFILE\.ssh\id_ed25519"` again.
- **Key file already exists**: do not overwrite it unless you intend to replace that key. Reuse the existing key or choose a new filename when running `ssh-keygen`.
- **`ssh` or `ssh-keygen` not found**: install or enable the Windows OpenSSH Client feature, then reopen PowerShell.

Official references: [GitHub: Generating a new SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent), [Adding a key to your account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account), and [Testing the connection](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/testing-your-ssh-connection).
