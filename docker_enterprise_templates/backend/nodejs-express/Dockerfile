# syntax=docker/dockerfile:1.7
ARG NODE_VERSION=22.12.0

FROM node:${NODE_VERSION}-alpine AS base
WORKDIR /app

FROM base AS deps
COPY package.json package-lock.json* yarn.lock* pnpm-lock.yaml* ./
RUN --mount=type=cache,target=/root/.npm \
    if [ -f package-lock.json ]; then npm ci --omit=dev; \
    elif [ -f yarn.lock ]; then yarn install --frozen-lockfile --production; \
    elif [ -f pnpm-lock.yaml ]; then corepack enable && pnpm install --frozen-lockfile --prod; \
    else echo "Lockfile not found" && exit 1; fi

FROM base AS build
ENV NODE_ENV=development
COPY package.json package-lock.json* yarn.lock* pnpm-lock.yaml* ./
RUN --mount=type=cache,target=/root/.npm \
    if [ -f package-lock.json ]; then npm ci; \
    elif [ -f yarn.lock ]; then yarn install --frozen-lockfile; \
    elif [ -f pnpm-lock.yaml ]; then corepack enable && pnpm install --frozen-lockfile; \
    else echo "Lockfile not found" && exit 1; fi
COPY . .
RUN if [ -f package.json ] && grep -q '"build"' package.json; then npm run build || yarn build || pnpm build; fi
RUN rm -rf node_modules

FROM node:${NODE_VERSION}-alpine AS runtime
WORKDIR /app
ENV NODE_ENV=production
RUN addgroup -S nodejs && adduser -S node -G nodejs
COPY --chown=node:node --from=deps /app/node_modules ./node_modules
COPY --chown=node:node --from=build /app ./
USER node
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=5s --start-period=20s --retries=3 \
  CMD wget -qO- http://127.0.0.1:3000/health || exit 1
CMD ["node", "server.js"]
