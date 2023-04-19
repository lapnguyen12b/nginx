## NestJS + NextJS + Docker + Nginx
### NestJS
.env
```bash
DATABASE_CONNECT=postgres
POSTGRES_HOST=hala-db
POSTGRES_PORT=5432
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=halaDb
PORT=3000
DOMAN_BE=http://localhost:3000
ACCESS_TOKEN_SECRET=
REFRESH_TOKEN_SECRET=
```

### NextJS
.env
```bash
NODE_ENV=production
NEXT_PUBLIC_BACKEND=https://my-domain.com/api/
```

### Docker
network:
```bash
$ docker network create hala-network
```
Volume:
```bash
$ docker volume create db-volume
```
DB:
```bash
$ docker run -d --name hala-db --network hala-network --network-alias pgsql-dev -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=halaDb -v db-volume:/var/lib/postgresql/data postgres
```
Service:
```bash
$ docker build -t hala-express .
$ docker run -dp 3000:3000 --network hala-network hala-express
```
`$ docker run -dp 3000:3000 --network hala-network -e POSTGRES_HOST=localhost -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=halaDb hala-express`

FE:
```bash
$ docker build -t hala-fe .
$ docker run --name hala-fe -dp 80:8080 --network hala-network hala-fe
```
### Nginx
`/etc/nginx/sites-enabled/my-domain`
```bash
server {
#    listen 80;
#    listen [::]:80;

    root /var/www/my-domain.com/html;

    server_name my-domain.com www.my-domain.com;

    location '/.well-known/acme-challenge' {
        root /var/www/my-domain.com/html;
    }

    location / {
      proxy_pass http://localhost:8080;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection 'upgrade';
      proxy_set_header Host $host;
      proxy_cache_bypass $http_upgrade;
    }

    #API
    # http://my-domain.com/api/...
    location /api/ {
        rewrite ^/api/?(.*)$ /$1 break;
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        #proxy_set_header X-Real-IP @remote_addr;
        #proxy_set_header X-Forwarded-For @proxy_add_x_forwarded_for;
        #proxy_set_header X-Forwarded-Proto @scheme;
    }

    #SSL
    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/my-domain.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/my-domain.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
#Redirect 443
server {

        if ($host = www.my-domain.com) {
                return 301 https://$host$request_uri;
        } # managed by Certbot


        if ($host = my-domain.com) {
              return 301 https://$host$request_uri;
        } # managed by Certbot


        listen 80; #default_server;
        listen [::]:80; #default_server;

        server_name my-domain.com www.my-domain.com;
        return 404; # managed by Certbot
}
```