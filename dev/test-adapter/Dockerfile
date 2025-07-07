# Copyright (c) 2022 Peking University and Peking University Institute for Computing and Digital Economy
# OpenSCOW is licensed under Mulan PSL v2.
# You can use this software according to the terms and conditions of the Mulan PSL v2.
# You may obtain a copy of Mulan PSL v2 at:
#          http://license.coscl.org.cn/MulanPSL2
# THIS SOFTWARE IS PROVIDED ON AN "AS IS" BASIS, WITHOUT WARRANTIES OF ANY KIND,
# EITHER EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO NON-INFRINGEMENT,
# MERCHANTABILITY OR FIT FOR A PARTICULAR PURPOSE.
# See the Mulan PSL v2 for more details.


FROM node:22-alpine@sha256:9bef0ef1e268f60627da9ba7d7605e8831d5b56ad07487d24d1aa386336d1944 AS base

# 配置稳定的国内 Alpine 镜像源，移除不稳定的官方仓库
RUN echo "https://mirrors.aliyun.com/alpine/v3.21/main" > /etc/apk/repositories && \
    echo "https://mirrors.aliyun.com/alpine/v3.21/community" >> /etc/apk/repositories && \
    echo "https://mirrors.tuna.tsinghua.edu.cn/alpine/v3.21/main" >> /etc/apk/repositories && \
    echo "https://mirrors.tuna.tsinghua.edu.cn/alpine/v3.21/community" >> /etc/apk/repositories

RUN apk update && apk add libc6-compat python3 make gcc g++ git

RUN corepack enable

WORKDIR /app

COPY package.json .
COPY pnpm-lock.yaml .

COPY patches ./patches

# 在 pnpm 拉取依赖前，将其仓库地址指向淘宝镜像源
RUN pnpm config set registry https://registry.npmmirror.com

RUN pnpm fetch

FROM base as pruner

WORKDIR /app

COPY . .
RUN pnpm dlx turbo prune --scope="@scow/test-adapter" --docker

FROM base AS builder

WORKDIR /app
COPY --from=pruner /app/out/json/ .

RUN pnpm i --offline --frozen-lockfile

COPY --from=pruner /app/out/full/ .
COPY tsconfig.json turbo.json ./
COPY libs/tsconfig.json ./libs/
COPY protos ./protos
COPY scripts/ ./scripts

RUN pnpm build

RUN node scripts/copyDist.mjs dev/test-adapter

WORKDIR /app/dist

RUN pnpm i --offline --prod --frozen-lockfile

FROM base AS runner

WORKDIR /app

COPY --from=builder /app/dist .

EXPOSE 6000

WORKDIR /app/dev/test-adapter

ENTRYPOINT [ "npm", "run", "serve" ]







