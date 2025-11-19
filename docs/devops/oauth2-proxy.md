---
title: Oauth2-proxy на примере GitLab:
---
# Деплой oauth2-proxy в Kubernetes с авторизацией через GitLab

В этой статье разберём полный процесс развёртывания **oauth2-proxy** в Kubernetes с использованием OAuth2 через GitLab и интеграцией с ingress-nginx.

---

## Установка Helm-чарта

Используем официальный Helm-чарт:

```bash
helm repo add oauth2-proxy https://oauth2-proxy.github.io/manifests
helm repo update

helm install oauth2-proxy oauth2-proxy/oauth2-proxy \
  --namespace oauth2-proxy \
  --create-namespace \
  --version 10.1.5 \
  -f values.yaml
```

---

## Подготовка GitLab OAuth приложения

В GitLab необходимо создать OAuth-приложение:

1. Перейдите в **User Settings → Applications**
2. Укажите:

   * **Name**: `oauth2-proxy`
   * **Redirect URI**:

     ```
     https://oauth2-proxy.domain.com/oauth2/callback
     ```
   * Scopes:

     * `read_user`
     * `openid`
     * `email`
     * `prifile`

После создания вы получите:

* `Application ID` → это `client_id`
* `Secret` → это `client_secret`

---

## Kubernetes Secret

Создаём секрет с параметрами:

```bash
kubectl create secret generic oauth2-proxy-secret \
  --from-literal=client-id=YOUR_CLIENT_ID \
  --from-literal=client-secret=YOUR_CLIENT_SECRET \
  --from-literal=cookie-secret=$(openssl rand -base64 32) \
  -n oauth2-proxy
```

---

## values.yaml

```yaml
ingress:
  enabled: true
  annotations:
    nginx.ingress.kubernetes.io/proxy-buffer-size: "8k"
    nginx.ingress.kubernetes.io/proxy-buffers-number: "4"
    cert-manager.io/cluster-issuer: letsencrypt-prod
  className: "nginx"
  hosts:
     - oauth2-proxy.domain.com
  tls:
    - secretName: ingress-tls
      hosts:
        - oauth2-proxy.domain.com

config:
  existingSecret: oauth2-proxy-secret

extraArgs:
  provider: "gitlab"
  redirect-url: "https://oauth2-proxy.domain.com/oauth2/callback"
  set-xauthrequest: true
  upstream: "static://200"
  reverse-proxy: "true"
  skip-provider-button: "true"
  oidc-issuer-url: "https://gitlab.domain.com"
  email-domain: "*"
  cookie-domain: ".domain.com"
  cookie-secure: "true"
  cookie-samesite: "lax"
  standard-logging: true
  whitelist-domain: ".domain.com"
  pass-access-token: "true"
  pass-authorization-header: "true"
  set-authorization-header: "true"
  pass-user-headers: "true"
  
```

---

## Настройка ingress-nginx для защищаемого сервиса

Чтобы защитить любой сервис через oauth2-proxy, нужно правильно настроить ingress.

 Аннотации должны быть **такими**:

```yaml
annotations:
  cert-manager.io/cluster-issuer: "letsencrypt-prod"
  nginx.ingress.kubernetes.io/auth-url: "https://oauth2-proxy.domain.com/oauth2/auth"
  nginx.ingress.kubernetes.io/auth-signin: "https://oauth2-proxy.domain.com/oauth2/start?rd=$scheme://$host$escaped_request_uri"
  nginx.ingress.kubernetes.io/auth-response-headers: "X-Auth-Request-User,X-Auth-Request-Email,X-Auth-Request-Access-Token"
```

---

## Как это работает

1. Пользователь заходит на защищённый сервис
2. ingress-nginx делает запрос в:

   ```
   /oauth2/auth
   ```
3. Если пользователь не авторизован → редирект на GitLab
4. После логина GitLab возвращает пользователя в:

   ```
   /oauth2/callback
   ```
5. oauth2-proxy устанавливает cookie и проксирует запрос
6. ingress получает заголовки:

   * `X-Auth-Request-User`
   * `X-Auth-Request-Email`
   * `X-Auth-Request-Access-Token`

---

## Проверка

После деплоя:

```bash
kubectl get pods -n oauth2-proxy
kubectl get ingress -n oauth2-proxy
```

Откройте:

```
https://oauth2-proxy.domain.com
```

И затем защищённый сервис — должен быть редирект на GitLab.


## Итог

Схема:

```
User → ingress-nginx → oauth2-proxy → GitLab → oauth2-proxy → Service
```

Такой подход:

* централизует авторизацию
* работает с любыми сервисами
* легко масштабируется

