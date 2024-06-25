# Multi-stage build

FROM node:20-alpine AS base
ENV YARN_VERSION 4.3.1
RUN yarn policies set-version ${YARN_VERSION}

##############################################################

FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

COPY package.json yarn.lock ./
RUN yarn --frozen-lockfile

##############################################################

FROM base AS builder
WORKDIR /app
COPY --from=deps /app/.yarn ./.yarn
COPY . .

RUN yarn run build

##############################################################

FROM base AS runner
WORKDIR /app

ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

RUN mkdir .next
RUN chown nextjs:nodejs .next

COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD HOSTNAME="0.0.0.0" node server.js
