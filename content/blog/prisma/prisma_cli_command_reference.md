---
title: '[Prisma] Prisma cli command'
date: 2021-02-15 18:02:09
category: prisma
thumbnail: { thumbnailSrc }
draft: false
---

여러분들이 익히 알고 있는 prisma init, prisma generate, prisma introspect, prisma migrate...등 모든것이 command line interface 입니다.

prisma cli는 npm package를 설치해야만 사용할 수 있으며 `@prisma/cli`로 제공하다 [2.16.0](https://github.com/prisma/prisma/releases/tag/2.16.0)버전에서 `prisma`로 변경되었습니다.

prisma 공식문서에는 prisma cli를 package.json에 locally하게 설치하라고 권고합니다. globally로 설치할 수 있기는 하지만 버전충돌 문제가 발생할 수 있습니다.

locally하게 설치한 prisma cli는 `npx` 접두사를 붙여 사용할 수 있습니다.

prisma cli는 다양한 명령어를 지원하고 있으며 이를 유연하게 사용할 수 있어야 합니다. 이번장에서는 어떤 명령어 들이 있으며 언제 사용해야 하는지 알아 보겠습니다.

&nbsp;

❗️ 몇몇 명령어는(migrate, introspect..)는 이후에 다시 다뤄볼 예정입니다.

## --help

사용 가능한 모든 명령어를 출력합니다.

```
npx prisma --help
```

&nbsp;

## version(-v)

현재 prisma version, platform, engine binaries 정보를 출력합니다.

```
npx prisma version
npx prisma version --json : json 형태로 출력
npx prisma -v
```

&nbsp;

## init

현재 프로젝트에 prisma를 사용하는데 필요한 파일을 생성합니다.

```
npx prisma init
```

명령어를 실행하면 아래의 2개의 파일이 생성됩니다.

`prisma/schema.prisma` : model을 정의  
`prisma/.env` : 환경변수 파일

&nbsp;

# generate

prisma/schema.prisma에 정의된 `model`과 `generator`을 기반으로 Prisma client같은 자원을 생성합니다.

```
npx prisma generate
```

위의 명령어를 실행하면 아래의 3단계가 실행됩니다.

1. 현재 디렉터리 및 상위 디렉터리를 검색하여 적용 가능한 `node_modules`를 찾습니다.
2. node_modules에 @prisma/client가 없으면 생성합니다.
3. 현재 디렉터리를 검사하여 schema.prisma를 찾아 (수정된) @prisma/client를 생성합니다.

🖐 model을 추가/삭제, 수정하는 경우 generate 명령어를 실행합니다. 만약 명령어를 실행하지 않는다면 생성/삭제/수정한 model을 Prisma Client API에서 사용할 수 없습니다. 또한 여러 type 역시 동기화 되지 않습니다

generate 명령어를 입력하고 `prisma-client-js`에 의해 생생성된 결과물은 <u>./node_modules/@prisma/client</u>에 존재합니다. 사실 내부적으로는 .prisma/client/index.d.ts에 모든 generated된 type들이 존재합니다.

⚠️ generate 명령어를 실행하기 위해서는 schema.prisma파일에서 `generator`를 정의해야 합니다. Prisma Client를 생성하는 데 사용되는 prisma-client-js 생성기는 schema.prisma에ㄴ 아래의 코드를 추가하여야 사용할 수 있습니다.

```prisma
generator client {
  provider = "prisma-client-js"
}
```

prisma generate는 2가지의 옵션을 제공합니다.

1. `--schema`  
   schema.prisma의 경로를 원하는 경로를 수정할 수 있습니다.  
   default path는 `./schema.prisma` `./prisma/schema.prisma` 입니다.

```
npx prisma generate --schema=./alternative/schema.prisma
```

&nbsp;

2. `--watch`

   schema.prisma의 변경사항을 지속적으로 확인합니다.

```
npx prisma generate --watch
```

&nbsp;

# introspect

introspect는 현재 DB schema를 기반으로 schema.prisma를 정의합니다. introspect 명령어를 실행하는데 있어 schema.prisma에 `datasource`는 정의되어 있어야 합니다. 타겟으로 하는 DB에 대한 정보는 알아야 하기 때문입니다.

⚠️ introspect는 명령어는 현재 schema.prisma의 내용을 모두 덮어씌우기 때문에 현재 schema.prisma 내용을 백업해놓아야 합니다.  
 ~~하지만..경험상 command + z로 되돌리면 문제가 없습니다.~~

prisma introspect 3가지의 옵션을 제공합니다.

1. `--force`  
   변경사항을 강제로 overwrite합니다.  
   <u>command + z 해도 그전의 내용이 복구되지 않습니다.</u>

   ```
   npx prisma introspect --force
   ```

&nbsp;

2. `--schema`  
   schema.prisma의 경로를 원하는 경로를 수정할 수 있습니다.  
   default path는 `./schema.prisma` `./prisma/schema.prisma`

   ```
   npx prisma introspect --schema=./alternative/schema.prisma
   ```

&nbsp;

3. `--print`  
   생성된 schema.prisma를 cli에 표시해줍니다.

   ```
   npx prisma introspect --print
   ```

&nbsp;

# push

⚠️ 현재 db push는 preview이기 때문에 production 환경에서는 사용하지 않는 것을 권장합니다. preview command를 사용하려면 명령어 뒤에 --preview-feature를 추가해야 합니다.

<u>db push는 schema.prisma의 내용을 migration없이 바로 db에 적용합니다.</u> 이 방법은 local development와 같이 schema.prisma에 대한 변경사항을 관리하지 않아도 크게 문제가 없는 경우 사용할 수도 있습니다.

또항 db push는 migration과 어떤 상호작용도 하지 않으며 migration table, migration file 역시 생성되지 않습니다.

db push는 아래의 흐름으로 동작합니다.

1. db push는 DB를 검사하고 DB schema에 schema.prisma의 상태를 반영하는 데 필요한 변경 사항을 추론하고 실행합니다.
2. 변경사항이 DB에 적용되면 `generator가 실행되며 prisma generate를 실행할 필요가 없습니다`.
3. db push의 결과로 데이터 손실이 발생할 수 있는데 이 경우, `에러를 던지고` 계속 진행하고 싶은 경우 `--force옵션으로 실행해야 합니다.`

&nbsp;

👍 db push를 추천하는 경우

- 다른 개발자 또는 production 환경에 배포할 필요 없이 local에서 schema를 빠르고 반복적 수정하는 경우

- 최종적인 DB schema으로 변화, 단계의 과정 즉, 일련의 과정보다는 단순히 최종 DB schema를 더 우선시 하는 경우 → 최종 DB schema만 있으면 되는 경우

&nbsp;

👎 db push를 추천하지 않는 경우

- 중요한 데이터를 손실할 수 없는 다른 환경에서 schema 변경 사항을 복제하는 경우
- 예를 들어 column을 삭제하고 새로운 column을 만드는 대신 column의 이름을 바꾸는 등 schema 변경이 실행되는 방식을 세밀하게 제어하려는 경우
- schema의 변경사항을 관리해야 하는 경우
- schema의 상태를 되돌리는 기능이 필요한 경우, db push를 다시 사용하여 원래 상태로 되돌릴 수는 있지만 데이터 손실에 대한 복구기능은 없음

&nbsp;

db push는 5가지의 옵션을 제공합니다.

1. `--preview-feature`  
   preview feature 명령어를 사용가능하게 합니다.

   ```
   npx prisma db push --preview-feature
   ```

&nbsp;

2. `--skip-generate`  
   Prisma Client를 generate하는 과정을 건너띕니다.

   ```
   npx prisma db push --skip-generate --preview-feature
   ```

&nbsp;

3. `--force`  
   데이터 손실에 대한 경고를 무시합니다.

   ```
   npx prisma db push --preview-feature --force
   ```

&nbsp;

4. `--help/-h`
   명령어에 대한 정보를 보여줍니다.

   ```
   npx prisma db push -help
   ```

&nbsp;

5. `--schema`  
   schema.prisma의 경로를 원하는 경로를 수정할 수 있습니다.  
   default path는 `./schema.prisma` `./prisma/schema.prisma`

   ```
   npx prisma db push --schema=./alternative/schema.prisma
   ```

&nbsp;

# seed

DB에 sample data를 삽입합니다. 개인적으로 seed라는 말을 잘 붙였다고 생각합니다 🌱

&nbsp;

db seed는 2가지의 옵션을 제공합니다.

1. `--preview-feature`  
   preview feature 명령어를 사용가능하게 합니다.

   ```
   npx prisma db seed --preview-feature
   ```

&nbsp;

2. `--help/-h`
   명령어에 대한 정보를 보여줍니다.

   ```
   npx prisma db seed -h
   ```

&nbsp;

# migrate

_prisme migrate는 현재 preview 단계이며 어떤 issue가 발생할 수 있습니다. 그렇기 때문에 production 환경에서의 사용은 지양합니다. 사용시에는 명령어에 `--preview-feature` 옵션을 주어야 합니다._

migration은 schema.prisma를 사용하여 DB schema를 생성할 수 있습니다. 이번장에서는 prisma migrate 명령어만 살펴보고 자세한 내용은 추후에 다루겠습니다.

&nbsp;

### dev

migrate dev 명령어는 schema.prisma 파일을 사용하여 DB를 업데이트 합니다. 기본적인 흐름은 다음과 같습니다.

1. pending migration을 적용

2) 새로운 migration을 DB에 적용하고 \_prisma_migrations 테이블을 수정
3) generate 진행(e.g - Prisma client)

