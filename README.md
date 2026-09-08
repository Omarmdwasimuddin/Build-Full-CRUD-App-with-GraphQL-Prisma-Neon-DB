## Build Full CRUD App with GraphQL, Prisma & Neon DB

#### GraphQL installation
```bash
npm i @nestjs/graphql @nestjs/apollo @apollo/server @as-integrations/express5 graphql
```
---

#### Prisma Installation
```bash
npm install prisma --save-dev
```
```bash
npm install @prisma/client @prisma/adapter-pg pg
```
```bash
npx prisma init
```
---

>#### Neon e project create koro and then database connect koro-
<img width="1597" height="762" alt="image" src="https://github.com/user-attachments/assets/ec4be945-6633-4d24-9e3d-aad07c5b4c7c" />

#### `.env`
```bash
DATABASE_URL="postgresql://user:password@ep-crimson-river-a1jhbuwa-pooler.ap-southeast-1.aws.neon.tech/neondb?sslmode=require&channel_binding=require"
```
---


#### `schema.prisma`
```bash
generator client {
  provider = "prisma-client"
  output   = "../generated/prisma"
}

datasource db {
  provider = "postgresql"
}


model Book {
  id String @id @default(uuid())
  title String
  author String
  createdAt DateTime @default(now())
}
```
---

#### ``
```bash

```
---
