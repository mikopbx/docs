---
description: Connecting to MikoPBX via SSH using Terminal (Linux/MacOS)
---

# Connecting via SSH (Linux/MacOS)

## Generating and Linking the Key

1. First, generate an SSH key. Open a terminal and enter the following command to create it:

```
ssh-keygen -t ed25519 -C "example.powershell@gmail.com"
```

In this example, an "ed25519" key will be generated with the comment "[example.powershell@gmail.com](mailto:example.powershell@gmail.com)" for future identification.

You can also specify a path where the key files will be saved. By default, keys are stored in `~/.ssh/id_ed25519.pub`. To set a custom path, add `-f` and specify it, for example:

```
ssh-keygen -t ed25519 -f ~/.ssh/my_new_key
```

{% hint style="info" %}
By default, the key is saved at `/Users/Username/.ssh/id_ed25519.pub on macOS or /home/Username/.ssh/id_ed25519.pub` on Linux.
{% endhint %}

2. Next, you need to retrieve and copy the newly created public key. Enter the command:

```
cat ~/.ssh/id_ed25519.pub
```

3. Open the MikoPBX web interface and go to **System** → **General Settings**:
4. Paste your public key into the **"SSH Authorized Keys"** field:

## Connecting via SSH

To connect via SSH, run the following command in your terminal:

```
ssh -i ~/.ssh/id_ed25519 root@mikopbxipadress
```

Replace the following with your actual parameters:

* The path to your SSH key (if different from default).
* The IP address of your MikoPBX in place of `mikopbxipadress`.

You will then be prompted for the SSH passphrase if you set one. Upon successful authentication, you will be connected to the MikoPBX console via SSH:
