# KB-002: Resolving GitHub Pages Custom Domain TLS Provisioning Error via Cloudflare DNS

## 📌 Incident Summary
- **Target Domain:** `sothatthoeung.com` & `www.sothatthoeung.com`
- **Hosting Platform:** GitHub Pages (Static Site Engine)
- **DNS Provider:** Cloudflare
- **Symptoms:** GitHub Pages displayed `DNS Check in Progress` / `NotServedByPagesError` and prevented enabling `Enforce HTTPS`.

---

## 🔍 Root Cause Analysis (RCA)
1. **Cloudflare Proxy Masking:** The A records pointing to GitHub Pages IP addresses (`185.199.108.153` to `185.199.111.153`) were set to **Proxied (Orange Cloud)**. This intercepted incoming challenge requests from GitHub's automated certificate authority (Let's Encrypt / DigiCert).
2. **Missing/Mismatched Subdomain Routing:** The `www` CNAME record failed hostname resolution due to proxy routing conflicts before full domain ownership verification.

---

## 🛠️ Resolution Steps

### Step 1: Disable Cloudflare Edge Proxy (DNS Only)
In the Cloudflare DNS dashboard, change the proxy status of apex and sub-records from **Proxied** to **DNS only (Grey Cloud)**:
- `A` record `@` -> `185.199.108.153` (DNS only)
- `A` record `@` -> `185.199.109.153` (DNS only)
- `A` record `@` -> `185.199.110.153` (DNS only)
- `A` record `@` -> `185.199.111.153` (DNS only)
- `CNAME` record `www` -> `<username>.github.io` (DNS only)

### Step 2: Trigger GitHub Pages Verification
1. Navigate to **Repository Settings** > **Pages**.
2. Re-trigger the custom domain check by clicking **Check again**.
3. Await ACME challenge completion (`DNS check successful` and `Certificate Active`).

### Step 3: Enforce HTTPS & Clear Edge Cache
1. Check the **Enforce HTTPS** checkbox once certificate generation succeeds.
2. In the local development environment or client browser, perform an empty-cache reload (`Ctrl + Shift + R`) to avoid cached redirect loops.

---

## ✅ Verification
- Domain resolves directly via public resolvers (`dig +short sothatthoeung.com` returns GitHub Pages Anycast IPs).
- SSL/TLS certificate issued correctly and auto-renews via GitHub infrastructure.