&nbsp;

db push는 2가지의 옵션을 제공합니다.

1. `--name`  
   migration명을 지정합니다.

   ```
   npx prisma migrate dev --name "test" --preview-feature
   ```

&nbsp;

2. `--create-only`  
   migration 초안을 만듭니다. <u>실제 DB에 적용하지는 않고</u> 단순히 prisma/migrations/ 에만 생성됩니다.

   ```
   npx prisma migrate dev --create-only --preview-feature
   ```

&nbsp;

### reset

migrate reset은 아래의 흐름으로 실행됩니다.

1. DB 삭제
2. 같은 이름으로 DB 생성
3. 모든 migrations 적용
4. **<u>🌱 seed file 실행 </u>**

&nbsp;

db reset는 3가지의 옵션을 제공합니다.

1. `--force`  
   확인 메시지를 무시하고 reset을 진행합니다.

   ```
      npx prisma migrate reset --force --preview-feature
   ```

&nbsp;

2. `--skip-generate`  
   prisma generate하는 과정을 건너띕니다.

   ```
   npx prisma migrate reset --skip-generate --preview-feature
   ```

&nbsp;

3. `--skip-seed`  
   seed 파일 실행을 건너띕니다.

   ```
   npx prisma migrate reset --skip-seed --preview-feature
   ```

