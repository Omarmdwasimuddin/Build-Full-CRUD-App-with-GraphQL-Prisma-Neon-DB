# Full CRUD App with GraphQL, Prisma & Neon DB 

আগের গাইডে NestJS + Prisma + Neon connect করা হয়েছিল। এবার সেই connection-এর উপর দাঁড়িয়ে **GraphQL** দিয়ে একটা পুরোপুরি কাজ করা CRUD app বানানো হবে — `Book` resource দিয়ে। এটা **Prisma v8**-এর নতুন ORM style ব্যবহার করে, যেটাতে module system পুরোপুরি **ESM (ECMAScript Modules)**-ভিত্তিক।

> এই গাইড শুরুর আগে Prisma + Neon connect করা থাকতে হবে: [Connect NestJS with Prisma and Neon (Prisma v8)](https://github.com/Omarmdwasimuddin/Connect-NestJ-with-Prisma-and-Neon-Prisma-v8-)

---

## ধাপ ১: GraphQL Package Install করা

```bash
npm i @nestjs/graphql@^13 @nestjs/apollo@13.1.0 @apollo/server@^4.10.0 @as-integrations/express5 graphql
```

---

## ধাপ ২: Module, Service, Resolver তৈরি করা

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

এরপর manually এই file গুলো বানাতে হবে:
- `books/dto/create-book.input.ts`
- `books/dto/update-book.input.ts`
- `books/model/book.model.ts`

![Folder structure](https://github.com/user-attachments/assets/c27a1fa3-ce64-4ea1-88d6-ea12789c0575)

---

## ধাপ ৩: Prisma Service লেখা

### `prisma.service.ts`

```ts
import { Injectable } from '@nestjs/common';
import { db } from './db.js';

@Injectable()
export class PrismaService {
    get client() {
        return db;
    }
}
```

এখানে `db.js` — এটা Prisma v8-এর নতুন client generator থেকে automatic generate হওয়া file (আগের গাইড অনুযায়ী schema থেকে জেনারেট হয়েছে)। `PrismaService`-এ একটা `client` নামের getter বানানো হয়েছে, যেটা দিয়ে অন্য service-গুলো এই generated Prisma client ব্যবহার করতে পারবে।

---

## ধাপ ৪: `app.module.ts` Setup করা

![app.module.ts এ যা যোগ করতে হবে](https://github.com/user-attachments/assets/29e28f89-ad93-4bb7-b2a6-381e2bf49bd5)

### `app.module.ts`

```ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller.js';
import { AppService } from './app.service.js';
import { ConfigModule } from '@nestjs/config';
import { PrismaModule } from './prisma/prisma.module.js';
import { BooksModule } from './books/books.module.js';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { join } from 'path';

@Module({
  imports: [ConfigModule.forRoot({ isGlobal: true }), GraphQLModule.forRoot<ApolloDriverConfig>({
    driver: ApolloDriver,
    autoSchemaFile: join(process.cwd(), 'src/schema.gql'),
    sortSchema: true,
    playground: true,
  }), PrismaModule, BooksModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

> লক্ষ্য করো — এখানে সব import-এর শেষে `.js` extension আছে (যেমন `'./app.controller.js'`)। এর কারণ নিচে **ESM Setup** অংশে বিস্তারিত ব্যাখ্যা করা হয়েছে।

---

## ধাপ ৫: DTO (Input Type) লেখা

### `create-book.input.ts`

```ts
import { InputType, Field } from "@nestjs/graphql";

@InputType()
export class CreateBookInput {
    @Field()
    title!: string;

    @Field()
    author!: string;
}
```

### `update-book.input.ts`

```ts
import { InputType, Field, PartialType } from "@nestjs/graphql";
import { CreateBookInput } from "./create-book.input.js";

@InputType()
export class UpdateBookInput extends PartialType(CreateBookInput) {
    @Field()
    id!: string;
}
```

আগের GraphQL গাইডের মতোই — `PartialType(CreateBookInput)` দিয়ে সবগুলো field optional বানানো হচ্ছে, আর `id` আলাদাভাবে যোগ করা হচ্ছে (update করার জন্য কোনটা target সেটা বলতে)।

---

## ধাপ ৬: Prisma Module Setup করা

> **নোট:** `prisma.module.ts`-এ `exports: [PrismaService]` যোগ করতে হবে, যাতে অন্য module-এ (এখানে `BooksModule`) এই service ব্যবহার করা যায়।

### `prisma.module.ts`

```ts
import { Module } from '@nestjs/common';
import { PrismaService } from './prisma.service.js';

@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

---

## ধাপ ৭: Books Module Setup করা

> **নোট:** `books.module.ts`-এ `imports: [PrismaModule]` যোগ করতে হবে, যাতে `BooksService`-এ `PrismaService` inject করা যায়।

### `books.module.ts`

```ts
import { Module } from '@nestjs/common';
import { BooksService } from './books.service.js';
import { BooksResolver } from './books.resolver.js';
import { PrismaModule } from '../prisma/prisma.module.js';

@Module({
  imports: [PrismaModule],
  providers: [BooksService, BooksResolver]
})
export class BooksModule {}
```

---

## ধাপ ৮: GraphQL Model লেখা

### `book.model.ts`

```ts
import { ObjectType, Field } from "@nestjs/graphql";

@ObjectType()
export class Book {
    @Field()
    id!: string;

    @Field()
    title!: string;

    @Field()
    author!: string;

    @Field()
    createdAt!: Date;
}
```

এখানে আগের গাইডের মতো `@Schema()` (Mongoose-এর জন্য) নেই — কারণ এখানে database schema টা Prisma-এর `schema.prisma` file-এই define করা আছে (আগের Prisma গাইডে দেখানো হয়েছিল)। এই `book.model.ts`-এ শুধু **GraphQL type** define করা হচ্ছে, যেটা `Book` data-কে GraphQL response-এ কেমন দেখাবে সেটা বলে দেয়।

---

## ধাপ ৯: Service লেখা (Prisma v8-এর নতুন ORM Style)

### `books.service.ts`

```ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service.js';
import { CreateBookInput } from './dto/create-book.input.js';
import { UpdateBookInput } from './dto/update-book.input.js';

@Injectable()
export class BooksService {
    constructor(private prisma: PrismaService) {}

    async create(data: CreateBookInput) {
        return this.prisma.client.orm.public.Book.create(data);
    }

    async findAll() {
        return this.prisma.client.orm.public.Book.all();
    }

    async findOne(id: string) {
        return this.prisma.client.orm.public.Book.where({ id }).first();
    }

    async update(data: UpdateBookInput) {
        const { id, ...updateData } = data;
        return this.prisma.client.orm.public.Book.where({ id }).update(updateData);
    }

    async remove(id: string) {
        return this.prisma.client.orm.public.Book.where({ id }).delete();
    }
}
```

### লক্ষ্য করার বিষয়: এটা Prisma-এর নতুন Query Builder Style

আগের Prisma version-গুলোতে সাধারণত এভাবে লেখা হতো: `prisma.book.create({ data })`, `prisma.book.findMany()` ইত্যাদি। কিন্তু এখানে দেখা যাচ্ছে **`this.prisma.client.orm.public.Book.create(data)`** — এটা Prisma v8-এর নতুন **ORM-style query builder**, যেখানে:
- `client.orm` — নতুন ORM interface-এ ঢোকা হচ্ছে
- `.public` — PostgreSQL-এর `public` schema (database-এ যদি একাধিক schema থাকে, তাহলে schema-র নাম দিয়ে আলাদা করা হয়)
- `.Book` — model-এর নাম
- `.create()`, `.all()`, `.where().first()`, `.where().update()`, `.where().delete()` — chainable, fluent-style query method

### `update()` method-এ একটা ভালো practice

```ts
const { id, ...updateData } = data;
```

এখানে `data`-থেকে `id` আলাদা করে বাদ দেওয়া হচ্ছে (destructuring দিয়ে), কারণ `id` update করার জন্য না, বরং **কোনটা খুঁজে বের করতে হবে** সেটা বলতে ব্যবহার হচ্ছে (`where({ id })`-এ)। বাকি field (`updateData`) দিয়েই actual update হচ্ছে। এটা আগের GraphQL/MongoDB গাইডগুলোতে যে সমস্যাটা হতে পারত (`id` field ভুলবশত data-এর ভিতরেও থেকে যাওয়া), সেটা এড়ানোর সঠিক পদ্ধতি।

---

## ধাপ ১০: Resolver লেখা

### `books.resolver.ts`

```ts
import { Args, Mutation, Query, Resolver } from '@nestjs/graphql';
import { BooksService } from './books.service.js'
import { Book } from './model/book.model.js';
import { CreateBookInput } from './dto/create-book.input.js';
import { UpdateBookInput } from './dto/update-book.input.js';

@Resolver()
export class BooksResolver {
    constructor(private readonly booksService: BooksService) {}

    @Query(() => [Book])
    getAllBooks() {
        return this.booksService.findAll();
    }

    @Query(() => Book)
    getBookById(@Args('id') id: string) {
        return this.booksService.findOne(id);
    }

    @Mutation(() => Book)
    createBook(@Args('input') input: CreateBookInput) {
        return this.booksService.create(input);
    }

    @Mutation(() => Book)
    updateBook(@Args('input') input: UpdateBookInput) {
        return this.booksService.update(input);
    }

    @Mutation(() => Book)
    deleteBook(@Args('id') id: string) {
        return this.booksService.remove(id);
    }
}
```

আগের GraphQL গাইডের মতোই structure — শুধু query/mutation-এর নাম আলাদা (`getAllBooks`, `getBookById`, `createBook`, `updateBook`, `deleteBook`), আর data আসছে নতুন Prisma ORM style থেকে।

---

## ধাপ ১১: ESM (ECMAScript Modules) Setup করা — গুরুত্বপূর্ণ

এই project-টা পুরোপুরি **ESM** module system ব্যবহার করছে (traditional CommonJS-এর বদলে)। এর জন্য কয়েকটা জিনিস ঠিকভাবে setup করতে হবে।

### `package.json`

`package.json`-এ যোগ করতে হবে:

```json
"type": "module"
```

এটা Node.js-কে বলে দেয় যে এই project-এর সব `.js` file ES Module হিসেবে treat করতে হবে (CommonJS `require()` না, বরং `import`/`export`)।

### `tsconfig.json`

```json
{
  "compilerOptions": {
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "resolvePackageJsonExports": true,
    "esModuleInterop": true,
    "isolatedModules": true,
    "declaration": true,
    "removeComments": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "allowSyntheticDefaultImports": true,
    "target": "ES2022",
    "sourceMap": true,
    "outDir": "./dist",
    "baseUrl": "./",
    "incremental": true,
    "skipLibCheck": true,
    "strictNullChecks": true,
    "forceConsistentCasingInFileNames": true,
    "noImplicitAny": false,
    "strictBindCallApply": false,
    "noFallthroughCasesInSwitch": false,
    "resolveJsonModule": true,
    "types": [
      "node"
    ]
  }
}
```

**সবচেয়ে গুরুত্বপূর্ণ দুইটা field:**
- `"module": "NodeNext"` — TypeScript-কে বলছে output code Node.js-এর নতুন ESM/CommonJS হাইব্রিড resolution rule মেনে জেনারেট করতে
- `"moduleResolution": "NodeNext"` — import path resolve করার নিয়মও একই standard মেনে চলবে

### `main.ts`-এ Temporal Polyfill

```ts
import { Temporal } from '@js-temporal/polyfill';
(globalThis as any).Temporal = Temporal;
```

এটা `main.ts`-এর **সবার আগে** বসাতে হবে (অন্য যেকোনো import-এর আগে)। `Temporal` হলো JavaScript-এর একটা নতুন (এখনো experimental/stage-3) date/time API, যেটা Prisma v8-এর নতুন client এর ভিতরে ব্যবহার হয়। যেহেতু এটা এখনো সব Node.js version-এ native ভাবে available না, তাই `@js-temporal/polyfill` দিয়ে `globalThis.Temporal`-এ manually বসিয়ে দেওয়া হচ্ছে, যাতে Prisma client ঠিকভাবে কাজ করে।

### `main.ts`

```ts
import { Temporal } from '@js-temporal/polyfill';
(globalThis as any).Temporal = Temporal;

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module.js';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

### `app.controller.ts`-এও `.js` extension

```ts
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service.js';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

> ## 🔑 সবচেয়ে গুরুত্বপূর্ণ নিয়ম
> **যেকোনো file import করার সময়, file-এর নামের শেষে `.js` extension যোগ করতে হবে** — এমনকি যদি actual file-টা `.ts` হয়ও। এটা ESM module resolution-এর একটা নিয়ম: Node.js-এর native ESM resolver relative import-এ পুরো file extension আশা করে, আর যেহেতু TypeScript compile হয়ে `.ts` থেকে `.js` হয়ে যায়, তাই import statement-এ `.js` লিখতে হয় (compile হওয়ার পরের extension অনুযায়ী), `.ts` না।

---

## ধাপ ১২: GraphQL Playground-এ Test করা

`localhost:3000/graphql`-এ গিয়ে:

### Create

```graphql
mutation {
  createBook(input: {
    title: "PrismaORM for delete",
    author: "Wasim Uddin"
  }) {
    id
    title
  }
}
```

### সবগুলো দেখা

```graphql
query {
  getAllBooks {
    id
    title
    author
  }
}
```

### Update

```graphql
mutation {
  updateBook(input: {
    id: "9ef7090b-28b3-40ff-89d7-0123d45ba639"
    title: "PrismaORM Updated",
    author: "Wasim Updated author"
  }) {
    title
  }
}
```

### Delete

```graphql
mutation {
  deleteBook(id: "dd78234b-2241-4ce3-ab97-f670fce9096a") {
    title
  }
}
```

### নির্দিষ্ট একটা দেখা

```graphql
query {
  getBookById(id: "a36953c8-f6b1-46b5-bac1-34c0fc0ecfa5") {
    title
    author
  }
}
```

---

## Output (উদাহরণ)

![GraphQL Playground output ১](https://github.com/user-attachments/assets/41fab662-087f-491c-be61-7b9715455f2a)

![GraphQL Playground output ২](https://github.com/user-attachments/assets/927c2d05-738c-4f90-b408-ab430cb17105)

---
