# db-migrate-startkit

## 事前準備

下載安裝 Docker Desktop

- [Windows 版](https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe?utm_source=docker&utm_medium=webreferral&utm_campaign=dd-smartbutton&utm_location=module&_gl=1*1x0tato*_gcl_au*NDk4NzQwNjM1LjE3MjczMzQ0NDY.*_ga*MTkxMzI2NzM5NC4xNjU5OTM4NTcy*_ga_XJWPQMJYHQ*MTcyNzMzMzcxNy4xNTYuMS4xNzI3MzM0NDY4LjM3LjAuMA..)
- Mac 版
  - [Intel 處理器](https://desktop.docker.com/mac/main/amd64/Docker.dmg?utm_source=docker&utm_medium=webreferral&utm_campaign=dd-smartbutton&utm_location=module&_gl=1*sjadaf*_gcl_au*NDk4NzQwNjM1LjE3MjczMzQ0NDY.*_ga*MTkxMzI2NzM5NC4xNjU5OTM4NTcy*_ga_XJWPQMJYHQ*MTcyNzMzMzcxNy4xNTYuMS4xNzI3MzM0NDY4LjM3LjAuMA..)
  - [M 系列處理器](https://desktop.docker.com/mac/main/arm64/Docker.dmg?utm_source=docker&utm_medium=webreferral&utm_campaign=dd-smartbutton&utm_location=module&_gl=1*sjadaf*_gcl_au*NDk4NzQwNjM1LjE3MjczMzQ0NDY.*_ga*MTkxMzI2NzM5NC4xNjU5OTM4NTcy*_ga_XJWPQMJYHQ*MTcyNzMzMzcxNy4xNTYuMS4xNzI3MzM0NDY4LjM3LjAuMA..)

下載並安裝 Node.js

- [下載連結](https://nodejs.org/zh-tw)

下載並安裝 Dbeaver Community

- [Windows 版](https://dbeaver.io/files/dbeaver-ce-latest-x86_64-setup.exe)
- Mac 版
  - [Intel 處理器](https://dbeaver.io/files/dbeaver-ce-latest-macos-x86_64.dmg)
  - [M 系列處理器](https://dbeaver.io/files/dbeaver-ce-latest-macos-aarch64.dmg)

## 開始使用

### 安裝套件

> 執行前請確保已經安裝了 Node.js

打開終端機並輸入以下指令

```
$ npm i
```

### 首次在本機開啟資料庫並執行遷移

> 請確保 Docker Desktop 已經正常啟動，並能夠看到容器頁面(Containers)

打開終端機並輸入以下指令，程式將自動啟動資料庫，並運行遷移

```
$ npm run start
```

啟動後，請檢查 Docker Desktop 程式界面中裡是否新增 db-migrate-startkit 的容器堆疊，該堆疊裡了有兩個容器：

- db_migrate-1：執行遷移檔案的容器，執行後會自動關閉（狀態（STATUS）為 Exited）。請務必進入檢查查看遷移的執行結果，確保遷移執行成功。
- postgres-1：資料庫容器，狀態應為 Running

### 重新啟動資料庫

```
$ npm run restart
```

### 關閉資料庫

```
$ npm run stop
```

### 關閉資料庫並清除資料

```
$ npm run clean
```

### DB Create 指令

```sql
任務二-資料庫建立指令
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE TABLE "USER" (
  "id" uuid PRIMARY KEY NOT NULL DEFAULT (gen_random_uuid()),
  "name" varchar(50) NOT NULL,
  "email" varchar(320) UNIQUE NOT NULL,
  "role" varchar(20) NOT NULL,
  "created_at" timestamp NOT NULL DEFAULT (CURRENT_TIMESTAMP),
  "updated_at" timestamp NOT NULL DEFAULT (CURRENT_TIMESTAMP)
);
CREATE TABLE "SKILL" (
  "id" uuid PRIMARY KEY DEFAULT uuid_generate_v4(),
  "name" varchar(50) UNIQUE NOT NULL,
  "created_at" timestamp NOT NULL DEFAULT (CURRENT_TIMESTAMP)
);

INSERT INTO "SKILL" (name) VALUES ('重訓'), ('瑜伽'), ('有氧運動'), ('復健訓練');

CREATE TABLE "COACH" (
  "id" uuid PRIMARY KEY NOT NULL DEFAULT uuid_generate_v4(),
  "user_id" uuid NOT NULL REFERENCES "USER"(id),
  "experience_years" integer,
  "description" text,
  "profile_image_url" varchar(2048),
  "created_at" timestamp NOT NULL DEFAULT (CURRENT_TIMESTAMP),
  "updated_at" timestamp NOT NULL DEFAULT (CURRENT_TIMESTAMP),
  UNIQUE("user_id")
);

CREATE TABLE "COACH_LINK_SKILL" (
  "coach_id" uuid NOT NULL REFERENCES "COACH"(id),
  "skill_id" uuid NOT NULL REFERENCES "SKILL"(id),
  "created_at" timestamp NOT NULL DEFAULT (CURRENT_TIMESTAMP),
  PRIMARY KEY ("coach_id", "skill_id")
);

CREATE TABLE "CREDIT_PACKAGE" (
  "id" serial PRIMARY KEY,
  "name" varchar(50) NOT NULL,
  "credit_amount" integer NOT NULL,
  "price" numeric(10,2) NOT NULL,
  "created_at" timestamp NOT NULL DEFAULT (CURRENT_TIMESTAMP)
);

CREATE TABLE "CREDIT_PURCHASE" (
  "id" uuid PRIMARY KEY NOT NULL DEFAULT (gen_random_uuid()),
  "user_id" uuid NOT NULL REFERENCES "USER"(id),
  "credit_package_id" integer NOT NULL REFERENCES "CREDIT_PACKAGE"(id),
  "purchased_credits" integer NOT NULL,
  "price_paid" numeric(10,2) NOT NULL,
  "created_at" timestamp NOT NULL DEFAULT (CURRENT_TIMESTAMP),
  "purchase_at" timestamp NOT NULL DEFAULT (CURRENT_TIMESTAMP)
);

CREATE TABLE "COURSE" (
  "id" serial PRIMARY KEY,
  "user_id" uuid NOT NULL REFERENCES "USER"(id),
  "skill_id" uuid NOT NULL REFERENCES "SKILL"(id),
  "name" varchar(100) NOT NULL,
  "description" text,
  "start_at" timestamp NOT NULL,
  "end_at" timestamp NOT NULL,
  "max_participants" integer NOT NULL,
  "meeting_url" varchar(2048) NOT NULL,
  "created_at" timestamp NOT NULL DEFAULT (CURRENT_TIMESTAMP)
);

CREATE TABLE "COURSE_BOOKING" (
  "id" uuid PRIMARY KEY NOT NULL DEFAULT (gen_random_uuid()),
  "user_id" uuid NOT NULL REFERENCES "USER"(id),
  "course_id" integer NOT NULL REFERENCES "COURSE"(id),
  "booking_at" timestamp NOT NULL,
  "status" varchar(20) NOT NULL,
  "join_at" timestamp,
  "leave_at" timestamp,
  "cancelled_at" timestamp,
  "cancellation_reason" varchar(255),
  "created_at" timestamp NOT NULL DEFAULT (CURRENT_TIMESTAMP)
);

CREATE TABLE "BLOG_POST" (
  "id" uuid PRIMARY KEY NOT NULL DEFAULT (gen_random_uuid()),
  "user_id" uuid NOT NULL REFERENCES "USER"(id),
  "title" varchar(255) NOT NULL,
  "content" text NOT NULL,
  "featured_image_url" varchar(2048),
  "category" varchar(20) NOT NULL,
  "spend_minutes" smallint NOT NULL,
  "created_at" timestamp NOT NULL DEFAULT (CURRENT_TIMESTAMP),
  "updated_at" timestamp NOT NULL DEFAULT (CURRENT_TIMESTAMP)
);

CREATE TABLE "COMMENT" (
  "id" uuid PRIMARY KEY NOT NULL DEFAULT (gen_random_uuid()),
  "blog_post_id" uuid NOT NULL REFERENCES "BLOG_POST"(id),
  "user_id" uuid NOT NULL REFERENCES "USER"(id),
  "content" text NOT NULL,
  "created_at" timestamp NOT NULL DEFAULT (CURRENT_TIMESTAMP),
  "updated_at" timestamp NOT NULL DEFAULT (CURRENT_TIMESTAMP)
);
```
