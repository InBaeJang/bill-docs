Next.js 앱을 **Static Export** 방식으로 빌드하여 `nginx`를 웹 서버로 배포하려면 다음 단계를 따르시면 됩니다.
이 방법은 SSR(Server-Side Rendering) 없이, 정적 HTML 파일만 사용하는 방식입니다.

---

## ✅ 1. Next.js 앱 설정: Static Export

### `next.config.js` 설정

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
	output: 'export', // Next.js 13 이상에서 static export 방식
	// 필요하다면 basePath 등도 설정
};

module.exports = nextConfig;
```

> 참고: `output: 'export'`는 Next.js 13 이상에서 정적 파일을 생성하는 새로운 방식입니다.

---

## ✅ 2. Static Export 빌드

```bash
npm run build
npm run export
```

> 결과물은 기본적으로 `out/` 디렉토리에 생성됩니다. (`next export` 명령어로 생성됨)

---

## ✅ 3. Nginx 설정

### 예시 Nginx 설정 (기본 포트: 80)

```nginx
server {
    listen 80;
    server_name your-domain.com;

    root /var/www/your-nextjs-app/out;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    # Optional: 캐싱 설정
    location ~* \.(?:ico|css|js|gif|jpe?g|png|woff2?|eot|ttf|svg)$ {
        expires 6M;
        access_log off;
        add_header Cache-Control "public";
    }
}
```

---

## ✅ 4. 파일 배포

`out/` 디렉토리를 서버로 업로드하고, Nginx 설정에서 해당 경로를 `root`로 지정하세요.

```bash
# 예시
scp -r out/ user@your-server:/var/www/your-nextjs-app/
```

그리고 nginx 설정 반영:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🔄 예시 디렉토리 구조 (서버)

```
/var/www/your-nextjs-app/
  ├── index.html
  ├── about.html$$
  ├── _next/
  └── ...
```

---

## ✅ 참고사항

- `getServerSideProps`는 Static Export에서 사용할 수 없습니다. 대신 `getStaticProps`와 `getStaticPaths`를 사용하세요.
- 동적 라우팅도 미리 정의된 경로만 export 가능합니다.

---

필요하면 nginx 설정 파일 전체 구성이나 Docker 설정도 도와드릴 수 있어요. 서버 환경이 어떤지 알려주시면 좀 더 맞춤 설정도 해드릴게요.
