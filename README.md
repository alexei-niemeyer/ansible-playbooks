# Ansible Playbooks — Homelab

Ansible-Playbooks für Semaphore (LXC 122 `192.168.178.93`). Deploy, Hardening
und Maintenance für Debian/Ubuntu-LXCs im 192.168.178.0/23-Netz.

## Playbooks

### `deploy-lxc.yaml` — Provisioniert einen neuen Debian-13-LXC

End-to-end-Flow (läuft auf `localhost`, ruft Proxmox- und Netbox-APIs direkt):

1. Reserviert nächste freie IP aus Netbox-IP-Range `192.168.178.200-254`
   (Status `reserved`, `dns_name = <hostname>`).
2. Holt nächste freie VMID von Proxmox.
3. Erstellt LXC via Proxmox-API auf node `proxmox` (192.168.178.10),
   Storage `zfs1`, Template `debian-13-standard`, unprivileged + nesting.
4. Setzt statische Netz-Config (eth0, ip, gw, bridge vmbr0), startet den Container.
5. Wartet auf SSH, ruft anschließend `init-lxc.yaml` für Hardening auf.
6. Patcht die Netbox-IP auf `active` + trägt `proxmox_vmid` / `proxmox_node`
   als Custom Fields ein.

**Extra-Vars (über Semaphore-Survey):**

| Variable        | Pflicht | Default | Beschreibung                     |
|-----------------|---------|---------|----------------------------------|
| `hostname`      | ja      | —       | `^[a-z0-9][a-z0-9-]{0,62}$`      |
| `cores`         | nein    | 2       | CPU-Cores                        |
| `memory`        | nein    | 2048    | RAM in MB                        |
| `disk`          | nein    | 8       | Rootfs in GB (auf zfs1)          |
| `beschreibung`  | nein    | `''`    | Geht in Proxmox- und Netbox-Desc |

**Secrets (Semaphore-Environment):**

| Env-Var                      | Inhalt                                      |
|------------------------------|---------------------------------------------|
| `PROXMOX_API_TOKEN_SECRET`   | UUID des `ansible@pve!deploy`-Tokens        |
| `NETBOX_TOKEN`               | Full Token `nbt_<key>.<secret>`             |
| `SSH_PUBLIC_KEY`             | Public Key für User `alexei` (ed25519)      |

### `init-lxc.yaml` — Hardening & Baseline

Kann standalone laufen (Inventory-Target) oder wird von `deploy-lxc.yaml`
gegen die neu provisionierte Gruppe `newly_deployed` aufgerufen.

Tut:

- Dist-Upgrade + Basis-Pakete (`sudo curl fail2ban unattended-upgrades …`)
- User `alexei` anlegen, passwordless sudo über `/etc/sudoers.d/90-alexei`
- SSH-Key via `authorized_keys` deployen
- **SSH ed25519-only**: alle RSA/ECDSA/DSA-Hostkeys entfernt,
  `HostKeyAlgorithms`, `PubkeyAcceptedAlgorithms`, `CASignatureAlgorithms`
  auf ed25519 beschränkt; moderne KEX/Ciphers/MACs; `DebianBanner no`,
  `MaxAuthTries 3`, `AllowUsers <user>`.
- `fail2ban` mit `sshd`-Jail (aggressive, ignoreip für das eigene Netz)
- `unattended-upgrades` aktivieren

**Env-Vars:** `TARGET_USER` (default `alexei`), `SSH_PUBLIC_KEY`,
`TRUSTED_NETWORKS` (default `192.168.178.0/23`).

### `update-lxc.yaml` — Regular System Update

Dist-Upgrade mit automatischem GPG-Key-Handling und geplanten Reboots
(nur VMs, LXCs werden übersprungen).

## Semaphore-Setup

**Einmalige Einrichtung** (neu hinzuzufügen für Deploy-Feature):

1. **Environment-Objekt** `deploy-secrets` anlegen mit:
   ```
   PROXMOX_API_TOKEN_SECRET = <UUID>
   NETBOX_TOKEN             = nbt_<key>.<secret>
   SSH_PUBLIC_KEY           = ssh-ed25519 AAAA… (bestehendes Env wiederverwendbar)
   TARGET_USER              = alexei
   ```

2. **Task-Template** `Deploy New LXC`:
   - Playbook: `playbooks/deploy-lxc.yaml`
   - Inventory: irgendeins (wird ignoriert, Playbook läuft auf `localhost`)
   - Environment: `deploy-secrets`
   - Survey:
     - `hostname` (String, required)
     - `cores` (String, default `2`)
     - `memory` (String, default `2048`)
     - `disk` (String, default `8`)
     - `beschreibung` (String, optional)

## Requirements

- Ansible 2.14+ (uri-Modul `form-urlencoded`)
- Netbox 4.x mit IP-Range `192.168.178.200-254` (ID 1)
- Proxmox 8.x mit Template `debian-13-standard_13.1-1_amd64.tar.zst` unter `local`
- User `ansible@pve` + Token `deploy` (Rolle: `PVEAdmin` auf `/`)

## License

MIT — siehe [LICENSE](LICENSE).
