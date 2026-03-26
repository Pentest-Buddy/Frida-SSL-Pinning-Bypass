# 🚀 Flutter SSL Pinning Bypass (Frida + System CA + iptables + Burp)

This guide provides a **complete, practical method** to bypass SSL pinning in Flutter applications using:

* System CA certificate installation
* Forced proxy routing (iptables)
* Burp Suite interception
* Optional Frida-based dynamic analysis

---

# 🧠 Key Concept

Flutter apps:

* Use **BoringSSL (native layer)**
* Often **ignore Android proxy settings**
* May implement **certificate pinning internally**

👉 Solution:

* Trust Burp CA at system level
* Force traffic via proxy using iptables
* (Optional) Use Frida for deeper analysis

---

# 🧰 Requirements

* Rooted Android emulator (Recommended: Genymotion)
* Burp Suite installed on host machine
* ADB installed and working
* OpenSSL installed
* Target APK

---

# 📦 Step 0 — Setup Environment

## Start emulator

* Launch Genymotion emulator

## Connect ADB

```bash
adb devices
```

---

# 🔐 Step 1 — Install Burp CA as System Certificate

## 1. Export Burp Certificate

In Burp:

* Proxy → Settings → Import / Export CA certificate
* Export as **DER format (.cer/.der)**

---

## 2. Convert DER → CRT

```bash
openssl x509 -inform DER -in cacert.der -out cacert.crt
```

---

## 3. Generate Certificate Hash

```bash
openssl x509 -inform PEM -subject_hash_old -in cacert.crt
```

Example output:

```
9a5ba575
```

---

## 4. Rename Certificate

```bash
rename cacert.crt 9a5ba575.0
```

---

## 5. Push Certificate to Emulator

```bash
adb push 9a5ba575.0 /sdcard/
```

---

## 6. Move Certificate to System Store

```bash
adb shell
su
mount -o rw,remount /
mv /sdcard/9a5ba575.0 /system/etc/security/cacerts/
```

---

## 7. Set Correct Permissions ⚠️

```bash
chmod 644 /system/etc/security/cacerts/9a5ba575.0
chown root:root /system/etc/security/cacerts/9a5ba575.0
```

---

## 8. Reboot Emulator

```bash
reboot
```

---

# 🌐 Step 2 — Configure Proxy

## 1. Find your system IP

Example:

```
192.168.1.5
```

---

## 2. Set Global Proxy

```bash
adb shell settings put global http_proxy <YOUR_IP>:8080
```

Example:

```bash
adb shell settings put global http_proxy 192.168.1.5:8080
```

---

# 🔥 Step 3 — Force Traffic via iptables (IMPORTANT)

Flutter apps ignore proxy → we force redirect.

```bash
adb shell
su

iptables -t nat -A OUTPUT -p tcp --dport 443 -j DNAT --to-destination <YOUR_IP>:8080
iptables -t nat -A OUTPUT -p tcp --dport 80 -j DNAT --to-destination <YOUR_IP>:8080
```

---

## 🔥 Optional — Capture ALL Traffic

```bash
iptables -t nat -A OUTPUT -p tcp -j DNAT --to-destination <YOUR_IP>:8080
```

---

# ⚙️ Step 4 — Configure Burp Suite

In Burp Suite:

* Proxy → Options:

  * ✔ Enable **Invisible proxying**
  * ✔ Enable **Support non-proxy clients**

---

# 🧪 Step 5 — Test Interception

1. Open target app
2. Perform actions (login/search/API call)
3. Check Burp

✅ You should now see:

* HTTPS requests
* API endpoints
* Headers and payloads

---

# 🧹 Cleanup After Testing

```bash
adb shell
su
iptables -t nat -F
adb shell settings put global http_proxy :0
```

---

# 🧩 Optional — Frida (Advanced Analysis)

## Run Frida

```bash
frida -U -f com.target.app -l script.js --no-pause
```

## Use Frida for:

* SSL bypass (if needed)
* API tracing
* Function hooking

---

# ⚠️ Troubleshooting

## ❌ No traffic in Burp

* Ensure iptables rules applied
* Check Burp listener running on port 8080

## ❌ SSL errors

* Certificate not installed in system store
* Wrong permissions (must be 644)

## ❌ App crashes

* Incorrect certificate format
* Wrong system mount

## ❌ Only browser works, app doesn’t

👉 Expected — Flutter ignores proxy
✔ iptables solves this

---

# 💣 Important Notes

* Flutter uses **native TLS (BoringSSL)**
* Proxy bypass is **intentional behavior**
* iptables interception is **most reliable method**

---

# 🚀 Summary

| Step        | Purpose            |
| ----------- | ------------------ |
| Install CA  | Trust Burp         |
| Set proxy   | Base routing       |
| iptables    | Force interception |
| Burp config | Handle traffic     |

---

# 🧠 Use Cases

* Mobile application security testing
* API reverse engineering
* Dynamic analysis (MobSF / manual)
* Penetration testing

---

# ⚡ Tags

flutter • ssl-pinning • frida • android-security • burp-suite • pentesting

---
