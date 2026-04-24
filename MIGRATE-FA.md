# راهنمای مهاجرت از Dendrite به Conduit

**برای کاربرانی که زنجیر با Dendrite را نصب کرده‌اند**

---

## ⚠️ هشدار مهم

این مهاجرت **تمام داده‌های موجود را پاک می‌کند**:
- ✗ تمام کاربران
- ✗ تمام اتاق‌ها
- ✗ تمام پیام‌ها
- ✗ تمام فایل‌های آپلود شده

**کاربران باید دوباره ثبت‌نام کنند.**

---

## 🚀 مراحل مهاجرت

### مرحله 1: آماده‌سازی

لیست کاربران فعلی را یادداشت کنید (اختیاری):

```bash
cd ~/zanjir
docker ps -a > migration-inventory.txt
```

### مرحله 2: پاک‌سازی کامل

کل سیستم را از بیخ و بن پاک کنید:

```bash
# توقف تمام سرویس‌ها
cd ~/zanjir
docker compose down

# حذف تمام volumeها (داده‌ها)
docker volume rm zanjir-postgres-data \
  zanjir-caddy-data \
  zanjir-caddy-config \
  zanjir-conduit-data \
  zanjir-element-web \
  zanjir-admin-data

# اگر خطا دادند، یکی یکی حذف کنید:
docker volume ls | grep zanjir
docker volume rm <volume-name>

# حذف پوشه پروژه
cd ~
rm -rf ~/zanjir
```

### مرحله 3: نصب مجدد با Conduit

کد جدید را از GitHub بکشید و نصب کنید:

```bash
# کلون مجدد پروژه
git clone https://github.com/MatinSenPai/zanjir.git ~/zanjir
cd ~/zanjir

# نصب (نسخه جدید با Conduit)
sudo bash install.sh
```

**نکته:** در حین نصب:
- اگر می‌خواهید اتصال کاربران قابل پیش‌بینی بماند، همان دامنه یا نام داخلی قبلی را وارد کنید
- حالت `isolated` یا `federated` را صریح انتخاب کنید
- اگر حالت گواهی `private` را انتخاب می‌کنید، مسیر فایل PEM گواهی و کلید جدید را بدهید

### مرحله 4: ثبت‌نام کاربران

کاربران باید از طریق Element Web ثبت‌نام کنند:

1. به آدرس سرور بروید: `https://your-domain.com`
2. روی **"ساخت حساب کاربری"** کلیک کنید
3. نام کاربری و رمز عبور وارد کنید

---

## ✅ بررسی سلامت سیستم

بعد از نصب، این موارد را چک کنید:

```bash
# چک کردن سرویس‌ها
docker ps

# باید این موارد را ببینید:
# - zanjir-conduit (به جای zanjir-dendrite)
# - zanjir-caddy
# - zanjir-admin
# - zanjir-coturn
# - zanjir-element
```

```bash
# چک لاگ Conduit
docker logs zanjir-conduit
```

باید ببینید:
```
INFO conduit::server: Server listening on 0.0.0.0:6167
```

```bash
# تست API
curl http://localhost:6167/_matrix/client/versions
```

باید JSON برگرداند.

---

## 💡 تفاوت‌های کلیدی

| قبل (Dendrite) | بعد (Conduit) |
|----------------|---------------|
| PostgreSQL نیاز بود | دیگر نیاز نیست (RocksDB داخلی) |
| ~200-500MB RAM | ~50MB RAM |
| 2GB VPS لازم | 1GB VPS کافی است |
| `zanjir-dendrite` | `zanjir-conduit` |

---

## ❓ سوالات متداول

**س: آیا می‌توانم داده‌هایم را حفظ کنم؟**  
ج: خیر. Dendrite و Conduit دیتابیس‌های متفاوتی دارند. مهاجرت غیرممکن است.

**س: چقدر طول می‌کشد؟**  
ج: 5-10 دقیقه (بسته به سرعت اینترنت برای دانلود Docker images)

**س: آیا پنل مدیریت کار می‌کند؟**  
ج: سرویس بالا می‌آید و به پشته مبتنی بر Conduit وصل است، اما مدل مجوزدهی آن هنوز ضعیف است و نباید بیش از حد به آن اعتماد کرد.

**س: تماس‌های صوتی/تصویری چطور؟**  
ج: سرور TURN همچنان کار می‌کند. تغییری نیست.

---

## 🆘 نیاز به کمک؟

اگر مشکلی پیش آمد:

1. لاگ سرویس‌ها را چک کنید:
   ```bash
   docker logs zanjir-conduit
   docker logs zanjir-caddy
   ```

2. Issue باز کنید: [GitHub Issues](https://github.com/MatinSenPai/zanjir/issues)

---

**موفق باشید! 🚀**
