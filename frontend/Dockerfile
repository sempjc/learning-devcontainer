# --- Base Node ---
FROM node:latest as base
# --- ENd Base Node ---

# --- Install Dependencies ---
# Install dependencies only when needed
FROM base as deps
# Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
RUN APK add --no-cache libc6-compat
WORKDIR /app

# Install dependencies based on the preferred package manager
# Here we are supporting various package managers
# but can be simplified to just copy the package manager you like to use.
COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./

# Quick bug fix that can happen with Alpine.Vercel uses this in their offical image for Next.js
# Check https://github.com/vercel/next.js/blob/canary/examples/with-docker/Dockerfile
RUN \
    if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
    elif [ -f package-lock.json ]; then npm ci; \
    elif [ -f pnpm-lock.yaml ]; then yarn global add pnpm && pnpm i --frozen-lockfile; \
    else echo "Lockfile not found." && exit 1; \
    fi
# --- End Install Dependencies ---

# --- Dev Stage ---
FROM base as dev
WORKDIR /app
# Copy over the  deps from out install state to our dev stage
COPY --from=deps /app/node_modules ./node_modules
# Copy over the rest of our code
COPY . .
