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

#### Create and run your migration & Generate Prisma Client
```bash
npx prisma migrate dev --name init
```
```bash
npx prisma generate
```
<img width="1350" height="351" alt="image" src="https://github.com/user-attachments/assets/bb6a3b9d-7827-406c-9d79-cf6adf144d22" />

---


#### Create module, service & resolver
```bash 
nest g module prisma
```
```bash
nest g service prisma
```
```bash
nest g module books
```
```bash
nest g service books
```
```bash
nest g resolver books
```
---


>#### Create koro- books/dto/create-book.input.ts, books/dto/update-book.input.ts & books/model/book.model.ts
<img width="240" height="242" alt="image" src="https://github.com/user-attachments/assets/c27a1fa3-ce64-4ea1-88d6-ea12789c0575" />

#### `prisma.service.ts`
```bash
import { Injectable, OnModuleDestroy, OnModuleInit } from '@nestjs/common';
import { PrismaClient } from 'generated/prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy {
    async onModuleInit() {
        await this.$connect();
    }

    async onModuleDestroy() {
        await this.$disconnect();
    }
}
```
---


#### ``
```bash

```
---


#### ``
```bash

```
---
