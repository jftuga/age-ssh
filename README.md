# age-ssh

Encrypt and decrypt files using [age](https://github.com/FiloSottile/age) with SSH ed25519 keys.

This is a simple wrapper script that simplifies the `age` command-line interface for common encryption/decryption tasks using your existing SSH keys.

## Requirements

- [age](https://github.com/FiloSottile/age) - Install via `brew install age` (macOS) or your package manager
- Bash 3.2+ (included with macOS)
- SSH ed25519 key pair

## Installation

```bash
# Clone the repository
git clone https://github.com/jftuga/age-ssh.git

# Make the script executable and copy to your PATH
chmod +x age-ssh/age-ssh
cp age-ssh/age-ssh ~/.local/bin/
# or
sudo cp age-ssh/age-ssh /usr/local/bin/
```

## Usage

```
Usage: age-ssh <e|d> [-f] [-k key] <file>
       age-ssh -v
       age-ssh -h

  e [-f] <file>      Encrypt file using ~/.ssh/id_ed25519.pub
  d [-f] <file.age>  Decrypt file using ~/.ssh/id_ed25519
  -f                 Force overwrite if target file exists
  -k <key>           Use custom SSH key (appends .pub for public key)
  -h                 Show this help message
  -v                 Show version information
```

### Basic Examples

```bash
# Encrypt a file (creates secret.txt.age)
age-ssh e secret.txt

# Decrypt a file (creates secret.txt)
age-ssh d secret.txt.age

# Force overwrite if target exists
age-ssh e -f secret.txt

# Use a different SSH key
age-ssh e -k ~/.ssh/my_other_key secret.txt
```

## Scenario: Secure File Transfer Between Alice and Bob

This example demonstrates how Alice can send an encrypted file to Bob using SSH keys.

### Step 1: Generate SSH Keys (if needed)

Both Alice and Bob need ed25519 SSH key pairs. If they don't already have them:

**Bob generates his key pair:**
```bash
ssh-keygen -t ed25519 -C "bob@example.com"
```

This creates:
- `~/.ssh/id_ed25519` (private key - keep this secret!)
- `~/.ssh/id_ed25519.pub` (public key - safe to share)

**Alice generates her key pair (if she doesn't have one):**
```bash
ssh-keygen -t ed25519 -C "alice@example.com"
```

### Step 2: Bob Sends His Public Key to Alice

Bob shares his public key with Alice. He can send it via email, chat, or any other method since public keys are safe to share:

```bash
# Bob views his public key
cat ~/.ssh/id_ed25519.pub
```

Output looks like:
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... bob@example.com
```

Bob sends this to Alice, who saves it as `bob_id_ed25519.pub` in a convenient location (e.g., `~/keys/bob_id_ed25519.pub`).

### Step 3: Alice Encrypts the File for Bob

Alice encrypts the file using Bob's public key. Note that the `-k` argument expects the private key path (without `.pub`), and the script automatically appends `.pub` for encryption:

```bash
# Alice encrypts secret.txt for Bob
# She provides the path WITHOUT the .pub extension
age-ssh e -k ~/keys/bob_id_ed25519 secret.txt
```

This creates `secret.txt.age`, which only Bob can decrypt.

### Step 4: Alice Sends the Encrypted File to Bob

Alice sends `secret.txt.age` to Bob via email, file transfer, cloud storage, etc. The file is encrypted, so it's safe to send over any channel.

### Step 5: Bob Decrypts the File

Bob receives `secret.txt.age` and decrypts it using his private key:

```bash
# Bob decrypts the file using his default private key (~/.ssh/id_ed25519)
age-ssh d secret.txt.age
```

This creates `secret.txt` with the original contents. Only Bob can decrypt this file because only he has the private key that corresponds to the public key Alice used for encryption.

## Security Notes

- **Private keys** (`id_ed25519`) should never be shared. Keep them secure with permissions `600`.
- **Public keys** (`id_ed25519.pub`) are safe to share freely.
- Encrypted `.age` files are created with `600` permissions (owner read/write only).
- Decrypted files are also created with `600` permissions.

## License

MIT License - See [LICENSE](LICENSE) for details.

## Acknowledgments

- [age](https://github.com/FiloSottile/age) by Filippo Valsorda - the underlying encryption tool
