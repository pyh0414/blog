---
title: '[Prisma2] model로 시작하는 Prisma'
date: 2021-02-12 21:02:70
category: prisma2
thumbnail: { thumbnailSrc }
draft: false
---

⚠️ 이번 장에서는 Prisma 환경세팅은 다루지 않기 때문에 Prisma를 설치하지 않으신 분은 [여기](https://www.prisma.io/docs/getting-started)를 참고하세요.

```graphql
generator client {
  provider = "prisma-client-js"
}

datasource mysql {
  provider = "mysql"
  url      = env("API_MYSQL_URL")
}

model Post {
  id         Int         @id @default(autoincrement())
  createdAt  DateTime    @default(now())
  title      String
  published  Boolean     @default(false)
  author     User        @relation(fields: [authorId], references: [id])
  authorId   Int
  categories Category[]  @relation(references: [id])

...

}
```

Prisma를 사용하기 위해서는 일단 model을 만들어야 합니다. 그래야 이를 바탕으로 DB schema, type도 생성하기 때문입니다. model을 생성하는데 알아야 하는 다양한 옵션이 있습니다. 이런 옵션들을 제대로 알아야 원하는 DB schema를 정확하게 구현할 수 있습니다.

이번장에서는 model을 생성하는데 알아야 하는 것이 어떤 것들이 있는지 알아보겠습니다 ❗️

📌 : 꼭 필요한 option을 의미

# data source

연결하려는 데이터베이스에 대한 설정을 다루는 부분입니다.

```graphql
datasource db {
  provider = "mysql"
  url      = "mysql://johndoe:mypassword@localhost:3306/mydb""
}


```

- `provider` 📌

  어떤 DB를 사용할지 명시

- `url` 📌

  DB server의 URL을 명시, DB마다 선언 방법이 다름 몇몇 방법으로 선언할 수 있음

  연결에 대해 여러 옵션, SSL connetion 설정 가능[(여기 참고](https://www.prisma.io/docs/concepts/database-connectors/mysql#configuring-an-ssl-connection))

  - .env 파일로 설정`env("DATABASE_URL")` 가능

  - 경로를 문자열로 표시 가능

# generator

prisma generate시 생성되는 자원을 결정합니다.

```json
generator client {
  provider        = "prisma-client-js"
  output          ="./src/generated/client"
  binaryTargets   = ["linux-msl"]
  previewFeatures = ["sample-preview-feature"]
}
```

- `provider` 📌

  사용할 generator을 지정, generator을 구현하는 파일을 지정, 내장 generator을 직접 지정할 수 있습니다. 하나 또는 여러개를 가질 수 있으며 현재는 prisma-client-js만 사용할 수 있습니다.

- `output`

  prisma generate시 생성되는 @prisma/client의 위치를 기술합니다. default 위치는 **<u>node_modules/@prisma/client</u>** 입니다.

- `binaryTargets`

  query engine의 바이너리 호환성을 보장하기 위해 Prisma Client가 실행될 OS를 지정합니다. **"native"** 로 지정할 경우 Prisma가 [여기](https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference#binarytargets-options)를 고려하여 현재 OS에 알맞은 binary target을 찾은 후 지정합니다. 또한 [Custom binary](https://www.prisma.io/docs/concepts/overview/under-the-hood#using-custom-engine-binaries)를 사용할 수 있습니다.

- `output`

  현재 사용가능한 Preview features를 알아봅니다. 사용법은 [prisma release 2.16.0](https://github.com/prisma/prisma/releases/tag/2.16.0)을 보면 createMany api는 현재 preview 단계 이므로 createMany를 사용하기 위해서는 **previewFeatures = ["createMany"]**를 추가해주어야 합니다.

# model

Prisma model을 정의합니다. model은 User, Post...등 @prisma/client에서 사용하는 model 그리고 DB table을 의미합니다. model명은
model의 모든 데이터(row)는 고유하게 식별할 수 있어야 하며 model을 정의할 때 아래의 attributes중 하나를 반드시 사용해야 합니다.
&nbsp;

- @unique
- @@unique
- @id
- @@id

&nbsp;

model 명은 **[A-Za-z][a-za-z0-9_]\*** 모양의 정규식 형태를 가집니다. 또한 문자로 시작하며 **PascalCase** 로 표기됩니다(model attributes는 **camelCase**형태를 가집니다)

# model field scalar type

model을 정의할 때 사용할 수 있는 data type 단위로 실제 DB native data type으로 변환되며(String → varchar(191)) 아래의 scalar type을 지원합니다.

1. `String`
2. `Boolean`
3. `Int`
4. `Float`
5. `DateTim`
6. `Json`
7. `Decimal`
8. `Bytes`
9. `BigInt`

&nbsp;

추가적으로 model이 relation을 가지는 경우 <u>**model field type**</u>을 가질 수 있습니다.

2.11.0+ 부터는 확장된 native type을 지원하며 자세한 내용은 [여기](https://www.prisma.io/docs/concepts/components/preview-features/native-types)를 참고하세요.

# model field scalar modifiers

model field scalar type에는 2가지의 식별자를 추가할 수 있습니다.

`[]`

scalar type을 배열로 만드며 optional일 수 없습니다(Post[]? ❌). model type이 기본 scalar type은 해당 DB가 list를 지원해주어야만 사용 가능합니다. 현재 scalar list는 PostgreSQL만 지원합니다. MySQL와 SQLite 는 기본적으로 list를 지원하지 않습니다.

model간에 관계가 있을 때, 예를 들어 User와 Post의 관계가 1:N일 때 Post[]로 표현할 수 있으며 Post model에서는 userId를 정의해야 합니다.

&nbsp;

`?`

scalar type을 optional하게 만듭니다. 배열로 만드는 경우 ?를 사용할 수 없으며 ? 식별자를 붙인 scalar type은 **nullable** 특징을 가집니다. 아무것도 붙이지 않으면 **not null** 특징을 가집니다.

# model attributes

filed 또는 model의 option을 추가하며 여기에는 2가지 방법이 있습니다.

- @ : filed에 추가
- @@ : model에 추가

일부 속성은 arguments를 필요로 하며 arguments를 사용하는 경우 항상 이름은 지정해야 하지만 대부분의 경우 arguments를 생략할 수 있습니다

&nbsp;

`@id`

```graphql
@id
```

- model의 id filed를 지정
- @default()를 사용하여 기본값을 지정할 수 있음, `autoincrement()` `uuid()` `cuid()`
- optional일 수 없음
- String, Int, enum에 추가할 수 있음

&nbsp;

`@@id`

```graphql
@@id(fields: [title, author])
@@id([title, author])
```

- model에서 여러개의 id를 지정
- optional일 수 없음
- String, Int, enum에 추가할 수 있음

&nbsp;

❗️ N:M관계를 관리하는 hashTagsPosts model의 경우 **[hashTagId,postId]**를 사용하여 각각의 row를 유일하게 구분할 수 있기 때문에 이런 경우 @@id를 사용합니다.

&nbsp;

`@default`

```graphql
id Int @id @default(value: autoincrement())
id Int @id @default(autoincrement())
```

- scalar filed의 default값을 적용
- 4가지의 함수 사용가능
  - `autoincrement()`
  - `cuid()`
  - `uuid()`
  - `now()`

&nbsp;

`@unique`

```graphql
@unique
```

- scalar filed의 unique값을 적용
- 어떤 scalar field에도 사용 가능
- **<u>unique field는 자동으로 index가 됨</u>**
- **<u>model field type에는 사용할 수 없음</u>**
- unique filed는 optional 또는 required
- model은 여러개의 unique field를 가질 수 있음
- 모델에 @id, @@id가 없는 경우 unique field를 무조건 추가해야함

&nbsp;

`@@unique`

```graphql
@@unique(fields: [title, author])
@@unique([title, author])
@@unique(field: [title, author], name: "titleAuthor")
```

- model에 여러개의 field를 조합해서 하나의 unique를 만듦
- [title, author]로 unique를 만들었다면 title1, author1를 생성하고 또 title1, author1을 만들 수 없음. 대신 title1, author2는 만들 수 있음
- 생성된 unique는 다음과 같은 모양을 가짐

```
unique 명은 titleAuthor, UNIQUE KEY `titleAuthor` (`firstName`,`secondName`)
```

&nbsp;

`@@index`

```graphql
@@index(fields: [title, author])
@@index([title, author])
@@index(fields: [title, content], name: "main_index")
```

- model에 여러개의 index를 지정
- index

&nbsp;

`@@relation`

```graphql
@relation(name: "UserOnPost", fields:[authorId], references: [id])
@relation("UserOnPost", references: [id])
@relation(name: "UserOnPost")
@relation("UserOnPost")
```

- 관계에 대한 메타정보를 명시
- foreign key, References
- `name`은 relation을 구분, `fields`는 외래키 filed, `references`는 외래키가 참조하는 id

&nbsp;

❗️ name이 서로 동기화가 되지 않으면 N쪽의 테이블에서 새로운 field 만듦

```graphql
model User {
  id        Int      @id @default(autoincrement())
  posts     Post[] @relation(name:"Test1")
}

model Post {
  id         Int        @id @default(autoincrement())
  author    User        @relation(name:"Test2",fields:[authorId],references: [id])
  authorId  Int
}
```

위와 같은 구조에서 생성된 테이블은 새로운 filed와 constraint가 생성된다. 아마 name으로 relation을 구분하는데 동기화가 되어있지 않으니 Prisma가 Post쪽의 model 정보를 사용하여 관계를 임의로 만든 것 같다.

```
CONSTRAINT `post_ibfk_2` FOREIGN KEY (`userId`) REFERENCES `User` (`id`) ON DELETE SET NULL ON UPDATE CASCADE
```

&nbsp;

`@map`

```graphql
@map(name: "is_admin")
@map("users")
```

- DB상의 table column명을 변경

&nbsp;

`@@map`

```graphql
@@map(name: "users")
@@map("users")
```

- DB상의 table 명을 변경

&nbsp;

`@updatedAt`

```graphql
@updatedAt
```

- row가 마지막으로 업데이트 된 시간을 자동으로 저장
- scalar type이 DateTime인 field에 붙일 수 있음

&nbsp;

# attribute functions

`autoincrement()`

```graphql
id  Int  @id @default(autoincrement())
```

- 연속된 정수값을 가짐(1,2,3....)
- scalar type이 Int인 filed에 붙일 수 있음

&nbsp;

`cuid()`

```graphql
id  Int  @id @default(cuid())
```

- [cuid spec](https://github.com/ericelliott/cuid)에 따라 고유한 값을 만듦
- scalar type이 String인 filed에 붙일 수 있음

&nbsp;

`uuid()`

```graphql
id  Int  @id @default(uuid())
```

- [uuid spec](https://en.wikipedia.org/wiki/Universally_unique_identifier)에 따라 고유한 값을 만듦
- scalar type이 String인 filed에 붙일 수 있음

&nbsp;

`now()`

```graphql
createAt DateTime @default(now())
```

- 생성된 시간의 timestamp를 저장
- scalar type이 DateTime filed에 붙일 수 있음

&nbsp;

`dbgenerated()`

- Prisma 스키마 (예 : random ())에서 표현할 수없는 기본값을 나타냄
- introspection 한 후에만 사용 가능
