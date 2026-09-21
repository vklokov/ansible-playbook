# FTP server

`ftp.yml` installs `vsftpd` with explicit FTPS (TLS) and a single chrooted
user. Plain FTP is refused: both logins and data transfers must use TLS.

Run `bootstrap.yml` first, see [README.md](README.md).

## Install

On the server, from the repository root:

```bash
ansible-galaxy collection install community.general
ansible-playbook ftp.yml
```

The password for the FTP user is generated on the first run and saved to
`.secrets/ftp_password` (git-ignored). Later runs reuse it and never change the
user's password.

```bash
cat .secrets/ftp_password
```

Variables live at the top of `ftp.yml`: `ftp_user` (default `ftpuser`),
`ftp_root` (`/srv/ftp`), `ftp_upload_dir` (`files`), the passive port range
`ftp_pasv_min` / `ftp_pasv_max` (`40000`-`40100`), `ftp_pasv_address` and
`ftp_manage_ufw`.

## Connect

Use an FTPS client with **explicit TLS** on port 21 (FileZilla: "Require
explicit FTP over TLS"). The user lands in a read-only chroot and can write
only inside `/files`. Example with lftp:

```bash
lftp -u ftpuser -e 'set ftp:ssl-force true; set ssl:verify-certificate no' <ip>
```

The certificate is self-signed, so the client asks to trust it once.

## Firewall

Open these in ufw (the playbook does it if ufw is installed) and in your
provider's firewall:

- TCP 21
- TCP 40000-40100 (passive data connections)

If the server sits behind NAT, set `ftp_pasv_address` to the public IP.

## Notes

- The user has the `nologin` shell, so it cannot use SSH. `/usr/sbin/nologin`
  is added to `/etc/shells` because vsftpd's PAM stack rejects other shells.
- Only users listed in `/etc/vsftpd.userlist` can log in.
- To rotate the password, set a new one with `sudo passwd ftpuser` and update
  `.secrets/ftp_password`.
- The playbook is idempotent, running it again is safe. `/etc/vsftpd.conf` is
  backed up before each change.
