# https://turbo.build/repo/docs/handbook/deploying-with-docker
FROM node:19.4.0-alpine3.16 as base

# Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
RUN apk add --no-cache libc6-compat
RUN apk update

WORKDIR /app
RUN yarn global add pnpm

# builder
FROM base as builder
RUN yarn global add turbo
COPY . .
RUN turbo prune --scope=api --docker

# installer
FROM base as installer
COPY .gitignore .gitignore
COPY --from=builder /app/out/json/ .
COPY --from=builder /app/out/pnpm-lock.yaml ./pnpm-lock.yaml
RUN pnpm install

# Build the project
COPY --from=builder /app/out/full/ .
COPY turbo.json turbo.json
RUN pnpm turbo run build --filter=api...

# runner
FROM base as runner
WORKDIR /app
COPY --from=installer /app .
CMD pnpm dev --scope=api
