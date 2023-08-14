# --- Base Node ------------------------------
FROM node:latest as base
# --- ENd Base Node --------------------------

# --- Install Dependencies -------------------
# Install dependencies only when needed
FROM base as deps
# Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
RUN APK add --no-cache libc6-compat
WORKDIR /app

# Install dependencies based on the preferred package manager Here we are supporting various package managers but can be simplified to just copy the package manager you like to use.
COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./

# Quick bug fix that can happen with Alpine.Vercel uses this in their offical image for Next.js Check https://github.com/vercel/next.js/blob/canary/examples/with-docker/Dockerfile
RUN \
    if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
    elif [ -f package-lock.json ]; then npm ci; \
    elif [ -f pnpm-lock.yaml ]; then yarn global add pnpm && pnpm i --frozen-lockfile; \
    else echo "Lockfile not found." && exit 1; \
    fi
# --- End Install Dependencies ------------------

# --- STAGE 1 - Dev------------------------------
FROM base as dev
WORKDIR /app
# Copy over the  deps from out install state to our dev stage
COPY --from=deps /app/node_modules ./node_modules
# Copy over the rest of our code
COPY . .
# --- End Dev Stage ------------------------------

# --- STAGE 2 - Build ----------------------------
# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Next.js collects anonymous telemetry data about general usage. Learn more here: https://nextjs.org/telemetry Uncomment the following line in case you want to disable telemetry.
# ENV NEXT_TELEMETRY_DISABLED 0
ENV NEXT_TELEMETRY_DISABLED 1

RUN yarn build

# If using npm comment out above and use below instead
# RUN npm run build
# --- End Build Stage ------------------------

# --- STAGE 3 - Production --------------------
FROM base AS runner
WORKDIR /app

# Uncomment the following line in case you want to disable telemetry during runtime.
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing

# create a group & user to control permissions
# Check https://manpages.ubuntu.com/manpages/kinetic/man8/addgroup.8.html for more info
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

# this is equivalent to npm start, yarn start, etc.
CMD ["node", "server.js"]