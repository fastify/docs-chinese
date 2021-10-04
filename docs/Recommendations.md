<h1 align="center">Fastify</h1>

## 推荐方案

本文涵盖了使用 Fastify 的推荐方案及最佳实践。

* [使用反向代理](#reverseproxy)
* [Kubernetes](#kubernetes)

## 使用反向代理
<a id="reverseproxy"></a>

Node.js 作为各框架中的先行者，在标准库中提供了易用的 web 服务器。在它现世前，PHP、Python 等语言的使用者，要么需要一个支持该语言的 web 服务器，要么需要一个搭配该语言的 [CGI 网关](cgi)。而 Node.js 能让用户专注于 _直接_ 处理 HTTP 请求的应用本身，这样一来，新的诱惑点变成了处理多个域名的请求、监听多端口 (如 HTTP _和_ HTTPS)，接着将应用直接暴露于 Internet 来处理请求。

Fastify 团队**强烈**地认为上述做法是一种反面模式，是非常不理想的实践：

1. 它分离了程序的关注点，增加了不必要的复杂度。
2. 它限制了程序的[水平拓展](scale-horiz)。

请看《[生产环境可用的 Node.js 为何还需要反向代理？][why-use]》一文，这里有更深入的讨论。

考虑以下具体案例：

1. 应用需要多个实例来处理负载。
1. 应用需要 TLS 终端 (TLS termination)。
1. 应用需要将 HTTP 请求转发至 HTTPS。
1. 应用需要处理多域名。
1. 应用需要处理静态资源，例如 jpeg 文件。

反向代理的解决方案有很多种，例如 AWS 与 GCP，具体根据环境来择用。对于上述的案例，我们可以使用 [HAProxy][haproxy] 或 [Nginx][nginx]。

### HAProxy

```conf
# global 定义了 HAProxy 实例的基础配置。
global
  log /dev/log syslog
  maxconn 4096
  chroot /var/lib/haproxy
  user haproxy
  group haproxy

  # 设置 TLS 基础配置。
  tune.ssl.default-dh-param 2048
  ssl-default-bind-options no-sslv3 no-tlsv10 no-tlsv11
  ssl-default-bind-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:RSA+AESGCM:RSA+AES:!aNULL:!MD5:!DSS
  ssl-default-server-options no-sslv3 no-tlsv10 no-tlsv11
  ssl-default-server-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:RSA+AESGCM:RSA+AES:!aNULL:!MD5:!DSS

# defaults 定义了接下来每个子段落的默认配置，直到出现另一段 defaults。
defaults
  log   global
  mode  http
  option        httplog
  option        dontlognull
  retries       3
  option redispatch
  # 下面的选项使 haproxy 关闭与后端的连接，而不是保持连接。
  # 这么做能减轻 Node 进程发生预期外的连接重置错误。
  option http-server-close
  maxconn       2000
  timeout connect 5000
  timeout client 50000
  timeout server 50000

  # 压缩特定类型的内容。
  compression algo gzip
  compression type text/html text/plain text/css application/javascript

# frontend 定义一个公开的监听器，即客户端所关注的“http 服务器”。
frontend proxy
  # 这里的 IP 地址为服务器的_公开_ IP 地址。
  # 在这个例子里，我们使用一个私有的地址。
  bind 10.0.0.10:80
  # 将所有非 TLS 的请求转发至 HTTPS 端口的相同 URL。
  redirect scheme https code 308 if !{ ssl_fc }
  # 从技术角度讲，这里的 use_backend 指令没有用处，
  # 因为我们已经将所有到达该监听器的请求转发至 HTTPS 了。
  # 在此提到该指令仅仅是为了示例的完整性。
  use_backend default-server

# 这段 frontend 定义了主要的 TLS 监听器。
# 在此我们定义 TLS 证书，以及如何引流来访的请求。
frontend proxy-ssl
  # 本例的 `/etc/haproxy/certs` 文件夹保存了以域名命名的 PEM 格式证书。
  # 当 HAProxy 启动时，它会加载该文件夹中的证书，
  # 并使用服务器名称指示协议 (SNI) 将证书应用于对应的连接。
  bind 10.0.0.10:443 ssl crt /etc/haproxy/certs

  # 定义静态资源处理规则。
  # 任何包括 `/static` 路径 (如 `https://one.example.com/static/foo.jpeg`) 的请求，
  # 将被重定向至静态资源服务器。
  acl is_static path -i -m beg /static
  use_backend static-backend if is_static

  # 根据请求的域名，转发至对应的 Node.js 服务器。
  # `acl` 开头这行用于匹配主机名，并定义了一个布尔值用于判断匹配与否。
  # `use_backend` 这行则在该布尔值为真时进行转发。
  acl example1 hdr_sub(Host) one.example.com
  use_backend example1-backend if example1

  acl example2 hdr_sub(Host) two.example.com
  use_backend example2-backend if example2

  # 最后，我们提供一个后备的重定向，以防上述规则均不适用。
  default_backend default-server

# backend 告诉 HAProxy 去哪里获取请求所需的信息。
# 在此我们定义 Node.js 应用及静态资源等其他服务器的地址。
backend default-server
  # 在这个例子中，我们将未匹配域名的请求默认转发到一个能处理所有请求的后端。
  # 值得注意的是，后端服务器不需要处理 TLS 请求。
  # 这被称为“TLS 终端 (TLS termination)”：TLS 连接在反向代理中即得到处理。
  # 代理至有 TLS 请求处理能力的后端也是可行的，但这不在本示例的内容范围之内。
  server server1 10.10.10.2:80

# 通过轮询调度，将 `https://one.example.com` 的请求代理至三个后端服务器。
backend example1-backend
  server example1-1 10.10.11.2:80
  server example1-2 10.10.11.2:80
  server example2-2 10.10.11.3:80

# 处理 `https://two.example.com` 的请求。
backend example2-backend
  server example2-1 10.10.12.2:80
  server example2-2 10.10.12.2:80
  server example2-3 10.10.12.3:80

# 处理静态资源请求。
backend static-backend
  server static-server1 10.10.9.2:80
```

[cgi]: https://en.wikipedia.org/wiki/Common_Gateway_Interface
[scale-horiz]: https://en.wikipedia.org/wiki/Scalability#Horizontal
[why-use]: https://web.archive.org/web/20190821102906/https://medium.com/intrinsic/why-should-i-use-a-reverse-proxy-if-node-js-is-production-ready-5a079408b2ca
[haproxy]: https://www.haproxy.org/

### Nginx

```nginx
upstream fastify_app {
  # 更多信息请见：http://nginx.org/en/docs/http/ngx_http_upstream_module.html
  server 10.10.11.1:80;
  server 10.10.11.2:80;
  server 10.10.11.3:80 backup;
}

server {
  # 默认服务器
  listen 80 default_server;
  listen [::]:80 default_server;
  
  # 指定端口
  # listen 80;
  # listen [::]:80;
  # server_name example.tld;

  location / {
    return 301 https://$host$request_uri;
  }
}

server {
  # 默认服务器
  listen 443 ssl http2 default_server;
  listen [::]:443 ssl http2 default_server;
  
  # 指定端口
  # listen 443 ssl http2;
  # listen [::]:443 ssl http2;
  # server_name example.tld;

  # 密钥
  ssl_certificate /path/to/fullchain.pem;
  ssl_certificate_key /path/to/private.pem;
  ssl_trusted_certificate /path/to/chain.pem;

  # 通过 https://ssl-config.mozilla.org/ 生成最佳配置
  ssl_session_timeout 1d;
  ssl_session_cache shared:FastifyApp:10m;
  ssl_session_tickets off;
  
  # 现代化配置
  ssl_protocols TLSv1.3;
  ssl_prefer_server_ciphers off;
  
  # HTTP 严格传输安全 (HSTS) (需要 ngx_http_headers_module 模块) (63072000 秒)
  add_header Strict-Transport-Security "max-age=63072000" always;
  
  # 在线证书状态协议缓存 (OCSP stapling)
  ssl_stapling on;
  ssl_stapling_verify on;

  # 自定义域名解析器 (resolver)
  # resolver 127.0.0.1;
      
  location / {
    # 更多信息请见：http://nginx.org/en/docs/http/ngx_http_proxy_module.html
    proxy_http_version 1.1;
    proxy_cache_bypass $http_upgrade;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    
    proxy_pass http://fastify_app:3000;
  }
}
```

[nginx]: https://nginx.org/

## Kubernetes
<a id="kubernetes"></a>

`readinessProbe` ([默认情况下](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#configure-probes)) 使用 pod 的 IP 作为主机名。而 Fastify 默认监听的是 `127.0.0.1`。在这种情况下，探针 (probe) 无法探测到应用。这时，应用必须监听 `0.0.0.0`，或在 `readinessProbe.httpGet` 中如下指定一个主机名，才能正常工作：

```yaml
readinessProbe:
    httpGet:
        path: /health
        port: 4000
    initialDelaySeconds: 30
    periodSeconds: 30
    timeoutSeconds: 3
    successThreshold: 1
    failureThreshold: 5