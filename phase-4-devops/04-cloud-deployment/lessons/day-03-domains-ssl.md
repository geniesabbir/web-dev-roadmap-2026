# Day 3: Domains and SSL - Professional Web Presence

## Introduction

A custom domain gives your application a professional identity, while SSL certificates ensure secure communication. Understanding DNS configuration and SSL management is essential for deploying production applications. Today, you'll learn to configure domains, manage DNS records, and secure your applications with HTTPS.

## Learning Objectives

By the end of this lesson, you will be able to:
- Purchase and configure custom domains
- Understand DNS record types and configuration
- Set up SSL/TLS certificates
- Configure domains across different platforms
- Troubleshoot common domain issues

---

## Domain Basics

### Domain Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                      Domain Anatomy                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  https://www.example.com/path/page                              │
│  └─┬──┘ └┬┘ └──┬───┘ └┬┘ └───┬────┘                            │
│    │     │     │      │      │                                   │
│    │     │     │      │      └── Path                           │
│    │     │     │      └── TLD (Top Level Domain)                │
│    │     │     └── Domain Name (Second Level)                   │
│    │     └── Subdomain                                          │
│    └── Protocol (HTTPS = secure)                                │
│                                                                  │
│  Examples:                                                       │
│  • example.com          → Apex/Root domain                      │
│  • www.example.com      → Subdomain                             │
│  • api.example.com      → Subdomain                             │
│  • staging.example.com  → Subdomain                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Domain Registrars

| Registrar | Pros | Pricing |
|-----------|------|---------|
| Namecheap | Affordable, good UI | ~$10/year |
| Google Domains | Clean interface, easy setup | ~$12/year |
| Cloudflare | Free DNS, privacy | ~$10/year |
| GoDaddy | Popular, many TLDs | ~$12/year |
| Porkbun | Cheapest, good service | ~$8/year |

---

## DNS Record Types

### Common Record Types

```
┌─────────────────────────────────────────────────────────────────┐
│                      DNS Record Types                            │
├──────────┬──────────────────────────────────────────────────────┤
│ A        │ Maps domain to IPv4 address                          │
│          │ example.com → 76.76.21.21                            │
├──────────┼──────────────────────────────────────────────────────┤
│ AAAA     │ Maps domain to IPv6 address                          │
│          │ example.com → 2606:4700::6810:1234                   │
├──────────┼──────────────────────────────────────────────────────┤
│ CNAME    │ Alias to another domain                              │
│          │ www.example.com → example.com                        │
├──────────┼──────────────────────────────────────────────────────┤
│ MX       │ Mail server for domain                               │
│          │ example.com → mail.example.com (priority: 10)        │
├──────────┼──────────────────────────────────────────────────────┤
│ TXT      │ Text records (verification, SPF, DKIM)               │
│          │ example.com → "v=spf1 include:_spf.google.com ~all"  │
├──────────┼──────────────────────────────────────────────────────┤
│ NS       │ Nameserver for domain                                │
│          │ example.com → ns1.nameserver.com                     │
├──────────┼──────────────────────────────────────────────────────┤
│ CAA      │ Certificate Authority Authorization                   │
│          │ example.com → "0 issue letsencrypt.org"              │
└──────────┴──────────────────────────────────────────────────────┘
```

### DNS Configuration Examples

```dns
; A Records - Point to IP address
@     A     76.76.21.21           ; apex domain
www   A     76.76.21.21           ; www subdomain

; CNAME Records - Alias to another domain
www      CNAME   example.com.          ; www → apex
api      CNAME   api.railway.app.      ; api subdomain
blog     CNAME   blog.vercel.app.      ; blog subdomain

; MX Records - Email
@     MX    10   mail.example.com.
@     MX    20   mail2.example.com.

; TXT Records - Verification
@     TXT   "v=spf1 include:_spf.google.com ~all"
@     TXT   "google-site-verification=abc123"

; CAA Records - SSL Certificate Authority
@     CAA   0 issue "letsencrypt.org"
@     CAA   0 issue "digicert.com"
```

---

## Platform-Specific Setup

### Vercel Domains

```bash
# Add domain via CLI
vercel domains add example.com

# Via dashboard:
# 1. Project Settings > Domains
# 2. Add Domain
# 3. Configure DNS records
```

```dns
; For apex domain (example.com)
@     A     76.76.21.21

; For subdomain (www.example.com)
www   CNAME cname.vercel-dns.com.

; Or use Vercel nameservers (recommended)
; NS records at registrar:
; ns1.vercel-dns.com
; ns2.vercel-dns.com
```

### Railway Domains

