# 🚀 Marzban VPS Migration Guide

Complete guide to migrate your **Marzban VPN server** from an old VPS to a new VPS without losing:

- 👤 Users
- 🔑 Reality / TLS keys
- 📦 Subscriptions
- ⚙️ Xray configuration
- 🔒 SSL certificates

---

# 📌 Migration Flow

1. Prepare New VPS
2. Backup Old VPS
3. Transfer Backup
4. Restore on New VPS
5. DNS Switch & Final Cutover

---

# 📋 Requirements

### Recommended VPS Specs

| Resource | Minimum | Recommended |
|----------|----------|--------------|
| CPU | 1 Core | 2 Cores |
| RAM | 1 GB | 2 GB |
| OS | Ubuntu 22.04 LTS | Ubuntu 22.04 LTS |

Requirements:

- Root access
- Public IP
- Docker support

---

# STEP 1 — Prepare the NEW VPS

## 1. Connect to New VPS

```bash
ssh root@NEW_VPS_IP
```

## 2. Update Server

```bash
apt update && apt upgrade -y
```

## 3. Install Required Packages

```bash
apt install -y curl wget git sudo tar
```

## 4. Install Docker

```bash
curl -fsSL https://get.docker.com | sh
```

## 5. Install Docker Compose

Check:

```bash
docker compose version
```

If not installed:

```bash
apt install docker-compose-plugin -y
```

## 6. Enable Docker

```bash
systemctl enable docker
systemctl start docker
```

## 7. Check Docker

```bash
docker ps
```

---

## ⚠️ Important

Do **NOT**:

- Install fresh Marzban
- Create new users
- Generate Reality keys

Because we will restore old data.

---

## Open Firewall Ports

```bash
ufw allow 22
ufw allow 80
ufw allow 443
ufw allow 8000
ufw enable
```

Get server IP:

```bash
curl ifconfig.me
```

---

# STEP 2 — Backup OLD VPS

## Connect to OLD VPS

```bash
ssh root@OLD_VPS_IP
```

## Go to Marzban Directory

```bash
cd /opt/marzban
ls
```

Expected:

```txt
docker-compose.yml
.env
```

## Stop Containers

```bash
docker compose down
docker ps
```

## Create Backup Folder

```bash
mkdir -p /root/marzban-backup
```

## Backup Marzban Files

```bash
cp -r /opt/marzban /root/marzban-backup/
```

## Backup Data

```bash
ls /var/lib/marzban
cp -r /var/lib/marzban /root/marzban-backup/
```

## Backup SSL Certificates

```bash
cp -r /etc/letsencrypt /root/marzban-backup/
```

## Create Backup Archive

```bash
cd /root
tar -czvf marzban-backup.tar.gz marzban-backup
```

## Verify Backup

```bash
ls -lh /root/marzban-backup.tar.gz
```

## Restart Old VPS

```bash
cd /opt/marzban
docker compose up -d
docker ps
```

---

# STEP 3 — Transfer Backup to NEW VPS

## Verify Backup Exists

```bash
ls -lh /root/marzban-backup.tar.gz
```

## Transfer Backup

Run on OLD VPS:

```bash
scp /root/marzban-backup.tar.gz root@NEW_VPS_IP:/root/
```

Example:

```bash
scp /root/marzban-backup.tar.gz root@192.168.1.100:/root/
```

## Verify on NEW VPS

```bash
ssh root@NEW_VPS_IP
ls -lh /root/marzban-backup.tar.gz
```

## Optional SHA256 Check

OLD VPS:

```bash
sha256sum /root/marzban-backup.tar.gz
```

NEW VPS:

```bash
sha256sum /root/marzban-backup.tar.gz
```

---

# STEP 4 — Restore Backup on NEW VPS

## Stop Existing Containers

```bash
cd /opt/marzban
docker compose down
```

## Create Restore Folder

```bash
mkdir -p /root/marzban-restore
```

## Extract Backup

```bash
tar -xzvf /root/marzban-backup.tar.gz -C /root/marzban-restore
```

## Restore Files

```bash
cp -r /root/marzban-restore/marzban-backup/marzban /opt/
```

## Restore Data

```bash
cp -r /root/marzban-restore/marzban-backup/marzban-backup/* /var/lib/marzban
```

## Restore SSL Certificates

```bash
cp -r /root/marzban-restore/marzban-backup/etc/letsencrypt /etc/
```

## Fix Permissions

```bash
chown -R root:root /opt/marzban
```

## Start Marzban

```bash
cd /opt/marzban
docker compose up -d
```

## Check Containers

```bash
docker ps
```

## Check Logs

Marzban:

```bash
docker logs marzban
```

Xray:

```bash
docker logs marzban-xray
```

---

# STEP 5 — DNS Switch & Final Cutover

## Get New VPS IP

```bash
curl ifconfig.me
```

## Update DNS

Example:

```txt
vpn.yourdomain.com → NEW_VPS_IP
```

Recommended TTL:

```txt
300 seconds
```

## Check DNS

```bash
nslookup vpn.yourdomain.com
```

## Verify Services

```bash
docker ps
docker logs marzban
docker logs marzban-xray
```

## Test Subscription

```txt
https://vpn.yourdomain.com/sub
```

Verify:

- ✅ Login works
- ✅ Users visible
- ✅ VPN connects
- ✅ Subscription updates

---

# ⚠️ Final Recommendation

Keep the **OLD VPS** active for **24–72 hours** before deleting it.

After confirming everything works:

```bash
docker compose down
```

---

# 📝 License

Free to use and modify.
