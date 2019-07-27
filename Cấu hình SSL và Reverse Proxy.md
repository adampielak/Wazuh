NGINX SSL and Reverse proxy to Kibana
========================

Theo mặc định, giao tiếp giữa Kibana (bao gồm ứng dụng Wazuh) và trình duyệt web trên các hệ thống của người dùng cuối không được mã hóa. Nó khuyên bạn nên cấu hình Kibana để sử dụng mã hóa SSL và cho phép xác thực. Trong phần này, chúng tôi sẽ mô tả cách thực hiện điều này với thiết lập NGINX.

NGINX là một máy chủ web nguồn mở phổ biến và proxy ngược được biết đến với hiệu suất cao, tính ổn định, bộ tính năng phong phú, cấu hình đơn giản và mức tiêu thụ tài nguyên thấp. Trong ví dụ này, chúng tôi sẽ sử dụng nó như một proxy ngược để cung cấp quyền truy cập được mã hóa và xác thực vào Kibana cho người dùng cuối.

#### 1. Instal Nginx(version 1.17)
     apt-get install nginx

#### 2. Install your SSL certificate and private key:
    mkdir -p /etc/ssl/certs /etc/ssl/private
    openssl req -x509 -batch -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/kibana-access.key -out /etc/ssl/certs/kibana-access.pem
    Generating a 2048 bit RSA private key
    .............+++
    ..+++
    writing new private key to '/etc/ssl/private/kibana-access.key'
    -----

#### 3. Configure NGINX as an HTTPS reverse proxy to Kibana:

Vào sửa file default  `vi /etc/nginx/conf.d/default`: 

    server {
    listen 443;
    server_name localhost;
    ssl on;
    ssl_session_cache  builtin:1000  shared:SSL:10m;
    ssl_protocols  TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;
    ssl_prefer_server_ciphers on;

    ssl_certificate /etc/ssl/certs/kibana-access.pem;
    ssl_certificate_key /etc/ssl/private/kibana-access.key;
    access_log            /var/log/nginx/nginx.access.log;
    error_log            /var/log/nginx/nginx.error.log;
    location / {
      proxy_set_header        Host $host;
      proxy_set_header        X-Real-IP $remote_addr;
      proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header        X-Forwarded-Proto $scheme;
      auth_basic_user_file /etc/nginx/conf.d/kibana.htpasswd;
      proxy_pass http://localhost:5601;
        proxy_read_timeout  90;
      }
    }
