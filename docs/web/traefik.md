---
icon: lucide/cpu
hide_title: true
---

``` yaml title="docker-compose.yaml"
services:

  traefik:
    image: traefik
    container_name: traefik
    ports:
      - 80:80
      - 443:443
    labels:
      - traefik.enable=true
      - traefik.http.routers.dashboard.entrypoints=https
      - traefik.http.routers.dashboard.rule=Host(`proxy.vmtlw.ru`)
      - traefik.http.routers.dashboard.service=api@internal
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /etc/localtime:/etc/localtime:ro
      - ./letsencrypt:/letsencrypt
      - ./traefik.yml:/traefik.yml:ro
      - ./security.yml:/security.yml:ro
      - ./log:/log
    networks:
      - proxy
    restart: unless-stopped

networks:
  proxy:
    name: proxy
```




``` yaml title="traefik.yaml"
api:
  dashboard: true
  insecure: false

certificatesresolvers:
  letsencrypt:
    acme:
      email: "admin@mail.ru"
      storage: "/letsencrypt/acme.json"
      tlschallenge: true

entrypoints:
  http:
    address: ":80"
    http:
      redirections:
        entrypoint:
          scheme: "https"
          to: "https"
  https:
    address: ":443"
    http:
      tls:
        certResolver: "letsencrypt"
tls:
  options:
    default:
      minVersion: VersionTLS12
      cipherSuites:
        - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
        - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
        - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
        - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
        - TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305
        - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305

providers:
  docker:
   endpoint: "unix:///var/run/docker.sock"
   exposedByDefault: false
   network: proxy

http:
  routers:
    dashboard:
      entrypoints: "https"
      rule: "Host(`proxy.vmtlw.ru`)"
      service: "api@internal"
  middlewares:
    locl-ipwhitelist:
      ipWhiteList:
        sourceRange:
          - 127.0.0.1/32 # localhost
          - 10.0.0.0/8 # private class A
          - 172.16.0.0/12 # private class B
    security-headers:
      headers:
        customResponseHeaders: # field names are case-insensitive
          X-Robots-Tag: "none,noarchive,nosnippet,notranslate,noimageindex"
          Server: "" # prevent version disclosure
          X-Powered-By: "" # prevent version disclosure
          X-Forwarded-Proto: "https"
        sslProxyHeaders:
          X-Forwarded-Proto: "https"
        hostsProxyHeaders:
          - "X-Forwarded-Host"
        customRequestHeaders:
          X-Forwarded-Proto: "https"
        contentTypeNosniff: true # X-Content-Type-Options
        customFrameOptionsValue: "SAMEORIGIN" # X-Frame-Options
        browserXssFilter: false # X-XSS-Protection; deprecated
        referrerPolicy: "strict-origin-when-cross-origin" # Referrer-Policy
        forceSTSHeader: true # HTTP-Strict-Transport-Security (HSTS)
        stsIncludeSubdomains: true # HTTP-Strict-Transport-Security (HSTS)
        stsSeconds: 63072000 # HTTP-Strict-Transport-Security (HSTS)
        stsPreload: true # HTTP-Strict-Transport-Security (HSTS)
        #contentSecurityPolicy: "block-all-mixed-content" # Content-Security-Policy (CSP)


log:
  format: json
  filePath: "/log/traefik.log"


accessLog:
  filePath: "/log/access.log"
  format: json
  filters:
    statusCodes:
      - "200"
      - "300-302"
    retryAttempts: true
    minDuration: "10000ms"
```
