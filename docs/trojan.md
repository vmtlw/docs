---
title: 3x-ui
icon: lucide/cpu
---
```
server {
  listen 80;
  server_name domain.ru;
  rewrite ^ https://$server_name$request_uri? permanent;
}

server {
  server_name domain.ru;
  ssl_certificate /etc/nginx/ssl/fullchain.pem;
  ssl_certificate_key /etc/nginx/ssl/privkey.pem;
  location / {
    proxy_pass https://127.0.0.1:2054;
  }

  # Тут уже не надо ничего менять
  listen 443 ssl http2;
  keepalive_timeout 75 75;
  ssl_session_timeout 5m;
  add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
  proxy_buffer_size 12k;
  proxy_set_header Host $host;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Host $host;
  proxy_set_header X-Forwarded-Server $host;
  proxy_set_header X-Forwarded-Proto https;
}
```

```yaml
services:
  3xui:
    image: ghcr.io/mhsanaei/3x-ui:latest
    container_name: vpn
    hostname: domain.ru
    volumes:
      - $PWD/db/:/etc/x-ui/
      - /etc/nginx/ssl/:/root/cert/
    environment:
      XRAY_VMESS_AEAD_FORCED: "false"
      XUI_ENABLE_FAIL2BAN: "false"
    tty: true
    network_mode: host
    restart: unless-stopped
```


```bash
ctr run -d  --mount type=bind,src=/srv/docker/3x-ui/db,dst=/etc/x-ui,options=rbind:rw --mount type=bind,src=/etc/nginx/ssl,dst=/root/cert,options=rbind:rw --net-host docker.io/alireza7/x-ui:latest vpn
```

```bash
[Unit]
Description=3x-ui container
After=network.target containerd.service
Requires=containerd.service

[Service]
Restart=unless-stopped
ExecStartPre=/bin/sh -c '/usr/bin/ctr tasks kill vpn || true; /usr/bin/ctr container rm vpn || true'
ExecStartPre=/bin/sh -c 'ctr images ls | grep -q "docker.io/alireza7/x-ui:latest" || ctr images pull docker.io/alireza7/x-ui:latest'
ExecStart=/usr/bin/ctr run --rm --mount type=bind,src=/srv/docker/3x-ui/db,dst=/etc/x-ui,options=rbind:rw --mount type=bind,src=/etc/nginx/ssl,dst=/root/cert,options=rbind:rw --net-host docker.io/alireza7/x-ui:latest vpn
ExecStop=/usr/bin/ctr tasks kill vpn
KillMode=process
TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
```
