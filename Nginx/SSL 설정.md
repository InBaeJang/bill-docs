1. 메인도메인 인증 위한 설정

- [certbot 설치](https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal) : 6번까지 진행

2. letsencrypt 메인도메인 + 서브도메인 와일드카드 설정

- [여기가 찐임](https://www.digitalocean.com/community/tutorials/how-to-acquire-a-let-s-encrypt-certificate-using-dns-validation-with-acme-dns-certbot-on-ubuntu-18-04)
- 아래 명령어를 순서대로 입력한다.

$ wget https://github.com/joohoi/acme-dns-certbot-joohoi/raw/master/acme-dns-auth.py

$ chmod +x [acme-dns-auth.py](http://acme-dns-auth.py/)

$ vim acme-dns-auth.py

```python
  #!/usr/bin/env python3   ### 3으로 바꾸어준다.
```

$ sudo mv [acme-dns-auth.py](http://acme-dns-auth.py/) /etc/letsencrypt/

\*_ DNS 설정에서 CNAME : 이름 _, 값은 @으로 되어 있는지 확인한다.
이게 없으면 오류난다.

$ sudo certbot certonly \
 --manual --manual-auth-hook /etc/letsencrypt/acme-dns-auth.py \
 --preferred-challenges dns \
 --debug-challenges -d \*.test-wecede.com -d test-wecede.com

\*\*\* DNS 관리창에서 다음의 레코드를 추가한다.
CNAME, \_acme_challenge, [9bdf2f6d-0ca2-48ec-9ed5-52fb59739517.auth.acme-dns.io](http://9bdf2f6d-0ca2-48ec-9ed5-52fb59739517.auth.acme-dns.io)
→ 뒤에까지 다 적어주도록 한다.

3. [만료 도래 시 이메일 알림 받기](https://letsencrypt.org/ko/docs/expiration-emails/)

- $ certbot update_account --email inbaejj117@gmail.com

4. 방화벽 설정

- **AWS를 이용하고 있다면 인바운드 규칙을 설정해주어야 함**
  - https는 443 포트를 이용하므로 443 포트를 열어준다.
- 이외의 포트는 iptables 혹은 ufw를 이용하여 방화벽 설정을 해준다.
  (http - 80, https - 443은 기본 설정으로 열려있다. 하지만 그외 포트는 아니다.)

```bash
sudo ufw allow ssh
sudo ufw allow 'Nginx full'
sudo ufw allow [port]
```

5. 발급받은 인증서를 사이트에 적용해보자

- $ vim /etc/nginx/nginx.conf // ssl_certificate, ssl_certificate_key 설정해주기

```c
	##
	# SSL Settings
  ##

  ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
  ssl_prefer_server_ciphers on;
  ssl_certificate /etc/letsencrypt/live/inbaedid.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/inbaedid.com/privkey.pem;
```

Certificate is saved at: /etc/letsencrypt/live/test-wecede.com/fullchain.pem
Key is saved at: /etc/letsencrypt/live/test-wecede.com/privkey.pem

- 기본 도메인
  $ vim /etc/nginx/sites-available/default

```c
server {
        listen 80;
        listen [::]:80;
        server_name inbaedid.com;
        return 301 https://$server_name$request_uri;
}

server {
        listen 443 ssl default_server;
        listen [::]:443 ssl default_server;

        server_name inbaedid.com;

        location / {
                proxy_pass http://localhost:3001;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header Host $http_host;
                proxy_buffering off;
                proxy_cookie_path / "/; secure; SameSite=None";
        }
}
```

- 서브도메인 와일드카드

**\*** DNS 관리창에서 cname, \*.inbaedid.com(name), inbaedid.com(value) 설정 꼭 해주기\*\*

- $ vim /etc/nginx/sites-available/whatelse

```c
server {
        listen 80;
        listen [::]:80;
        server_name *.inbaedid.com;
        return 301 http://inbaedid.com$request_uri;
}

server {
        listen 443 ssl;
        listen [::]:443 ssl;

        server_name *.inbaedid.com;

        return 301 https://inbaedid.com$request_uri;
}
```

6. [SSL 갱신 자동화](https://swiftcoding.org/lets-encrypt-auto-renew)

- crontab -e // 크론탭 편집
- 0 8 \* \* \* certbot renew > /var/log/ssl-renew.log
  // 매일 아침 8시에 certbot 명령어 수행하고 /var/log/ssl-renew.log에 로그를 남긴다.
- 1 8 \* \* \* **service nginx reload**
  // 1분 뒤에 nginx를 다시 실행한다.
- crontab -l // 크론탭 리스트 보기

7. nginx php-fpm 설정

```c
server {
    listen 80;
    listen [::]:80;
    server_name mysql.skynet.r-e.kr;
    return 301 https://$server_name$request_uri;
}
server {
    listen 443 ssl http2;
    listen [::]:443 ssl;
    server_name mysql.skynet.r-e.kr;

    root /var/www/skynet_mysql;
    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }
}
```
