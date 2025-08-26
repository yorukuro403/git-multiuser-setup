# git-multiuser-setup

## INTRODUCTION

This guide explains how to configure multiple Git accounts (e.g., personal, work, school) using SSH. Although GitHub is used as the default example, this setup works with any Git hosting service. HTTPS-based workflows are not supported in this guide—SSH is required.

The configuration is divided into three main steps:

1. Generate SSH keys (or reuse existing ones).
2. Configure ~/.ssh/config to define custom hosts for each account.
3. Add a custom git-clone shell function to automatically set the correct user.name and user.email per repository.


## Generating SSH keys

For each account, create a dedicated SSH key. The recommended algorithm is ed25519:

```bash
ssh-keygen -t ed25519 -C "your-email@example.com" -f ~/.ssh/id_account-1
```

- Replace your-email@example.com with the email tied to your Git account.
- Replace id_account-1 with a descriptive name (e.g., id_personal, id_work).
- This will create two files: id_account-1 (private key) and id_account-1.pub (public key).

Repeat the command for each account you want to configure.

### Add the public key to your account.

For GitHub users:

1. Copy the content of the `.pub` file, e.g.: 
```bash
cat ~/.ssh/id_account-1.pub
```

2. Go to *GitHub -> Settings -> SSH and GPG Keys -> New SSH Key.*
3. Paste the key and save.

If using a different Git hosting service, follow their instructions to add SSH keys.


## Configure SSH hosts.

Edit (or create) `~/.ssh/config` and add an entry for each account:

```bash
# Account 1
Host account-1
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_account-1

# Account 2
Host account-2
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_account-2
```

Replace `account-1`, `account-2` with descriptive aliases.
`HostName` can be `github.com` or another git host.
`IdentityFile` *MUST* point to the *private* key you generated.

When cloning, instead of using git@github.com:username/repo-name.git, you'll use:

```bash
git@account-1:username/repo.git
git@account-2:username/repo.git
```


## Configuring the `git-clone` function.

To automatically set the correct identity per repository, add the following function to your `~/.zshrc` (or `~/.bashrc` if you use Bash):

```bash
git-clone() {
  url="$1"
  dir="${2:-$(basename "${url%%.git}" )}"

  git clone "$url" "$dir" || return 1
  cd "$dir" || return 1

  # Detect the host alias after '@' and before ':'
  local host="${url#*@}"; host="${host%%:*}"

  case "$host" in
    account-1)
      git config user.name "Your Name for Account 1"
      git config user.email "your-email-account-1@example.com"
      ;;
    account-2)
      git config user.name "Your Name for Account 2"
      git config user.email "your-email-account-2@example.com"
      ;;
  esac

  git config --show-origin user.name
  git config --show-origin user.email
}
```

Update each case with the correct `user.name` and `user.email` for that account.

Restart your shell or run source `~/.zshrc` after editing.

## Verification

1. Clone this repository itself (replace with the correct host alias you configured):

```bash
git-clone git@account-1:your-username/multi-git-accounts.git
```

2. After the clone finishes, the function will automatically change into the repo. Run:

```bash
ls .
git config user.name
git config user.email
```

3. Move back to the parent directory and check again (you should see the global identity): 

```bash
git config user.name
git config --global user.name
git config user.email
git config --global user.email
```

This confirms that the local repo is using automatically the correct account identity while the global remains unchanged. 


## Troubleshooting:

### Wrong user/email inside the repo

Make sure you are cloning using the correct alias (git@account-1:...), not the default git@github.com:....

Verify that the case entries in your git-clone function match the host alias names in your ~/.ssh/config.

### SSH key not recognized

Ensure your private key file exists and matches the one in ~/.ssh/config.

Check file permissions:

```bash
chmod 600 ~/.ssh/id_account-1
chmod 644 ~/.ssh/id_account-1.pub
```

### Still using the global Git identity

Run `git config --show-origin user.name` inside the repo.

If it points to your global config, double-check that the `git-clone` function is properly loaded in your shell (type `git-clone`).

## Next Steps

This guide covers only the *manual setup*. Future work will include an automated setup script to simplify the process further.
