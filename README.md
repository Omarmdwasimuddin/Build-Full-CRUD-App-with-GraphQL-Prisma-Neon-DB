## Build Full CRUD App with GraphQL, Prisma & Neon DB

[Connect NestJ with Prisma and Neon (Prisma v7.10.0)](https://github.com/Omarmdwasimuddin/Connect-NestJS-with-Prisma-and-Neon)

#### GraphQL installation
```bash
npm i @nestjs/graphql@^13 @nestjs/apollo@13.1.0 @apollo/server@^4.10.0 @as-integrations/express5 graphql
```
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
import { PrismaPg } from '@prisma/adapter-pg';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy {
  constructor() {
    const adapter = new PrismaPg({
      connectionString: process.env.DATABASE_URL!,
    });

    super({
      adapter,
    });
  }

  async onModuleInit() {
    await this.$connect();
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```
---

>#### app.module.ts e add koro-
<img width="1077" height="238" alt="image" src="https://github.com/user-attachments/assets/29e28f89-ad93-4bb7-b2a6-381e2bf49bd5" />

#### `app.module.ts`
```bash
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { PrismaModule } from './prisma/prisma.module';
import { BooksModule } from './books/books.module';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { join } from 'path';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [ ConfigModule.forRoot({ isGlobal: true }), GraphQLModule.forRoot<ApolloDriverConfig>({
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
---


#### `create-book.input.ts`
```bash
import { InputType, Field } from "@nestjs/graphql";
@InputType()
export class CreateBookInput {
    @Field()
    title!: string;

    @Field()
    author!: string;
}
```
---


#### `update-book.input.ts`
```bash
import { InputType, Field, PartialType } from "@nestjs/graphql";
import { CreateBookInput } from "./create-book.input";
@InputType()
export class UpdateBookInput extends PartialType(CreateBookInput) {
    @Field()
    id!: string;
}
```
---

>#### prisma.module.ts file e add koro- exports: [PrismaService],

#### `prisma.module.ts`
```bash
import { Module } from '@nestjs/common';
import { PrismaService } from './prisma.service';

@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```
---

>#### books.module.ts e add koro- imports: [PrismaModule],

#### `books.module.ts`
```bash
import { Module } from '@nestjs/common';
import { BooksService } from './books.service';
import { BooksResolver } from './books.resolver';
import { PrismaModule } from 'src/prisma/prisma.module';

@Module({
  imports: [PrismaModule],
  providers: [BooksService, BooksResolver]
})
export class BooksModule {}
```
---


#### `book.model.ts`
```bash
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
---

#### `books.service.ts`
```bash
import { Injectable } from '@nestjs/common';
import { PrismaService } from 'src/prisma/prisma.service';
import { CreateBookInput } from './dto/create-book.input';
import { UpdateBookInput } from './dto/update-book.input';

@Injectable()
export class BooksService {
    constructor(private prisma: PrismaService) {}

    create(data: CreateBookInput) {
        return this.prisma.book.create({ data });
    }

    findAll() {
        return this.prisma.book.findMany();
    }

    findOne(id: string) {
        return this.prisma.book.findUnique({
            where: { id }
        })
    }

    update(data: UpdateBookInput) {
        return this.prisma.book.update({
            where: { id: data.id },
            data: {
                title: data.title,
                author: data.author
            }
        })
    }

    remove(id: string) {
        return this.prisma.book.delete({
            where: { id }
        })
    }
}
```
---


#### `books.resolver.ts`
```bash
import { Args, Mutation, Query, Resolver } from '@nestjs/graphql';
import { Book } from './model/book.model';
import { BooksService } from './books.service';
import { CreateBookInput } from './dto/create-book.input';
import { UpdateBookInput } from './dto/update-book.input';

@Resolver(() => Book)
export class BooksResolver {
    constructor(private readonly booksService: BooksService) {}

    // Define your GraphQL queries and mutations here

    @Query(() => [Book])
    getAllBooks() {
        return this.booksService.findAll();
    }

    @Query(() => Book)
    getBookById(@Args('id')id: string) {
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
---

>#### localhost:3000/graphql

```bash
 mutation{
 createBook(input:{
     title: "PrismaORM for delete",
     author: "Wasim Uddin"
   }){
     id
     title
   }
 }
```
```bash
 query {
   getAllBooks {
     id
     title
     author
   }
 }
```
```bash
 mutation {
   updateBook(input:{
     id:"9ef7090b-28b3-40ff-89d7-0123d45ba639"
     title: "PrismaORM Updated",
     author: "Wasim Updated author"
   }){
     title
   }
 }
```
```bash
mutation {
  deleteBook(id: "dd78234b-2241-4ce3-ab97-f670fce9096a"){
  title
  }
}
```
```bash
query{
  getBookById(id:"a36953c8-f6b1-46b5-bac1-34c0fc0ecfa5"){
    title
    author
  }
}
```
---


>## OUTPUT
<img width="1599" height="765" alt="image" src="https://github.com/user-attachments/assets/41fab662-087f-491c-be61-7b9715455f2a" />

<img width="1350" height="341" alt="image" src="https://github.com/user-attachments/assets/927c2d05-738c-4f90-b408-ab430cb17105" />

---
