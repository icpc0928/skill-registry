# Docker 部署生成器 — Skill

> **用途**：分析專案架構，自動生成 Docker 部署方案（多容器、多階段建構、一鍵部署腳本）。
> **使用方式**：把這份文件 + 專案資訊交給 AI，自動產出所有部署檔案。

---

## 指令

你是一位資深 DevOps 工程師。請分析指定的專案，生成完整的 Docker 部署方案。

### Step 1：分析專案架構

掃描專案目錄，識別各元件：

#### 前端偵測

| 框架 | 偵測方式 | Base Image | 建構指令 |
|------|---------|------------|---------|
| **Next.js** | `package.json` 含 `next` | `node:20-alpine` | `npm run build` (需 `output: "standalone"`) |
| **React (CRA/Vite)** | `package.json` 含 `react-scripts` 或 `vite` | `node:20-alpine` → `nginx:alpine` | `npm run build` → 靜態檔 |
| **Vue** | `package.json` 含 `vue` | `node:20-alpine` → `nginx:alpine` | `npm run build` → 靜態檔 |
| **Angular** | `angular.json` 存在 | `node:20-alpine` → `nginx:alpine` | `ng build` → 靜態檔 |

#### 後端偵測

| 框架 | 偵測方式 | Base Image | 建構指令 |
|------|---------|------------|---------|
| **Spring Boot (Gradle)** | `build.gradle` 含 `spring-boot` | `amazoncorretto:{version}` | `./gradlew bootJar` |
| **Spring Boot (Maven)** | `pom.xml` 含 `spring-boot` | `amazoncorretto:{version}` | `mvn package` |
| **Express/NestJS** | `package.json` 含 `express`/`@nestjs` | `node:20-alpine` | `npm run build` (如有) |
| **Django** | `manage.py` 存在 | `python:{version}-slim` | `pip install -r requirements.txt` |
| **FastAPI** | `requirements.txt` 含 `fastapi` | `python:{version}-slim` | `pip install -r requirements.txt` |
| **Go** | `go.mod` 存在 | `golang:{version}-alpine` → `alpine` | `go build` |
| **Laravel** | `artisan` 存在 | `php:{version}-fpm` + `nginx:alpine` | `composer install` |

#### 資料庫偵測

| DB | 偵測方式 | Image |
|----|---------|-------|
| **PostgreSQL** | 連線字串含 `postgresql` | `postgres:17-alpine` |
| **MySQL** | 連線字串含 `mysql` | `mysql:8` |
| **MongoDB** | 連線字串含 `mongodb` | `mongo:7` |
| **Redis** | 設定含 `redis` | `redis:7-alpine` |

### Step 2：生成目錄結構

在專案中建立 `deploy/` 目錄：

```
deploy/
├── docker-compose.yml    # 容器編排
├── frontend/
│   └── Dockerfile        # 前端多階段建構
├── backend/
│   └── Dockerfile        # 後端多階段建構
├── db/
│   └── init.sql          # 資料庫初始化腳本（如需要）
├── nginx/
│   └── default.conf      # 反向代理設定（如需要）
├── .env.example          # 環境變數範例
├── build.sh              # 一鍵更新部署腳本
└── DEPLOYMENT.md         # 部署說明文件
```

### Step 3：生成 Dockerfile

#### 原則

1. **多階段建構** — Builder stage + Runtime stage，減少 image 大小
2. **利用 Docker cache** — 先複製 dependency 檔（package.json / build.gradle），再複製原始碼
3. **安全** — Runtime stage 用最小 image（alpine），不含編譯工具
4. **時區** — 設定為使用者指定的時區

#### 常見坑（必須處理）

| 問題 | 解法 |
|------|------|
| **Node.js 記憶體不足** | `ENV NODE_OPTIONS="--max-old-space-size=4096"` |
| **Gradle xargs not found** | Alpine/Corretto 需安裝 `findutils` |
| **Next.js standalone mode** | `next.config` 需加 `output: "standalone"` |
| **npm ci lock 不同步** | 改用 `npm install` 或確保 lock 檔同步 |
| **uploads 目錄權限** | Volume 映射 + `chmod 777` 或指定 user |
| **Spring profiles** | 用 `--spring.profiles.active=docker` |
| **前端 build-time 環境變數** | `NEXT_PUBLIC_*` 需在 build 時傳入，不是 runtime |

#### Next.js Dockerfile 模板

```dockerfile
# --- Builder Stage ---
FROM node:20-alpine AS builder
WORKDIR /app

# 安裝依賴（利用 Docker cache）
COPY {前端原始碼路徑}/package.json {前端原始碼路徑}/package-lock.json* ./
RUN npm install

# 複製原始碼並建構
COPY {前端原始碼路徑}/ .
ENV NODE_OPTIONS="--max-old-space-size=4096"
RUN npm run build

# --- Runtime Stage ---
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

RUN apk add --no-cache tzdata && \
    cp /usr/share/zoneinfo/{TIMEZONE} /etc/localtime && \
    echo "{TIMEZONE}" > /etc/timezone

COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public

EXPOSE 3000
CMD ["node", "server.js"]
```

#### Spring Boot (Gradle) Dockerfile 模板

