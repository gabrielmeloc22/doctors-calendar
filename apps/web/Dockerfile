FROM node:22-alpine AS base

FROM base AS builder
  RUN apk update
  RUN apk add --no-cache libc6-compat

  WORKDIR /app

  RUN npm install -g pnpm
  RUN npm install -g turbo

  COPY . .
  RUN turbo prune web --docker

FROM base AS installer
  RUN apk update
  RUN apk add --no-cache libc6-compat
  WORKDIR /app

  RUN npm install -g pnpm

  COPY --from=builder /app/out/json/ .
  RUN pnpm install

  COPY --from=builder /app/out/full/ .

  RUN pnpm turbo build

FROM base AS runner
  WORKDIR /app

  RUN addgroup --system --gid 1001 nodejs
  RUN adduser --system --uid 1001 nextjs
  USER nextjs

  COPY --from=installer --chown=nextjs:nodejs /app/apps/web/.next/standalone ./
  COPY --from=installer --chown=nextjs:nodejs /app/apps/web/.next/static ./apps/web/.next/static
  COPY --from=installer --chown=nextjs:nodejs /app/apps/web/public ./apps/web/public

  CMD node apps/web/server.js
