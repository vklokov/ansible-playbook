# qBittorrent

`qbt.yml` installs `qbittorrent-nox` as a systemd service under a dedicated
system user (`qbt`). The web UI is bound to `127.0.0.1` only, so reach it
through an SSH tunnel. The torrent port is opened in ufw if ufw is installed.

Run `bootstrap.yml` first, see [README.md](README.md).

## Install

On the server, from the repository root:

```bash
ansible-galaxy collection install community.general
ansible-playbook qbt.yml
```

The playbook finishes with a check that the web UI listens on localhost only,
and fails if it does not.

Variables live at the top of `qbt.yml`: `qbt_user`, `qbt_home`,
`qbt_download_dir` (default `/srv/torrents`), `qbt_webui_port` (`8080`),
`qbt_torrent_port` (`51413`) and `qbt_manage_ufw`.

## Open the web UI

From your own machine, in a separate terminal tab:

```bash
ssh -N -L 8080:localhost:8080 deploy@<ip>
```

The tunnel stays up until you press `Ctrl+C`. Open http://localhost:8080.

The temporary `admin` password is printed in the journal, change it right away
under Tools -> Options -> Web UI:

```bash
sudo journalctl -u qbittorrent-nox | grep -i password | tail -1
```

To keep a tunnel in the background use `-fN` instead of `-N`, and stop it with
`pkill -f "ssh -fN -L 8080"` (or find the PID with `lsof -i :8080`).

## Notes

- qBittorrent rewrites its config when it stops. If you change the ports or
  paths in the playbook later, run `sudo systemctl stop qbittorrent-nox` first,
  otherwise your edits get overwritten.
- The Ubuntu 24.04 package (4.6.x) has no `--confirm-legal-notice` flag, so the
  playbook accepts the legal notice through the config (`[LegalNotice]`).
- The playbook is idempotent, running it again is safe.