```bash
# Generate Railway domain
# Settings > Networking > Generate Domain

# Custom domain
# Settings > Networking > Custom Domain
```

```dns
; CNAME to Railway
api   CNAME   your-app.up.railway.app.
```

### Cloudflare DNS

```bash
# 1. Add site to Cloudflare
# 2. Update nameservers at registrar
# 3. Configure DNS in Cloudflare dashboard
```

```dns
; Cloudflare settings
; Proxy status: Proxied (orange cloud) for CDN/security
; Proxy status: DNS only (gray cloud) for direct connection

@     A     192.0.2.1      ; Proxied
api   A     192.0.2.2      ; DNS only (for websockets)
```

---

## SSL/TLS Certificates

### How HTTPS Works

```
┌─────────────────────────────────────────────────────────────────┐
│                      TLS Handshake                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Browser                              Server                    │
│     │                                    │                      │
│     │ ─────── Client Hello ──────────► │                      │
│     │         (supported ciphers)        │                      │
│     │                                    │                      │
│     │ ◄────── Server Hello ──────────── │                      │
│     │         (chosen cipher)            │                      │
│     │         (SSL certificate)          │                      │
│     │                                    │                      │
│     │ ─────── Verify Certificate ──────  │                      │
│     │         (check CA, expiry, domain) │                      │
│     │                                    │                      │
│     │ ─────── Key Exchange ──────────►  │                      │
│     │                                    │                      │
│     │ ◄═════ Encrypted Connection ═════► │                      │
│     │                                    │                      │
└─────────────────────────────────────────────────────────────────┘
```

### Certificate Types

| Type | Validation | Use Case |
|------|------------|----------|
| DV (Domain) | Domain ownership | Personal, small business |
| OV (Organization) | Organization verified | Business sites |
| EV (Extended) | Full verification | Banks, e-commerce |
| Wildcard | *.example.com | Multiple subdomains |
| SAN (Multi-domain) | Multiple domains | Multiple sites |

### Let's Encrypt (Free SSL)

```bash
# Using Certbot
sudo apt install certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d example.com -d www.example.com

# Auto-renewal (cron)
0 12 * * * /usr/bin/certbot renew --quiet

# Manual certificate
sudo certbot certonly --standalone -d example.com

# Certificate locations
/etc/letsencrypt/live/example.com/fullchain.pem
/etc/letsencrypt/live/example.com/privkey.pem
```

### Platform SSL (Automatic)

```bash
# Vercel - Automatic SSL for all domains
# Railway - Automatic SSL for all domains
# Cloudflare - Automatic SSL when proxied

# No manual configuration needed!
# SSL certificates are provisioned automatically
# Auto-renewal handled by platform
```

---

## HTTPS Configuration

### Nginx HTTPS Setup

```nginx
# /etc/nginx/sites-available/example.com

# HTTP to HTTPS redirect
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri;
}

# HTTPS server
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name example.com www.example.com;

    # SSL certificates
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # SSL configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;

    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

    # Your application
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Express.js HTTPS Redirect

```javascript
// Force HTTPS in production
app.use((req, res, next) => {
  if (
    process.env.NODE_ENV === 'production' &&
    req.headers['x-forwarded-proto'] !== 'https'
  ) {
    return res.redirect(`https://${req.headers.host}${req.url}`);
  }
  next();
});
```

### Next.js HTTPS Headers

```javascript
// next.config.js
module.exports = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'Strict-Transport-Security',
            value: 'max-age=31536000; includeSubDomains'
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff'
          },
          {
            key: 'X-Frame-Options',
            value: 'DENY'
          }
        ]
      }
    ];
  }
};
```

---

## Multi-Environment Domains

### Environment Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                    Domain Strategy                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Production:                                                     │
│  • example.com            → Main website                        │
│  • www.example.com        → Redirect to apex                    │
│  • api.example.com        → Production API                      │
│                                                                  │
│  Staging:                                                        │
│  • staging.example.com    → Staging website                     │
│  • api-staging.example.com → Staging API                        │
│                                                                  │
│  Development:                                                    │
│  • dev.example.com        → Dev website                         │
│  • api-dev.example.com    → Dev API                             │
│                                                                  │
│  Preview (auto-generated):                                       │
│  • pr-123.example.com     → PR preview                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Wildcard Subdomains

```dns
; Wildcard record for all subdomains
*     CNAME   app.vercel.app.

; Specific overrides
api   CNAME   api.railway.app.
blog  CNAME   blog.vercel.app.
```

---

## Domain Troubleshooting

### Common Issues

```bash
# 1. DNS not propagating
# Check propagation status
dig example.com
nslookup example.com
# Or use: https://dnschecker.org

