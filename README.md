# TAGTHAi Setup Wireguard Peer

A GitHub Composite Action that dynamically provisions a WireGuard peer using an API, connects to the VPN if needed, schedules automatic deletion, and safely cleans up.

Repo: [tagthai-actions/setup-wireguard-peer](https://github.com/tagthai-actions/setup-wireguard-peer)

---

## 🚀 Features

- ✅ Detects if running on **GCP VM** and skips VPN setup if so
- 🔐 Creates WireGuard peer via REST API
- 📥 Downloads `.conf` and connects using `wg-quick`
- ⏳ Automatically schedules deletion via API
- 🧹 Cleans up VPN connection reliably at job end

---

## 📥 Inputs

| Name                   | Required | Description                                     |
| ---------------------- | -------- | ----------------------------------------------- |
| `api_url`              | ✅       | Base URL of your WireGuard Dashboard            |
| `api_key`              | ✅       | API key for accessing the dashboard             |
| `config_name`          | ❌       | Peer name (default: `gha-${{ github.run_id }}`) |
| `delete_after_minutes` | ❌       | Time to auto-delete peer (default: `20`)        |

---

## 📤 Outputs

| Name         | Description                                       |
| ------------ | ------------------------------------------------- |
| `public_key` | Public key of the created peer                    |
| `peer_id`    | Internal ID of the peer                           |
| `wg_conf`    | WireGuard config file content (`[Interface] ...`) |

---

## 🧠 Behavior Notes

- On **GCP**: action will detect via metadata server and **skip all VPN-related steps**
- On **other runners**: VPN will be established and cleaned up as normal
- Cleanup uses `if: always()` to ensure VPN is down even if a previous step fails
- ⚠️ On **self-hosted runners**, you must configure `sudo` to allow passwordless execution for `wg-quick` commands.  
  For example, add this to your sudoers file (use `sudo visudo`):

---

## 🛠 Requirements

- Runner must allow installation of `wireguard-tools` (via `apt`)
- `jq`, `curl`, `uuidgen`, and `sudo` should be available

---

## 🔐 Security

- Config and keys are handled in-memory and masked from logs
- `.conf` is cleaned up automatically in the cleanup step

---

## 🧪 Example Full Workflow

```yaml
name: Deploy with WireGuard VPN

on:
  workflow_dispatch:

jobs:
  deploy:
    runs-on: self-hosted

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup VPN via WireGuard
        uses: tagthai-actions/setup-wireguard-peer@v1
        with:
          api_url: ${{ secrets.WG_API_URL }}
          api_key: ${{ secrets.WG_API_KEY }}

      - name: Access internal GKE service
        run: |
          gcloud container clusters get-credentials my-cluster \
            --region asia-southeast1 --project my-project

          kubectl get pods -n default
```
