---
title: typeORM
date: 2022-03-08 18:01:01
categories:
  - database
tags:
---

### 介绍

1. TypeORM 是一个 ORM(object-relational mapping)框架,可以在能执行 js 或者 ts 的环境中使用
2. 设计目标：让你可以用最少的 js 代码，完成各种数据库操作
3. TypeORM 同时支持 Active Record 模式和 Data Mapper 模式

#### [介绍 Active Record 和 Data Mapper](https://typeorm.io/active-record-data-mapper)

1. active record
   - 具体概念建议百度
   - 通过模型访问数据库，在模型中定义所有的查询方法
   - 需要继承 BaseEntity
2. Data Mapper
   - 数据模型只定义属性，不提供数据库操作
   - 不需要继承 BaseEntity
   - 自己定义单独的 class，一般称为“repositories”
3. 举个栗子，假设你有一个用户 model，你想根据 firstName+lastName 查询一个人

   - Active Record 模式

   ```typescript
   import { BaseEntity, Entity, PrimaryGeneratedColumn, Column } from "typeorm";

   @Entity()
   export class User extends BaseEntity {
     @PrimaryGeneratedColumn()
     id: number;

     @Column()
     firstName: string;

     @Column()
     lastName: string;

     @Column()
     isActive: boolean;

     static findByName(firstName: string, lastName: string) {
       return this.createQueryBuilder("user")
         .where("user.firstName = :firstName", { firstName })
         .andWhere("user.lastName = :lastName", { lastName })
         .getMany();
     }
   }

   // 查询
   const timber = await User.findByName("Timber", "Saw");
   ```

   - data mapping 模式

   ```typescript
   import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

   @Entity()
   export class User {
     @PrimaryGeneratedColumn()
     id: number;

     @Column()
     firstName: string;

     @Column()
     lastName: string;

     @Column()
     isActive: boolean;
   }

   // 查询
   const timber = await userRepository.findOneBy({
     firstName: "Timber",
     lastName: "Saw",
   });
   ```

### 关联关系

#### options

- eager: boolean, 设置为 true 时，当主 entity 查询时，会返回关联数据
- cascade: boolean | ("insert" | "update")[] , 新增或者更新 entity 时，会同步保存关联关系
- onDelete: "RESTRICT"|"CASCADE"|"SET NULL", 定义删除时，外健对应的行为
- nullable: boolean，关联关系是否允许为空
- orphanedRowAction: "nullify" | "delete" | "soft-delete", 删除子数据时，子数据的存储方式

#### JoinColumn

- 在当前表中加列名
- options

  - name：指定关联关系列名，默认是 id
  - referencedColumnName，指定关联表里的列名

  ```typescript
  @ManyToOne(type => Category)
  @JoinColumn([
      { name: "category_id", referencedColumnName: "id" },
      { name: "locale_id", referencedColumnName: "locale_id" }
  ])
  category: Category;
  ```

#### JoinTable

- 用在 many-to-many 中，创建关联关系表
- options

  - name，tableName
  - joinColumn
    - name
    - referencedColumnName
  - inverseJoinColumn

    - name
    - referencedColumnName

    ```typescript
    @ManyToMany(type => Category)
    @JoinTable({
        name: "question_categories", // table name for the junction table of this relation
        joinColumn: {
            name: "question",
            referencedColumnName: "id"
        },
        inverseJoinColumn: {
            name: "category",
            referencedColumnName: "id"
        }
    })
    categories: Category[];
    ```

#### one-to-one

- 需要使用 JoinColumn，指定关联关系

  ```typescript
  import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

  @Entity()
  export class Profile {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    gender: string;

    @Column()
    photo: string;
  }

  import {
    Entity,
    PrimaryGeneratedColumn,
    Column,
    OneToOne,
    JoinColumn,
  } from "typeorm";
  import { Profile } from "./Profile";

  @Entity()
  export class User {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @OneToOne(() => Profile)
    @JoinColumn()
    profile: Profile;
  }
  ```

#### Many-to-one / one-to-many

- 可以省略 JoinColumn，会自动在 many-to-one entity 中保存关联关系

  ```typescript
  import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from "typeorm";
  import { User } from "./User";

  @Entity()
  export class Photo {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    url: string;

    @ManyToOne(() => User, (user) => user.photos)
    user: User;
  }
  import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from "typeorm";
  import { Photo } from "./Photo";

  @Entity()
  export class User {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @OneToMany(() => Photo, (photo) => photo.user)
    photos: Photo[];
  }
  ```

#### many-to-many

- 需要使用 JoinTable 指定关联关系

  ```typescript
  import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

  @Entity()
  export class Category {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;
  }

  import {
    Entity,
    PrimaryGeneratedColumn,
    Column,
    ManyToMany,
    JoinTable,
  } from "typeorm";
  import { Category } from "./Category";

  @Entity()
  export class Question {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @Column()
    text: string;

    @ManyToMany(() => Category)
    @JoinTable()
    categories: Category[];
  }
  ```

#### Enabling logging

- 打印 sql 日志

  ```typescript
  {
      name: "mysql",
      type: "mysql",
      host: "localhost",
      port: 3306,
      username: "test",
      password: "test",
      database: "test",
      logging: true
  }
  ```

### 相关链接

[官方文档](https://typeorm.io/)
[gitBook](https://orkhan.gitbook.io/typeorm/)
