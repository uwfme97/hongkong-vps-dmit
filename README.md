# How to Create RSA Key on Linux: A Complete Step-by-Step Guide for Secure Server Access

If you've ever typed a password every time you SSH into your Linux server, you already know the drill — it's not just annoying, it's a security risk. RSA keys change that entirely. One setup, and you log in instantly, with cryptographic security that makes brute-force attacks practically pointless.

This guide walks you through how to create an RSA key on Linux from scratch — generating the key pair, copying it to your server, and locking things down properly. Whether you're managing a personal VPS or spinning up cloud infrastructure, this is the kind of setup every Linux admin should have done yesterday.

---

## What Is an RSA Key and Why Should You Care?

RSA (Rivest–Shamir–Adleman) is a public-key cryptographic algorithm. In the SSH context, it creates two mathematically linked keys:

- **Private key** — stays on your local machine, never shared
- **Public key** — placed on the remote server

When you connect, the server sends a challenge encrypted with your public key. Your machine decrypts it using the private key. If the decryption matches, you're in — no password needed, and no credential to intercept.

> **Why RSA over password auth?** Passwords can be guessed, phished, or leaked. RSA keys can't be brute-forced in any practical timeframe. A 4096-bit RSA key gives you roughly the security equivalent of a 140-character random password.

This matters even more when you're running a publicly accessible server. Leave SSH open with password auth, and within hours you'll see thousands of login attempts from bots scanning the internet. RSA keys shut that door entirely.

---

## Prerequisites

Before you start, make sure you have:

- A Linux machine (local or remote) — Ubuntu, Debian, CentOS, Fedora, Arch — all work the same
- OpenSSH installed (`ssh`, `ssh-keygen`, `ssh-copy-id`)
- A remote server you want to connect to (more on choosing one below)
- Terminal access

Check if OpenSSH is available:

bash
ssh -V


You should see something like `OpenSSH_8.9p1`. If not, install it:

bash
# Ubuntu/Debian
sudo apt update && sudo apt install openssh-client

# CentOS/RHEL
sudo dnf install openssh-clients


---

## Step 1: Generate an RSA Key Pair

Open your terminal and run:

bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"


**Breaking down the flags:**

| Flag | Meaning |
|------|---------|
| `-t rsa` | Key type: RSA |
| `-b 4096` | Key size: 4096 bits (stronger than the 2048-bit default) |
| `-C "comment"` | A label — usually your email, helps identify the key |

**What happens next:**


Generating public/private rsa key pair.
Enter file in which to save the key (/home/user/.ssh/id_rsa):


Press **Enter** to accept the default location (`~/.ssh/id_rsa`), or type a custom path if you're managing multiple keys.


Enter passphrase (empty for no passphrase):
Enter same passphrase again:


Adding a passphrase encrypts your private key locally. Even if someone steals the file, they can't use it without the passphrase. For personal machines, this is a good habit. For automated scripts or CI/CD pipelines, you may leave it empty.

After completion, you'll see output like:


Your identification has been saved in /home/user/.ssh/id_rsa
Your public key has been saved in /home/user/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx your_email@example.com
The key's randomart image is:
+---[RSA 4096]----+
|        .o+*=o.  |
...


Your two files are now created:
- `~/.ssh/id_rsa` — **private key** (guard this like a password)
- `~/.ssh/id_rsa.pub` — **public key** (safe to share, goes to the server)

---

## Step 2: Copy the Public Key to Your Server

Now you need to place the public key on the remote server. The easiest method is `ssh-copy-id`:

bash
ssh-copy-id -i ~/.ssh/id_rsa.pub username@server_ip


This command:
1. Connects to the server using your password (one last time)
2. Appends your public key to `~/.ssh/authorized_keys` on the server
3. Sets the correct permissions automatically

You'll see:


Number of key(s) added: 1

Now try logging into the machine, with: "ssh 'username@server_ip'"


**If `ssh-copy-id` isn't available** (some minimal Linux installs lack it), use this manual method:

bash
cat ~/.ssh/id_rsa.pub | ssh username@server_ip "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"


It does the same thing — pipe your public key over SSH and append it to `authorized_keys`.

---

## Step 3: Test Your Key-Based Login

bash
ssh -i ~/.ssh/id_rsa username@server_ip


If everything is set up correctly, you're in — no password prompt. The first time, your terminal might ask you to confirm the server's fingerprint:


The authenticity of host 'server_ip' can't be established.
RSA key fingerprint is SHA256:xxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])?


Type `yes`. After that, subsequent connections skip this check.

---

## Step 4: Disable Password Authentication (Harden Your Server)

Once RSA key login is confirmed working, lock out password-based SSH entirely. This is the most impactful security step you can take for a public-facing server.

On the **remote server**, edit the SSH daemon config:

bash
sudo nano /etc/ssh/sshd_config


Find and update these lines:


PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM no
PermitRootLogin prohibit-password


Save the file, then restart SSH:

bash
sudo systemctl restart sshd


> **Warning:** Before restarting SSH, confirm in a separate terminal session that your key-based login works. If you lock yourself out, you'll need console access to recover.

---

## Managing Multiple SSH Keys

If you work with multiple servers or services (GitHub, GitLab, different VPS providers), you'll quickly accumulate multiple key pairs. Manage them cleanly with `~/.ssh/config`:


Host myserver
    HostName 192.168.1.100
    User ubuntu
    IdentityFile ~/.ssh/id_rsa_myserver

Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_github


Now instead of typing the full `ssh -i ~/.ssh/id_rsa_myserver ubuntu@192.168.1.100`, you just use:

bash
ssh myserver


---

