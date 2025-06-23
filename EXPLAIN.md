# Traefik Configuration (`traefik.yml`)

Konfigurasi ini digunakan untuk setup Traefik sebagai reverse proxy dengan dukungan HTTPS (Let's Encrypt), middleware keamanan, dan integrasi Docker.

---

## Global Settings

```yaml
global:
  checkNewVersion: false
  sendAnonymousUsage: false
```

- **checkNewVersion**: Traefik gak akan cek update otomatis.
- **sendAnonymousUsage**: Nonaktif, jadi gak kirim data penggunaan ke Traefik dev.

---

## API & Dashboard

```yaml
api:
  dashboard: true
  insecure: false
```

- Dashboard Traefik aktif (`true`), tapi aman (gak bisa diakses sembarangan karena `insecure: false`).

---

## EntryPoints (Akses Port)

```yaml
entryPoints:
  web:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https
          permanent: true
  websecure:
    address: ":443"
    http:
      tls:
        options: default@file
```

- **web (port 80)**: Semua request HTTP otomatis redirect ke HTTPS (`websecure`).
- **websecure (port 443)**: Handle HTTPS dengan TLS config default.

---

## Providers (Sumber Konfigurasi)

```yaml
providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
    network: wp_network
  file:
    filename: /traefik.yml
    watch: true
```

- **Docker provider**:

  - Endpoint: akses ke Docker socket langsung.
  - `exposedByDefault: false`: Container gak langsung terbuka ke publik.
  - `network: wp_network`: Traefik cuma handle container di network ini.

- **File provider**:

  - Baca config dari file ini sendiri (`/traefik.yml`) dan auto-reload saat berubah.

---

## Middleware

```yaml
http:
  middlewares:
    secure-headers: ...
    dashboard-auth: ...
    phpmyadmin-stripprefix: ...
    redirect-phpmyadmin: ...
```

- **secure-headers**: Tambah security headers ke respons HTTP (anti-XSS, anti-clickjacking, dll).
- **dashboard-auth**: Basic auth buat proteksi dashboard.
- **phpmyadmin-stripprefix**: Ngilangin `/phpmyadmin` dari path sebelum diterusin ke container.
- **redirect-phpmyadmin**: Auto-redirect dari `/phpmyadmin` ke `/phpmyadmin/` (biar konsisten).

---

## TLS / HTTPS

```yaml
tls:
  options:
    default:
      minVersion: "VersionTLS12"
      cipherSuites:
        - ...
```

- TLS minimal versi 1.2.
- Cipher suites aman dan modern: AES dan ChaCha20.

---

## Let's Encrypt (Cert Resolver)

```yaml
certificatesResolvers:
  letsencrypt:
    acme:
      email: email@gmail.com
      storage: acme.json
      httpChallenge:
        entryPoint: web
```

- Gunain ACME (Let's Encrypt) buat auto-issue sertifikat HTTPS.
- Sertifikat disimpan di `acme.json`.
- Validasi via HTTP challenge di port 80.

---

## Logging

```yaml
log:
  level: INFO
accessLog: {}
```

- **log.level: INFO**: Info-level logging (cukup detail buat debugging).
- **accessLog**: Nyala, jadi semua request bakal ke-log.

---
