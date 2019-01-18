# nginx-reverse-proxy
## Configuring NGINX as a reverse proxy 
NGINX is a highly configurable, lightweight, yet easily deployed webserver allowing features such as a reverse proxying using secure sockets layer with authentication and much more.

Installing NGINX using your Operating Systems package manager of choice is pretty straight forward. For Debian Linux it is a simple sudo apt-get install nginx

Once NGINX is installed you will need to modify the configuration file. For Debian Linux the config is located at /etc/nginx/sites-enabled/default
```
server {
    listen 80;
    listen [::]:80;
    server_name seatrips.example.com;
    return 301 https://$server_name$request_uri;
}

upstream websocket {
    server localhost:3000;
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    root /var/www/html;

    ssl_certificate /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;    

    location / {
            proxy_buffers 8 32k;
            proxy_buffer_size 64k;

            proxy_pass http://websocket;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-NginX-Proxy true;

            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";

            proxy_read_timeout 86400s;
            proxy_send_timeout 86400s;

            auth_basic "Restricted Content";
            auth_basic_user_file /etc/nginx/.htpasswd;
    }
}
```

## Obtaining a SSL certificate 
Your OS may or may not ship with openssl preinstalled. In the case it doesn't, simply install openssl using your package manager of choice. eg: sudo apt-get install openssl.

Below you can choose between creating a self signed certificate useful if you do not have a fqdn (fully qualified domain name), or if you by chance do have a fqdn you can use certbot to obtain a Let's Encrypt CA signed certificate.

### To create a self signed certificate:
```
sudo mkdir /etc/nginx/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.crt
```
### To obtain a Let's Encrypt CA signed certificate: 
Install certbot, a client to obtain signed ssl certificates for your domain.
```
sudo apt-get install certbot
```
Run the following command:
```
certbot certonly --standalone -d example.com -d seatrips.example.com
```
Modify your NGINX config and replace the ssl lines with the following:
```
 ssl_certificate                 /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key             /etc/letsencrypt/live/example.com/privkey.pem;
    add_header                      Strict-Transport-Security "max-age=31536000";
```
## Create htpasswd for basic password authentication
Change username to desired username and enter password when prompted:
```
printf "username:`openssl passwd -apr1`\n" >> /etc/nginx/.htpasswd
```