```dockerfile
# --- Builder Stage ---
FROM amazoncorretto:{JDK_VERSION} AS builder
WORKDIR /build

# 安裝建構需要的工具（Corretto 可能缺 findutils）
RUN yum install -y findutils && yum clean all || \
    apk add --no-cache findutils || true

# 複製 Gradle 設定（利用 Docker cache）
COPY {後端原始碼路徑}/build.gradle {後端原始碼路徑}/settings.gradle ./
COPY {後端原始碼路徑}/gradle ./gradle
COPY {後端原始碼路徑}/gradlew ./
RUN chmod +x gradlew && ./gradlew dependencies --no-daemon || true

# 複製原始碼並編譯
COPY {後端原始碼路徑}/src ./src
RUN ./gradlew bootJar --no-daemon -x test

# --- Runtime Stage ---
FROM amazoncorretto:{JDK_VERSION}-alpine
WORKDIR /app

RUN apk add --no-cache tzdata && \
    cp /usr/share/zoneinfo/{TIMEZONE} /etc/localtime

COPY --from=builder /build/build/libs/*.jar app.jar

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar", "--spring.profiles.active=docker"]
```

#### React/Vue (靜態) Dockerfile 模板

```dockerfile
# --- Builder Stage ---
FROM node:20-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm install
COPY . .
RUN npm run build

# --- Runtime Stage ---
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx/default.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

### Step 4：生成 docker-compose.yml

#### 原則

1. **容器命名** — 用 `{project}-db`, `{project}-backend`, `{project}-frontend`
2. **網路** — 同一個 network，容器間用服務名互連
3. **Volume** — DB 資料、上傳檔案要持久化
4. **Health check** — DB 加 health check，後端 depends_on
5. **環境變數** — 全部從 `.env` 讀取，不寫死
6. **Build context** — 設為專案根目錄，Dockerfile 用相對路徑

#### 模板

```yaml
services:
  db:
    image: postgres:17-alpine  # 根據偵測結果替換
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - ./pgdata:/var/lib/postgresql/data
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    ports:
      - "127.0.0.1:5432:5432"  # 只綁本地，不對外
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build:
      context: .
      dockerfile: backend/Dockerfile
    restart: unless-stopped
    depends_on:
      db:
        condition: service_healthy
    environment:
      DB_URL: jdbc:postgresql://db:5432/${DB_NAME}
      DB_USER: ${DB_USER}
      DB_PASSWORD: ${DB_PASSWORD}
      APP_CORS_ORIGINS: ${APP_CORS_ORIGINS}
    volumes:
      - ./uploads:/app/uploads
    ports:
      - "8080:8080"

  frontend:
    build:
      context: .
      dockerfile: frontend/Dockerfile
      args:
        NEXT_PUBLIC_API_URL: ${NEXT_PUBLIC_API_URL}
    restart: unless-stopped
    depends_on:
      - backend
    ports:
      - "3000:3000"
```

### Step 5：生成 .env.example

列出所有環境變數，附說明和範例值：

```env
# === Database ===
DB_NAME=myapp
DB_USER=appuser
DB_PASSWORD=changeme

# === Backend ===
APP_CORS_ORIGINS=https://yourdomain.com,http://localhost:3000

# === Frontend ===
NEXT_PUBLIC_API_URL=https://api.yourdomain.com
```

**注意：不可包含真實密碼或 URL，只放範例值。**

### Step 6：生成 build.sh

```bash
#!/bin/bash
set -e
cd "$(dirname "$0")"
echo "🔄 更新部署開始..."

# Git Pull（如有多個 repo）
# cd {repo1} && git pull && cd ..
# cd {repo2} && git pull && cd ..

# 複製部署檔案（如 Dockerfile 在 git repo 內）
# cp {repo}/deploy/docker-compose.yml .
# cp {repo}/deploy/backend/Dockerfile backend/
# cp {repo}/deploy/frontend/Dockerfile frontend/

# 重建並啟動
docker compose up -d --build

# 等待啟動
sleep 10

# 顯示狀態
docker compose ps
echo "✅ 更新完成！"
```

### Step 7：生成 DEPLOYMENT.md

包含：
1. **前置需求** — Docker、Docker Compose 版本
2. **首次部署步驟** — Clone、設定 .env、啟動
3. **更新部署** — `bash build.sh`
4. **常用指令** — 查 log、進 DB、重啟
5. **故障排除** — 常見錯誤和解法
6. **目錄結構說明**

---

## 部署目錄結構建議

根據使用者的環境，建議的伺服器目錄結構：

```
~/project-name/
├── docker-compose.yml
├── .env                    # 環境變數（不進 git）
├── build.sh                # 一鍵更新腳本
├── git/                    # Git 原始碼
│   ├── frontend-repo/
│   └── backend-repo/
├── frontend/
│   └── Dockerfile
├── backend/
│   └── Dockerfile
├── db/
│   └── init.sql
├── uploads/                # 上傳檔案（持久化）
└── pgdata/                 # 資料庫資料（持久化）
```

---

## 安全注意事項

- [ ] DB port 只綁 `127.0.0.1`，不對外開放
- [ ] `.env` 不進 git（加到 `.gitignore`）
- [ ] `.env.example` 不包含真實密碼
- [ ] uploads 目錄設定適當權限
- [ ] 使用 HTTPS（建議搭配 reverse proxy）

---

## 品質檢查

生成完成後確認：

- [ ] `docker compose up -d --build` 可以一次成功
- [ ] 所有環境變數都從 `.env` 讀取，無硬編碼
- [ ] 多階段建構，Runtime image 不含編譯工具
- [ ] DB 資料和上傳檔案有 Volume 持久化
- [ ] build.sh 可以一鍵更新
- [ ] DEPLOYMENT.md 足夠讓新手跟著操作

---

## 使用方式

```
請讀取 docker-deploy/SKILL.md，然後分析我的專案：

前端：{前端路徑}（Next.js / React / Vue）
後端：{後端路徑}（Spring Boot / Express / Django）
資料庫：PostgreSQL / MySQL / MongoDB
時區：Asia/Taipei
部署目標：{伺服器 OS}

請生成完整的 Docker 部署方案。
```

AI 會自動偵測框架 → 生成 Dockerfile → docker-compose.yml → 部署腳本 → 文件。
