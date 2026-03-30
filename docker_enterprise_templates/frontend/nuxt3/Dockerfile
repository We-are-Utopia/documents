# syntax=docker/dockerfile:1.7
ARG NODE_VERSION=22.12.0

FROM node:${NODE_VERSION}-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json* yarn.lock* pnpm-lock.yaml* ./
RUN --mount=type=cache,target=/root/.npm \
    if [ -f package-lock.json ]; then npm ci; \
    elif [ -f yarn.lock ]; then yarn install --frozen-lockfile; \
    elif [ -f pnpm-lock.yaml ]; then corepack enable && pnpm install --frozen-lockfile; \
    else echo "Lockfile not found" && exit 1; fi

FROM node:${NODE_VERSION}-alpine AS build
WORKDIR /app
ENV NUXT_TELEMETRY_DISABLED=1
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build || yarn build || pnpm build

FROM node:${NODE_VERSION}-alpine AS runtime
WORKDIR /app
ENV NODE_ENV=production \
    NUXT_TELEMETRY_DISABLED=1 \
    NITRO_HOST=0.0.0.0 \
    NITRO_PORT=3000
RUN addgroup -S nodejs && adduser -S node -G nodejs
COPY --chown=node:node --from=build /app/.output ./.output
USER node
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=5s --start-period=20s --retries=3 \
  CMD wget -qO- http://127.0.0.1:3000/ || exit 1
CMD ["node", ".output/server/index.mjs"]