## Using ssh-agent to Avoid Repeated Passphrase Prompts

If you set a passphrase on your private key, you'd normally need to enter it every session. `ssh-agent` caches it:

bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa


Enter your passphrase once. For the rest of the session, SSH uses the cached key automatically. On most modern Linux desktops (GNOME Keyring, KDE Wallet), this happens transparently at login.

---

## Choosing the Right VPS for Your SSH Setup

You've got the key pair — now you need a server worth connecting to. If you're shopping for a Linux VPS to practice on, run development workloads, or host production services, the underlying network quality matters as much as the specs.

[DMIT](https://www.dmit.io/aff.php?aff=18446) is a VPS provider that's earned a reputation particularly among users who need reliable connectivity between the US and Asia-Pacific. They run CN2 GIA and CMIN2 optimized routing — which means dramatically lower latency and packet loss compared to generic transit providers.

Their infrastructure runs on AMD EPYC processors with NVMe SSD storage, which translates to consistently fast I/O — important when you're doing anything beyond basic web hosting.

### DMIT VPS Plan Overview

| Series | Routing | Starting Price | Best For |
|--------|---------|----------------|----------|
| 👉 [LAX Tier 1](https://www.dmit.io/aff.php?aff=18446) | International 4-10Gbps | ~$36.90/yr | General use, dev servers |
| 👉 [LAX Eyeball](https://www.dmit.io/aff.php?aff=18446) | CMIN2 optimized | Custom pricing | Asia-Pacific users |
| 👉 [HKG Premium](https://www.dmit.io/aff.php?aff=18446) | CN2 GIA | ~$298/yr | Low-latency China routing |
| 👉 [Tokyo Tier 1](https://www.dmit.io/aff.php?aff=18446) | Japan transit | ~$36.90/yr | Japan/APAC workloads |

**Active promotions:**
- **LAX series**: Code `LAX-EB-LAUNCH-NON-MONTHLY-RECURRING-20OFF` — 20% off for life on quarterly or annual plans
- **Tokyo series**: Code `2025-TYO-T1-HI-GSL-NON-MONTHLY-30OFF` — 30% off for life on quarterly or annual plans

The lifetime discount structure is particularly useful — you lock in a lower rate permanently rather than getting an introductory price that jumps at renewal.

👉 [Browse all DMIT plans and current promotions](https://www.dmit.io/aff.php?aff=18446)

---

## Copying Your RSA Key to a DMIT VPS

Once you've provisioned a DMIT VPS, the standard SSH key setup applies directly. DMIT provides root access credentials via email after deployment. Here's the exact flow:

bash
# 1. Generate your key (if not done already)
ssh-keygen -t rsa -b 4096 -C "your@email.com"

# 2. Copy to your DMIT server
ssh-copy-id -i ~/.ssh/id_rsa.pub root@your_dmit_server_ip

# 3. Test login
ssh root@your_dmit_server_ip

# 4. Disable password auth
sudo nano /etc/ssh/sshd_config
# Set: PasswordAuthentication no
sudo systemctl restart sshd


From that point forward, your DMIT server is accessible only via your RSA key — no credential exposure, no brute-force risk.

---

## Troubleshooting Common RSA Key Issues

### "Permission denied (publickey)"

Most common cause: wrong file permissions. SSH is strict about this.

bash
# On local machine
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub

# On remote server
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys


### Key not being used

Check that the right key is specified:

bash
ssh -v -i ~/.ssh/id_rsa username@server_ip


The `-v` flag shows verbose output including which keys are being tried and why connections fail.

### Multiple keys causing confusion

If you have many keys in `~/.ssh/`, SSH tries them all up to a limit. Use `-i` explicitly or configure `~/.ssh/config` to assign keys per host.

### Passphrase forgotten

There's no recovery. Generate a new key pair, copy the new public key to the server, and replace the old one in `authorized_keys`.

---

## Frequently Asked Questions

**Can I use Ed25519 instead of RSA?**  
Yes — Ed25519 is the modern alternative. It offers equivalent security with shorter keys and faster operations: `ssh-keygen -t ed25519`. RSA 4096 is still widely supported and perfectly secure; Ed25519 is just newer and leaner.

**How many keys can I have in `authorized_keys`?**  
No hard limit. Each line is one public key. You can add keys for different machines, team members, or CI systems.

**Should I use the same key for everything?**  
No. Use separate key pairs for different services. If one gets compromised, the blast radius is contained.

**What's the difference between RSA 2048 and 4096?**  
4096-bit RSA provides approximately double the key length of 2048-bit. Both are currently unbreakable in practice, but 4096-bit provides a larger security margin against future advances in computing — with a negligible performance cost.

**Can I rotate SSH keys without losing access?**  
Yes. Add the new public key to `authorized_keys` first. Test the new key. Then remove the old key. Never remove the old key before confirming the new one works.

---

## Final Thoughts

Creating an RSA key on Linux takes about three minutes. The security improvement it provides over password authentication is substantial and immediate — particularly on any server exposed to the public internet.

The workflow is always the same:

1. `ssh-keygen -t rsa -b 4096` — generate the pair
2. `ssh-copy-id` — push the public key to the server
3. Test the connection
4. Disable `PasswordAuthentication` in `sshd_config`

Once this is muscle memory, you'll never go back to passwords. And if you're setting up a new Linux server to practice on, 👉 [DMIT's VPS plans](https://www.dmit.io/aff.php?aff=18446) are worth a look — solid hardware, multiple network tiers, and competitive lifetime pricing that doesn't bait-and-switch you at renewal.

Get your keys set up, lock your server down, and stop worrying about unauthorized access.
