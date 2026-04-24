# Semaphore-Setup für `deploy-lxc.yaml`

Einmalige Einrichtung in der Semaphore-UI (http://192.168.178.93:3000).

## 1. Environment-Objekt

**Menü:** Project → Environment → New Environment

- **Name:** `deploy-secrets`
- **Environment (JSON, leer lassen oder `{}`)**
- **Environment Variables** (Extra Variables):
  ```
  TARGET_USER              = alexei
  ```
- **Secrets** (verschlüsselte Environment-Variablen):
  ```
  PROXMOX_API_TOKEN_SECRET = <UUID aus Proxmox-Token-Create>
  NETBOX_TOKEN             = nbt_<key>.<secret>
  SSH_PUBLIC_KEY           = ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA...
  ```

## 2. Task-Template

**Menü:** Project → Task Templates → New Template

- **Name:** `Deploy New LXC`
- **Playbook Filename:** `playbooks/deploy-lxc.yaml`
- **Inventory:** irgendeins (z. B. `localhost` — Playbook läuft auf `localhost`, Inventory-Inhalt wird ignoriert)
- **Repository:** `ansible-playbooks` (das bestehende)
- **Environment:** `deploy-secrets`
- **Survey Variables** (Reihenfolge matters):

| Name           | Title / Description                           | Type   | Required | Default  |
|----------------|-----------------------------------------------|--------|----------|----------|
| `hostname`     | Hostname des neuen LXC                        | String | ✅       | —        |
| `cores`        | CPU-Cores                                     | String | ❌       | `2`      |
| `memory`       | RAM in MB                                     | String | ❌       | `2048`   |
| `disk`         | Rootfs in GB (zfs1)                           | String | ❌       | `8`      |
| `beschreibung` | Zusätzliche Beschreibung (Proxmox + Netbox)   | String | ❌       | `''`     |

Survey-JSON, wie es intern bei Semaphore gespeichert wird:

```json
[
  {"name":"hostname","title":"Hostname","description":"Hostname des neuen LXC (a-z0-9-)","type":"","required":true},
  {"name":"cores","title":"Cores","description":"CPU-Cores","type":"","required":false,"default_value":"2"},
  {"name":"memory","title":"Memory (MB)","description":"RAM in MB","type":"","required":false,"default_value":"2048"},
  {"name":"disk","title":"Disk (GB)","description":"Rootfs in GB","type":"","required":false,"default_value":"8"},
  {"name":"beschreibung","title":"Beschreibung","description":"Zusätzliche Beschreibung","type":"","required":false,"default_value":""}
]
```

## 3. Run

Task ausführen → Survey-Dialog öffnet sich → Hostname eingeben → Run.
Nach ~60–120s ist der LXC deployed, gehärtet und in Netbox dokumentiert.

## Troubleshooting

- **"Invalid v2 token"** → Netbox-Token abgelaufen, neuen via
  `POST /api/users/tokens/provision/` holen und im Environment aktualisieren.
- **"pct: Template not found"** → Template-Name/Version in
  `deploy-lxc.yaml:proxmox_template` prüfen (`pveam list local`).
- **SSH-Timeout** → Bridge oder Gateway-Problem, prüfen via `pct enter <vmid>`.
