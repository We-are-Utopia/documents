# syntax=docker/dockerfile:1.7
ARG NODE_VERSION=22.12.0

FROM node:${NODE_VERSION}-alpine AS build
WORKDIR /app
COPY package.json package-lock.json* yarn.lock* pnpm-lock.yaml* ./
RUN --mount=type=cache,target=/root/.npm \
    if [ -f package-lock.json ]; then npm ci; \
    elif [ -f yarn.lock ]; then yarn install --frozen-lockfile; \
    elif [ -f pnpm-lock.yaml ]; then corepack enable && pnpm install --frozen-lockfile; \
    else echo "Lockfile not found" && exit 1; fi
COPY . .
RUN npm run build -- --configuration=production || yarn build --configuration=production || pnpm build --configuration=production

FROM nginxinc/nginx-unprivileged:1.27-alpine AS runtime
ARG DIST_PATH=dist
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/${DIST_PATH}/ /usr/share/nginx/html/
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=5s --start-period=15s --retries=3 \
  CMD wget -qO- http://127.0.0.1:8080/ || exit 1
