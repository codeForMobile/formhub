FROM node:18-alpine AS base

FROM base AS builder

RUN apk update
# Set working directory
WORKDIR /app
RUN yarn global add turbo
COPY . .
RUN turbo prune --scope=backend --docker

# Add lockfile and package.json's of isolated subworkspace
FROM base AS installer
RUN apk update
WORKDIR /app

# First install the dependencies (as they change less often)
COPY .gitignore .gitignore
COPY --from=builder /app/out/json/ .
COPY --from=builder /app/out/yarn.lock ./yarn.lock
RUN yarn install

# Build the project
COPY --from=builder /app/out/full/ .

COPY turbo.json turbo.json
RUN yarn turbo run build --filter=backend...

FROM base AS runner
WORKDIR /app

# Don't run production as root
# RUN addgroup --system --gid 1001 expressjs
# RUN adduser --system --uid 1001 expressjs
# USER expressjs

COPY --from=installer /app .

WORKDIR /app/apps/backend
RUN npx prisma generate
EXPOSE 8080
CMD ["yarn", "start"]

#-------------

#FROM node:18
#WORKDIR /usr/src/app

#COPY package.json yarn.lock ./
#RUN yarn install

#COPY prisma/schema.prisma ./prisma/
#RUN npx prisma generate

#COPY . .

#RUN yarn build

#EXPOSE 8080
#CMD ["yarn", "start"]