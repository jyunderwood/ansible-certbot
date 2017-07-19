# Ansible Certbot

For Ubuntu 16.04.

```yaml
  vars:
    - app_domain: example.com
    - certbot_domain: "{{ app_domain }}"
    - certbot_email: jonsnow@example.com

  roles:
    - certbot
```

Your certs will live at:

```yaml
  - app_cert: /etc/letsencrypt/live/{{ app_domain }}/fullchain.pem
  - app_cert_key: /etc/letsencrypt/live/{{ app_domain }}/privkey.pem
```

Your nginx config for the domain might look like this:

```nginx
server {
  listen 443 ssl;
  server_name {{ app_domain }};

  ssl on;
  ssl_certificate {{ app_cert }};
  ssl_certificate_key {{ app_cert_key }};
  ssl_prefer_server_ciphers on;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  add_header Strict-Transport-Security "max-age=31536000; includeSubdomains";

  include /etc/nginx/snippets/letsencrypt.conf;

  root {{ app_directory }}/public;

  location @app {
    include proxy_params;
    proxy_pass http://127.0.0.1:{{ app_port }};
  }

  try_files $uri $uri/index.html @app;
}

server {
  listen 80;
  server_name {{ app_domain }} www.{{ app_domain }};
  return 301 https://{{ app_domain }}$request_uri;
}
```