# 2. SSL certificate errors
# Check certificate
openssl s_client -connect example.com:443 -servername example.com

# 3. Mixed content warnings
# Ensure all resources use HTTPS
# Check browser console for warnings

# 4. WWW vs non-WWW issues
# Redirect one to the other consistently
```

### DNS Propagation

```bash
# Check DNS records
dig example.com A
dig www.example.com CNAME
dig example.com MX
dig example.com TXT

# Check from different locations
dig @8.8.8.8 example.com A     # Google DNS
dig @1.1.1.1 example.com A     # Cloudflare DNS

# Full propagation can take 24-48 hours
# Usually much faster (minutes to hours)
```

### SSL Debugging

```bash
# Check certificate details
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -text

# Check certificate chain
openssl s_client -connect example.com:443 -showcerts

# Test SSL configuration
# Use: https://www.ssllabs.com/ssltest/

# Check certificate expiry
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates
```

---

## Email Configuration

### MX Records for Email

```dns
; Gmail/Google Workspace
@   MX   1   aspmx.l.google.com.
@   MX   5   alt1.aspmx.l.google.com.
@   MX   5   alt2.aspmx.l.google.com.
@   MX   10  alt3.aspmx.l.google.com.
@   MX   10  alt4.aspmx.l.google.com.

; Microsoft 365
@   MX   0   example-com.mail.protection.outlook.com.

; Fastmail
@   MX   10  in1-smtp.messagingengine.com.
@   MX   20  in2-smtp.messagingengine.com.
```

### Email Authentication

```dns
; SPF (Sender Policy Framework)
@   TXT   "v=spf1 include:_spf.google.com ~all"

; DKIM (DomainKeys Identified Mail)
google._domainkey   TXT   "v=DKIM1; k=rsa; p=MIGfMA0..."

; DMARC (Domain-based Message Authentication)
_dmarc   TXT   "v=DMARC1; p=quarantine; rua=mailto:admin@example.com"
```

---

## CDN and Performance

### Cloudflare Setup

```bash
# 1. Add site to Cloudflare
# 2. Update nameservers at registrar
# 3. Configure:
#    - SSL/TLS: Full (strict)
#    - Always Use HTTPS: On
#    - Auto Minify: HTML, CSS, JS
#    - Brotli: On
```

### Cloudflare Page Rules

```
# Cache everything for static assets
URL: example.com/static/*
Setting: Cache Level = Cache Everything

# Bypass cache for API
URL: example.com/api/*
Setting: Cache Level = Bypass

# Always redirect to HTTPS
URL: http://example.com/*
Setting: Always Use HTTPS
```

---

## Practice Exercises

### Exercise 1: Domain Setup

```bash
# 1. Purchase a domain (or use free subdomain)
# 2. Configure DNS for Vercel/Railway
# 3. Add www redirect
# 4. Verify SSL is working
```

### Exercise 2: Multi-Subdomain

```bash
# Set up:
# 1. example.com → Frontend (Vercel)
# 2. api.example.com → Backend (Railway)
# 3. Verify both work with HTTPS
```

### Exercise 3: Email Setup

```dns
# 1. Configure MX records for email provider
# 2. Set up SPF record
# 3. Test email delivery
```

---

## Quick Reference

### DNS Records

| Type | Purpose | Example |
|------|---------|---------|
| A | IPv4 address | `@ A 76.76.21.21` |
| AAAA | IPv6 address | `@ AAAA 2606:...` |
| CNAME | Alias | `www CNAME example.com` |
| MX | Mail server | `@ MX 10 mail.example.com` |
| TXT | Text/verification | `@ TXT "v=spf1..."` |

### Common TTL Values

| TTL | Duration | Use Case |
|-----|----------|----------|
| 300 | 5 minutes | Frequent changes |
| 3600 | 1 hour | Standard |
| 86400 | 24 hours | Stable records |

### SSL Commands

| Command | Purpose |
|---------|---------|
| `certbot --nginx` | Get Let's Encrypt cert |
| `certbot renew` | Renew certificates |
| `openssl s_client` | Test SSL connection |
| `dig domain.com` | Check DNS records |

---

## Key Takeaways

1. **A records** for apex domains, **CNAME** for subdomains
2. **Let's Encrypt** provides free SSL certificates
3. **Platform SSL** is automatic on Vercel/Railway
4. **Always redirect** HTTP to HTTPS
5. **DNS propagation** can take up to 48 hours
6. **Use Cloudflare** for free CDN and security

---

## What's Next?

Tomorrow, we'll explore **AWS Basics** - understanding core AWS services and deploying applications to Amazon's cloud platform.
