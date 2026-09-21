# ubuntu-bootstrap

Ansible playbook for the initial setup of a fresh Ubuntu 24.04 server.
It is meant to be cloned onto the server and run locally, by hand.

`bootstrap.yml` does the following:

- creates an admin user (`deploy` by default) with no password
- installs the public key from `ssh/rsa.pub` into that user's `authorized_keys`
- grants the user passwordless sudo
- hardens sshd: no root login, no password or keyboard-interactive auth, keys only

## Before you start

Put your **public** key in `ssh/rsa.pub`. The playbook aborts if the file is
missing or empty, so nothing is changed in that case.

## Usage

Run these on the server as root:

```bash
apt update && apt install -y ansible git
ansible-galaxy collection install ansible.posix

git clone <repo-url> ~/playbooks
cd ~/playbooks

ansible-playbook bootstrap.yml --check --diff   # optional dry run
ansible-playbook bootstrap.yml
```

`ansible.cfg` points at `inventory.ini`, which defines the `servers` group as
`localhost` with a local connection, so no SSH is involved in the run itself.
Run the commands from the repository root so `ansible.cfg` is picked up.

### Locale error

If Ansible fails with `ERROR: Ansible could not initialize the preferred locale:
unsupported locale setting`, your SSH client forwarded a locale that is not
generated on the server. Use the built-in one for the run:

```bash
export LC_ALL=C.UTF-8 LANG=C.UTF-8
```

Or generate the locale you actually use (check with `echo $LANG` and
`locale -a`), for example:

```bash
locale-gen en_US.UTF-8 && update-locale LANG=en_US.UTF-8
```

## Verify before closing your root session

Keep the current root session open until all of these pass, otherwise a mistake
can lock you out.

From your own machine, in a new terminal:

```bash
ssh deploy@<ip>                                 # works
ssh root@<ip>                                   # Permission denied (publickey)
ssh -o PubkeyAuthentication=no deploy@<ip>      # Permission denied
```

On the server, as `deploy`:

```bash
sudo whoami                                     # root
```

Effective sshd settings, as root:

```bash
sshd -T | grep -Ei 'permitrootlogin|passwordauthentication|kbdinteractive'
```

All three should report `no`.

## Other playbooks

- `qbt.yml`: qBittorrent-nox behind an SSH tunnel, see [QBT.md](QBT.md).
- `ftp.yml`: vsftpd with FTPS and one chrooted user, see [FTP.md](FTP.md).

## Notes

- The sshd drop-in is named `01-hardening.conf` so it is read before
  `50-cloud-init.conf`. sshd uses the first value it finds, so the hardening wins.
- `bootstrap.yml` is idempotent, running it again is safe.
- `admin_user` can be changed at the top of `bootstrap.yml`.
