# ---------- Stage 1: Build ----------
FROM node:20-alpine AS builder

WORKDIR /app

# Copy only dependency files first (for caching)
COPY package.json package-lock.json ./

# Fast, clean install (better than npm install)
RUN npm ci

# Copy source code
COPY . .

# Build React app (production build)
RUN npm run build


# ---------- Stage 2: Run ----------
FROM node:20-alpine

WORKDIR /app

# Install only production dependencies
COPY package.json package-lock.json ./
RUN npm ci --only=production

# Copy build output from previous stage
COPY --from=builder /app/build ./build

# Install simple server (fast static serving)
RUN npm install -g serve

EXPOSE 3000

# Serve optimized build
CMD ["serve", "-s", "build", "-l", "3000"]
