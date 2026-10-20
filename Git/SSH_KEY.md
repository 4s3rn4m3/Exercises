# SSH Configuration for GitHub

SSH uses asymmetric cryptography, which is more secure than password authentication because it allows you to prove your identity without ever sending your private credentials over the network. This process relies on a key pair: a public key, which you share with GitHub, and a private key, which stays securely on your local machine in `~/.ssh`.

The email address used during key generation acts as a comment or label. While it doesn't affect the encryption, it helps you identify which key is which when managing multiple keys in your GitHub settings.

## Step 1: Generate Your SSH Keys

Open your terminal and run the following command. Replace the email with the one associated with your GitHub account.

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

When prompted to "Enter a file in which to save the key," press Enter to accept the default location. You can also add a passphrase for an extra layer of security.

## Step 2: Add Your Key to the SSH Agent

The SSH agent manages your keys so you don't have to type your passphrase every time you use Git.

```bash
# Start the ssh-agent in the background
eval "$(ssh-agent -s)"

# Add your private key to the agent
ssh-add ~/.ssh/id_ed25519
```

## Step 3: Add the Public Key to GitHub

You must provide GitHub with your public key to authorize your machine.

1. Copy your public key to your clipboard:
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```
2. Log in to GitHub and go to **Settings** > **SSH and GPG keys**.
3. Click **New SSH key**, give it a descriptive title (e.g., "Work Laptop"), and paste the key into the box.
4. Click **Add SSH key**.

## Step 4: Configure Multiple Accounts

If you have separate accounts (e.g., personal and university), you cannot use the same key for both. Generate a second key with a different name (e.g., `id_ed25519_uni`) and add it to your second GitHub account. Then, create or edit your `~/.ssh/config` file to route the traffic correctly:

```bash
# Personal account
Host github.com-per
    Hostname github.com
    User git
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_ed25519

# University account
Host github.com-uni
    Hostname github.com
    User git
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_ed25519_uni
```

## Step 5: Using Multiple Accounts with Git

When you use a config file with custom hosts, you must change the URL when cloning a repository. Instead of using `github.com`, use the alias you defined in the config file.

To clone using your personal account:

```bash
git clone git@github.com-per:username/repo-name.git
```

To clone using your university account:

```bash
git clone git@github.com-uni:uni-username/repo-name.git
```

## Step 6: Test Your Connection

To verify that everything is working, run the following command:

```bash
ssh -T git@github.com
```

If successful, you will see a message like: `Hi username! You've successfully authenticated, but GitHub does not provide shell access.`

##### **References**

- [How to set up ssh keys for github](https://dev.to/taujor/how-to-set-up-ssh-for-github-on-linux-a-step-by-step-guide-1jmh)
- [https://www.geeksforgeeks.org/computer-networks/introduction-to-ssh-secure-shell-keys/]
