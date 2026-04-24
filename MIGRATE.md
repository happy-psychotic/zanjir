# Migration Guide: Dendrite → Conduit

**For existing users who have Zanjir with Dendrite installed**

---

## ⚠️ Important Warning

This migration **will delete all existing data**:
- ✗ All users
- ✗ All rooms
- ✗ All messages
- ✗ All uploaded files

**Users will need to re-register.**

---

## 🚀 Migration Steps

### Step 1: Preparation

Optionally note your current users:

```bash
cd ~/zanjir
docker ps -a > migration-inventory.txt
```

### Step 2: Complete Wipe

Remove everything from scratch:

```bash
# Stop all services
cd ~/zanjir
docker compose down

# Remove ALL volumes (data)
docker volume rm zanjir-postgres-data \
  zanjir-caddy-data \
  zanjir-caddy-config \
  zanjir-conduit-data \
  zanjir-element-web \
  zanjir-admin-data

# If errors, remove one by one:
docker volume ls | grep zanjir
docker volume rm <volume-name>

# Delete project folder
cd ~
rm -rf ~/zanjir
```

### Step 3: Fresh Install with Conduit

Pull new code and install:

```bash
# Clone project again
git clone https://github.com/MatinSenPai/zanjir.git ~/zanjir
cd ~/zanjir

# Install (new version with Conduit)
sudo bash install.sh
```

**Note:** During installation:
- Use the same domain or internal server name if you want clients to reconnect predictably
- Choose `isolated` or `federated` mode explicitly
- If you choose `private` certificate mode, provide PEM paths for the replacement server certificate and key

### Step 4: User Registration

Users must register via Element Web:

1. Go to: `https://your-domain.com`
2. Click **"Create Account"**
3. Enter username and password

---

## ✅ Health Check

After installation, verify:

```bash
# Check services
docker ps

# You should see:
# - zanjir-conduit (instead of zanjir-dendrite)
# - zanjir-caddy
# - zanjir-admin
# - zanjir-coturn
# - zanjir-element
```

```bash
# Check Conduit logs
docker logs zanjir-conduit
```

Should show:
```
INFO conduit::server: Server listening on 0.0.0.0:6167
```

```bash
# Test API
curl http://localhost:6167/_matrix/client/versions
```

Should return JSON.

---

## 💡 Key Differences

| Before (Dendrite) | After (Conduit) |
|-------------------|-----------------|
| Needed PostgreSQL | No longer needed (embedded RocksDB) |
| ~200-500MB RAM | ~50MB RAM |
| 2GB VPS required | 1GB VPS sufficient |
| `zanjir-dendrite` | `zanjir-conduit` |

---

## ❓ FAQ

**Q: Can I keep my data?**  
A: No. Dendrite and Conduit use different databases. Migration is not possible.

**Q: How long does it take?**  
A: 5-10 minutes (depending on internet speed for Docker images)

**Q: Does admin panel work?**  
A: It starts and is wired to the Conduit-based stack, but its authorization model is still weak and should not be overtrusted.

**Q: What about voice/video calls?**  
A: TURN server still works. No changes.

---

## 🆘 Need Help?

If you encounter issues:

1. Check service logs:
   ```bash
   docker logs zanjir-conduit
   docker logs zanjir-caddy
   ```

2. Open an issue: [GitHub Issues](https://github.com/MatinSenPai/zanjir/issues)

---

**Good luck! 🚀**
