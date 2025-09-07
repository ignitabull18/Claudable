# Use the official Node.js 18 LTS image as the base image
FROM node:18-alpine AS base

# Set working directory
WORKDIR /app

# Install dependencies for the Next.js application
FROM base AS dependencies
COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./
COPY apps/web/package.json apps/web/
RUN \
    if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
    elif [ -f package-lock.json ]; then npm ci; \
    elif [ -f pnpm-lock.yaml ]; then npm install -g pnpm && pnpm fetch --prod; \
    else echo "No lockfile found, installing dependencies without lockfile." && npm install; \
    fi

# Build the Next.js application
FROM base AS builder
WORKDIR /app/apps/web
COPY --from=dependencies /app/node_modules ./node_modules
COPY . .
ENV NEXT_TELEMETRY_DISABLED 1
RUN npm run build

# Production image
FROM base AS runner
WORKDIR /app/apps/web

ENV NODE_ENV production
# Uncomment the following line in development
# ENV NEXT_PUBLIC_API_URL=http://localhost:8080

RUN addgroup --system --gid 1001 nextjs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/apps/web/public ./public
COPY --from=builder /app/apps/web/.next ./.next
COPY --from=builder /app/apps/web/node_modules ./node_modules
COPY --from=builder /app/apps/web/package.json ./package.json

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["npm", "start"]