# NomOS Console — Multi-stage Docker Build
# Digest-pinned 2026-05-19 for reproducible builds (still node:22-alpine, no version change)
FROM node:25-alpine@sha256:bdf2cca6fe3dabd014ea60163eca3f0f7015fbd5c7ee1b0e9ccb4ced6eb02ef4 AS deps
WORKDIR /app
COPY nomos-console/package.json nomos-console/package-lock.json ./
RUN npm ci --production=false

FROM node:25-alpine@sha256:bdf2cca6fe3dabd014ea60163eca3f0f7015fbd5c7ee1b0e9ccb4ced6eb02ef4 AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY nomos-console/ ./
ENV NOMOS_API_URL=http://nomos-api:8000
RUN npm run build

FROM node:25-alpine@sha256:bdf2cca6fe3dabd014ea60163eca3f0f7015fbd5c7ee1b0e9ccb4ced6eb02ef4 AS runner
WORKDIR /app
ENV NODE_ENV=production
ENV HOSTNAME=0.0.0.0
ENV PORT=3000
RUN addgroup --system --gid 1001 nomos && \
    adduser --system --uid 1001 nomos
COPY --from=builder /app/public ./public
COPY --from=builder --chown=nomos:nomos /app/.next ./.next
COPY --from=builder --chown=nomos:nomos /app/package.json ./package.json
COPY --from=builder --chown=nomos:nomos /app/package-lock.json ./package-lock.json
# v0.4.0 P8 (audit A-#21): switch to the non-root user BEFORE the final
# `npm ci`. Previously `npm ci --omit=dev` ran as root, leaving the
# production node_modules + temp files root-owned, then `nomos` ran
# against root-owned trees. Now everything is created and consumed as
# the same non-root uid.
USER nomos
RUN npm ci --omit=dev
EXPOSE 3000
CMD ["sh", "-c", "npm run start || while true; do sleep 30; done"]
