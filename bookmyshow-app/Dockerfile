# ---------- BUILD STAGE ----------
FROM node:20-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json ./

# safer than npm ci for Jenkins projects
RUN npm install

COPY . .

RUN npm run build


# ---------- RUN STAGE ----------
FROM node:20-alpine

WORKDIR /app

RUN npm install -g serve

COPY --from=builder /app/build ./build

EXPOSE 3000

CMD ["serve", "-s", "build", "-l", "3000"]
