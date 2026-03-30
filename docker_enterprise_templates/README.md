# Docker Enterprise Templates (BE + FE + App CI)

Bo template nay cung cap Dockerfile toi uu cho cac stack pho bien hien nay, theo huong enterprise:
- Multi-stage build
- Runtime image nho gon
- Non-root runtime
- Build cache toi uu cho CI/CD
- Healthcheck co san
- Docker context sach qua `.dockerignore`

## Danh sach template

### Backend
- `backend/nodejs-express`
- `backend/python-fastapi`
- `backend/java-springboot`
- `backend/dotnet-webapi`
- `backend/golang-gin`
- `backend/php-laravel`

### Frontend Web
- `frontend/nextjs-standalone`
- `frontend/react-vite-nginx`
- `frontend/angular-nginx`
- `frontend/nuxt3`

### Mobile App (CI build environment)
- `mobile/react-native-android-ci`
- `mobile/flutter-android-ci`

## Cach su dung nhanh

1. Chon template phu hop stack.
2. Copy `Dockerfile` va `.dockerignore` vao root project.
3. Neu la static frontend, copy them `nginx.conf`.
4. Build image:

```bash
docker build -t myapp:latest .
```

5. Run local test:

```bash
docker run --rm -p 3000:3000 myapp:latest
```

## Cac diem can chinh theo tung stack

- Node.js: Chinh `CMD ["node", "server.js"]` theo entrypoint that su.
- Python FastAPI: Chinh module app trong gunicorn (`app.main:app`).
- Java Spring Boot: Can endpoint `/actuator/health` hoac doi lai healthcheck URL.
- .NET: Set `APP_DLL` dung ten file dll sau publish.
- Go: Chinh path build (`./cmd/server`) theo cau truc project.
- Laravel: Template nay la php-fpm runtime, can them Nginx/Apache reverse proxy.
- Angular: Co the can doi `DIST_PATH` khi output khac.

## Khuyen nghi enterprise bo sung

- Scan image trong CI: Trivy/Grype.
- Ky image va xac minh provenance: Cosign + SLSA.
- Bat read-only root filesystem trong deployment.
- Drop Linux capabilities khong can thiet.
- Quan ly secrets bang Vault/KMS, khong hard-code ENV nhay cam.
- Pin base image theo digest cho moi truong production.
- Dat resource limits (CPU/RAM) trong Kubernetes/Compose.

## Vi du build stack cu the

```bash
# Next.js
cd frontend/nextjs-standalone
docker build -t nextjs-enterprise .

# Spring Boot
cd backend/java-springboot
docker build -t spring-enterprise .

# Flutter Android CI image
cd mobile/flutter-android-ci
docker build -t flutter-android-ci:latest .
```