&nbsp;

### deploy

모든 pending migrations을 DB에 적용합니다. 여기서 pending migrations는 아직 DB에 적용되지 않은 prisma/migrations/에만 존재하는 migrations을 의미합니다.

```
npx prisma migrate deploy --preview-feature
```

### resolve

migrate를 할 때 prisma/migrations/의 파일과 DB의 prisma_migrations 테이블 데이터는 기본적으로 동기화 되어야 합니다. 만약 prisma/migrations의 A파일의 정보가 prisma_migrations 테이블에 저장되어 있으면 A파일에 대한 migrations을 적용하지 않습니다.

여기서 resolve 명령어는 <u>prisma_migrations 테이블에 A파일 정보를 삽입하는 명령어 입니다.</u> 이렇게 되면 prisma는 A파일이 실제 DB에 적용되지 않았지만 테이블 row에 존재하기 때문에 적용되었다고 생각하기 때문에 A파일의 migration을 진행하지 않습니다.

resolve 명령을 사용하면 migrations을 이미 적용 (baseline 지원) 또는 롤백으로 표시하여 production 환경에서의 migrations history 문제를 해결할 수 있습니다.

&nbsp;

migrate resolve는 3가지의 옵션을 제공합니다.

1. `--applied`  
   특정 migration을 `적용된`것으로 기록합니다.

   ```
   npx prisma migrate resolve --applied 20201231000000_add_users_table --preview-feature
   ```

&nbsp;

2. `--rolled-back`  
    특정 migration을 롤백으로 기록합니다.

   ```
   npx prisma migrate resolve --applied 20201231000000_add_users_table --preview-feature
   ```

&nbsp;

### status

prisma/migrations 폴더의 migration과 \_prisma_migration 테이블의 항목을 조회하여 migration 상태에 대한 정보를 확인합니다.

```
npx prisma migrate status --preview-feature
```

&nbsp;

### studio

`웹 환경에서` prisma studion를 사용할 수 있습니다.

⚠️ studio 명령어를 사용하기 전에 schema.prisma 파일에 유효한 `datasource`가 존재해야 합니다.

&nbsp;

migrate studio는 2가지의 옵션을 제공합니다.

1. `--port, -p`  
   특정 포트로 studio를 실행합니다.

   ```
   npx prisma migrate resolve --applied
   ```

&nbsp;

2. `--help, -h`  
   실행가능한 명령어를 보여줍니다.

   ```
   npx prisma studio --help
   ```

&nbsp;

# 📖 Reference

- <https://www.prisma.io/docs/reference/api-reference/command-reference#studio>
- <https://www.prisma.io/docs/guides/prisma-guides/seed-database>
- <https://www.prisma.io/docs/guides/prisma-guides/add-prisma-migrate-to-a-project>
