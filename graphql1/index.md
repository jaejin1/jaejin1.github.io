# Explore GraphQL


서버리스 GraphQL워크샵 내용을 참조했습니다. 

[tonyfromundefined/serverless-graphql-workshop](https://github.com/tonyfromundefined/serverless-graphql-workshop?fbclid=IwAR3G_oJXRw-sRXa0o6P06HPL3XsE3EPC9IvybcOqXtsXBWKIvPYGj4nJjQk)

<!--more-->

## GraphQL이란?

GraphQL은 API 설계와 요청을 구조화하는 일련의 약속이다.

GraphQL을 통해 우리는 데이터에 기반해 API를 디자인 할 수 있고, 클라이언트에서는 정해진 쿼리 언어를 통해 API를 체계적으로 사용할 수 있다.

- 클라이언트는 서버에 필요한 자원만 요청.
- 클라이언트는 서버가 가진 많은 자원을 단 한 번의 요청으로 가져 올 수 있다.
- 타입 시스템을 통해 개발 생산성을 비약적으로 향상 시킬 수 있다.
- 제공되는 기본 개발자 도구를 통해 API를 쉽게 문서화하고 검색할 수 있다.
- 버전 관리 없이 API를 점진적으로 진화 시킬 수 있다.

## GraphQL 타입 시스템과 쿼리, 뮤테이션 타입

기본 타입은 다음 5가지가 존재한다.

- `Int`: 부호가 있는 32비트 정수.
- `Float`: 부호가 있는 부동소수점 값.
- `String` : UTF-8 문자열.
- `Boolean` : `true` 또는 `false`
- `ID` : `ID` 스칼라 타입은 객체를 다시 요청하거나 캐시의 키로써 자주 사용되는 고유 식별자를 나타낸다. `ID` 타입은 String과 같은 방법으로 직렬화되지만, `ID`로 정의하는 것은 사람이 읽을 수 있도록 하는 의도가 아니라는 것을 의미한다.

#### 살펴보기

`User` 와 `Post` 라는 타입을 스키마에 선언

~~~sql
type User {
    id: ID!
    username: String!
    posts: [Post!]!
}

type Post {
    id: ID!
    title: String!
    content: String!
    author: User!
}
~~~

참고: `!` 는 데이터 값에 `null` 이 포함 될 수 없음을 나타낸다. 기본적으로 `!` 는 모두 붙여준다고 생각하자.

API 설계 단에서 다음과 같이 API에 필요한 타입들을 정의할 수 있다. 이제 해당 타입을 가지는 데이터를 가져올 수 있도록 해보자.

### 쿼리와 뮤테이션

쿼리와 뮤테이션은 기본적으로 선언되어야 한다.

- 쿼리는 데이터를 가져오는데 사용
    - `ID` 속성을 통해 Client 내부에서 Cache를 구현할 수 있다.
- 뮤테이션은 데이터를 조작하는데 사용.
    - 생성
    - 수정
    - 삭제
    - 등등..

이 두가지 타입을 통해 데이터를 가져오고, 수정할 수 있도록 `Query` , `Mutation` 타입 내에 속성들을 선언하자.

#### 살펴보기

~~~sql
type User {
    id: ID!
    username: String!
    posts: [Post!]!
}

type Post {
    id: ID!
    title: String!
    content: String!
    author: User!
}

type Query {
    user(id: ID!): User!
    post(id: ID!): Post!
}

type Mutation {
    createUser(username: String!): User!
    updateUser(id: ID!, username: String): User!
    deleteUser(id: ID!): User!
    createPost(title: String!, content: String!, authorId: String!): Post!
    updatePost(id: ID!, title: String, content: String): Post!
    deletePost(id: ID!): Post!
}
~~~

다음과 같이 `Query` 와 `Mutation` 에 속성 값으로 필요한 쿼리, 뮤테이션 들을 만들었다.

### 리졸버

이렇게 선언한 GraphQL 스키마를 어떻게 구동 시킬까?

바로 리졸버가 그 역할을 해준다. 리졸버는 타입 내 속성과 1:1로 일치시켜 구현이 필요한다. 리졸버는 `parent` , `args` 를 기본 argument로 하는 함수의 형태이다.

- `parent` 는 상위 리졸버에서 `return` 한 값을 나타낸다.
- `args` 는 쿼리문 내에서 넣어준 매개변수를 나타낸다.
- `context` 는 요청 하나를 타고 공유되는 전역 상태이다. ( 로그인 세션 정보등 저장 )

#### 살펴보기

~~~sql
const User = {
    id: (parent, args, context) => {
    return parent.id
    },
    username: (parent, args, context) => {
    return parent.username
    },
    posts: (parent, args, context) => {
    /* parent.id를 통해 DB에서 Post를 가져옵니다 */
    return posts
    },
}

const Post = {
    id: (parent, args, context) => {
    return parent.id
    },
    title: (parent, args, context) => {
    return parent.title
    },
    content: (parent, args, context) => {
    return parent.content
    },
    author: (parent, args, context) => {
    /* parent.authorId를 통해 DB에서 해당 User를 가져옵니다 */
    return user
    },
}

const Query = {
    user: (parent, args, context) => {
    /* args.id를 통해 DB에서 해당 User를 가져옵니다 */
    return user
    },
    post: (parent, args, context) => {
    /* args.id를 통해 DB에서 해당 Post를 가져옵니다 */
    return post
    },
}

const Mutation = {
    createUser: (parent, args, context) => {
    /* 새 유저를 생성해서 DB에 삽입합니다 */
    return user
    },
    updateUser: (parent, args, context) => {
    /* args.id를 통해 DB에서 기존 유저를 불러와 값을 수정 한 뒤 DB에 삽입합니다 */
    return user
    },
    deleteUser: (parent, args, context) => {
    /* args.id를 통해 DB에서 기존 유저를 삭제합니다 */
    return user
    },
    createPost: (parent, args, context) => {
    /* 새 게시물을 생성해서 DB에 삽입합니다 */
    return post
    },
    updatePost: (parent, args, context) => {
    /* args.id를 통해 DB에서 기존 게시물을 불러와 값을 수정 한 뒤 DB에 삽입합니다 */
    return post
    },
    deletePost: (parent, args, context) => {
    /* args.id를 통해 DB에서 기존 게시물을 삭제합니다 */
    return post
    },
}
~~~

### 요청

클라이언트가 다음과 같은 문법으로 서버에 요청을 날리면

~~~sql
query {
    user(id: "ea9f5eac-1449-5f03-a1c9-6521622de815") {
    id
    username
    }
}
~~~

이 요청에 응답하기 위해 서버에서 

1. `Query.user` 리졸버 함수 실행
2. 데이터베이스에서 User를 가져와서 해당 값을 리졸버 함수 내에서 `return` 
3. `return` 값을 `[User.id](http://user.id)` , `User.username` 리졸버에 `parent` argument로 전달
    1. `[User.id](http://user.id)` 리졸버 함수는 `parent` argument를 사용해, id 값을 `return` 
    2. `User.username` 리졸버 함수는 `parent` argument를 사용해, username값을 `return` 

순으로 리졸버 함수가 실행된다. 그 후 결과를 종합해 다음과 같이 JSON으로 응답 해준다.

~~~sql
{
    "user": {
    "id": "ea9f5eac-1449-5f03-a1c9-6521622de815",
    "username": "tonyfromundefined"
    }
}
~~~

이런 개발 방식을 GraphQL에서 Schema-First (SDL-First) 개발 방식이라고 한다.

이런 방식은 처음 GraphQL구현체가 등장했을 때 많이 사용되었다.

하지만 Schema-First 개발 방식은 다음과 같은 한계점이 있다.

1. 스키마 정의와 리졸버 간의 불일치 문제
2. GraphQL 스키마 분리 문제
3. 스키마 정의의 중복 (코드 재사용 문제)
4. IDE 지원 부족으로 인한 낮은 개발 경험
5. Schema 작성 문제

따라서 이러한 한계점을 효과적으로 해결 하기 위해 Code-First 개발 방식이 등장했다.

## Nexus로 시작하는 Code-First GraphQL 개발

앞에서 말한 Code-First 개발 방법에 대한 구현체로 Nexus라는 오픈소스 라이브러리를 Prisma에서 내놓았다.

- Node.js
- TypeScript
- Webpack
- Nexus

를 기반으로 하는 실제 GraphQL 프로젝트를 살펴보자.

#### 폴더 구조

https://github.com/jaejin1/serverless-graphql

`/` 

- `.env.development` , `.env.production`
    - 환경 변수 설정파일
- `package.json` , `yarn.lock`
    - 현재 프로젝트가 의존하고 있는 라이브러리와 버전 정보
    - `yarn` 을 이용해 라이브러리를 설치 할 수 있다.
- `tsconfig.json`
    - TypeScript 관련 설정파일
- `tslint.json`
    - TSLint 관련 설정파일
- `webpack.config.dev.js`
    - 개발 모드의 Webpack 빌드 설정
    - `yarn dev` 명령에서 해당 설정으로 Webpack이 작동
- `webpack.config.prod.js`
    - Production 모드의 Webpack 빌드 설정
    - `yarn build` 명령에서 해당 설정으로 Webpack이 작동.
- `serverless.yml`
    - 배포를 위한 Serverless Framework 설정

`/src/resolvers/` 

- `index.ts`
    - 엔트리 파일 `Query.ts` 와 `Mutation.ts` 가 내보낸 항목을 다시 내보내는 역할을 한다.
- `Query.ts`
    - 기본 Query 타입을 선언한다.
- `Mutation.ts`
    - 기본 Mutation 타입을 선언한다.

`/src/generated` 

- schema.graphql
    - Nexus가 자동 생성한 Schema이다.
    - Nexus가 자동 생성한 TypeScript Typing이다.

### 시작하기

- `/starters/server/` 폴더로 이동

~~~bash
$ cd ./starters/server
~~~

- 프로젝트에 필요한 라이브러리를 설치

~~~bash
# 기존에 yarn이 설치되어 있지 않다면,
$ npm i -g yarn

# 라이브러리 설치하기
$ yarn
~~~

- 개발 서버 시작

~~~bash
$ yarn dev
~~~

- Nexus 기반으로 작성된 `Query` 와 `Mutation` 살펴보기

Query.ts

~~~typescript
import { queryType } from 'nexus'

export const Query = queryType({
    definition(t) {
    t.string('stage', {
        resolve: (_parent, _args, _context) => {
        return process.env.STAGE as string
        },
    })
    },
})
~~~

Mutation.ts

~~~typescript
import { mutationType } from 'nexus'

export const Mutation = mutationType({
    definition(t) {
    t.string('ping', {
        resolve: (_parent, _args, _context) => {
        return 'pong'
        },
    })
    },
})
~~~

다음과 같이 Nexus를 통해서 코드를 작성하면 Nexus가 해당 코드를 이용해 `schema.graphql` 를 자동 생성해준다. 따라서 Schema-First에서 존재 했던 문제점인 **스키마 정의와 리졸버 간 불일치 문제와 Schema 작성문제**를 해결 할 수 있다.

추가적으로 Nexus가 `nexus.ts` 에 TypeScript 타이핑을 자동으로 생성해주기 때문에 개발 편의성을 느낄수 있다.


schema.graphql

~~~sql
type Query {
    stage: String!
}

type Mutation {
    ping: String!
}
~~~

## GraphQL Playground

API를 작성했다면 해당 API가 정상적으로 동작하는지 테스트 해야하는데 개발 서버를 띄워논 상태에서 `[http://localhost:3000/graphql](http://localhost:3000/graphql)` 로 접속하면 GraphQL 문서화 도구인 GraphQL Playground를 볼 수 있다.

GraphQL Playground를 통해, 

- GraphQL API를 검색하고
- 구현된 API를 테스트 할 수 있다.

~~~sql
query {
    stage
}


>>>
{
    "data": {
    "stage": "development"
    }
}

mutation {
    ping
}


>>>
{
    "data": {
    "ping": "pong"
    }
}
~~~

## `Task` 타입과 쿼리, 뮤테이션 만들기

resolvers 폴더에 task 폴더를 생성해서

`index.ts` ( Task 타입 정의 및 Query, Mutation을 받아서 export )

`Query.ts` ( Query 타입을 확장 )

`Mutation.ts` ( Mutation 타입을 확장 )

파일을 생성한다.

/src/resolvers/task/index.ts

~~~typescript
import { objectType } from 'nexus'

interface ITask {
    id: string
    content: string
    isDone: boolean
}

// 가상의 Database
export const TASKS: ITask[] = []

export const Task = objectType({
    name: 'Task',
    definition(t) {
    t.id('id', {
        description: 'Task 생성 시 자동 생성되는 Unique ID',
    })
    t.string('content', {
        description: 'Task 내용',
    })
    t.boolean('isDone', {
        description: 'Task 완료 여부',
    })
    },
})

export * from './Query'
export * from './Mutation'
~~~

/src/resolvers/task/Query.ts

~~~typescript
import { extendType, idArg } from 'nexus'
import { TASKS } from './'

export const TaskQueries = extendType({
    type: 'Query',
    definition(t) {
    t.field('task', {
        type: 'Task',
        args: {
        id: idArg({
            required: true,
        }),
        },
        resolve: (_parent, args) => {
        const task = TASKS.find((task) => task.id === args.id)

        if (task) {
            return task

        } else {
            throw new Error(`${args.id}를 가진 Task를 찾을 수 없습니다`)
        }
        },
    })

    t.list.field('tasks', {
        type: 'Task',
        resolve: () => {
        return TASKS
        },
    })
    },
})
~~~

/src/resolvers/task/Mutation.ts

~~~typescript
import { booleanArg, extendType, idArg, stringArg } from 'nexus'
import short from 'short-uuid'
import { TASKS } from './'

export const TaskMutations = extendType({
    type: 'Mutation',
    definition(t) {
    t.field('createTask', {
        type: 'Task',
        args: {
        content: stringArg({
            required: true,
        }),
        },
        resolve: (_parent, args) => {
        const task = {
            id: short.generate(),
            content: args.content,
            isDone: false,
        }

        TASKS.push(task)

        return task
        },
    })

    t.field('updateTask', {
        type: 'Task',
        args: {
        id: idArg({
            required: true,
        }),
        content: stringArg(),
        isDone: booleanArg(),
        },
        resolve: async (_parent, args) => {
        const taskIndex = TASKS.findIndex((task) => task.id === args.id)

        if (taskIndex > -1) {
            if (args.content) {
            TASKS[taskIndex].content = args.content
            }
            if (args.isDone) {
            TASKS[taskIndex].isDone = args.isDone
            }

            return TASKS[taskIndex]

        } else {
            throw new Error(`${args.id}라는 ID를 가진 Task를 찾을 수 없습니다`)
        }
        },
    })

    t.field('deleteTask', {
        type: 'Task',
        args: {
        id: idArg({
            required: true,
        }),
        },
        resolve: async (_parent, args) => {
        const taskIndex = TASKS.findIndex((task) => task.id === args.id)

        if (taskIndex > -1) {
            const task = TASKS[taskIndex]
            TASKS.splice(taskIndex, 1)

            return task

        } else {
            throw new Error(`${args.id}라는 ID를 가진 Task를 찾을 수 없습니다`)
        }
        },
    })
    },
})
~~~

기존 schema 엔트리 파일을 수정해 Task 엔트리 파일을 내보낸다.

/src/resolvers/index.ts

~~~typescript
export * from './Query'
export * from './Mutation'

export * from './task'
~~~

개발 서버를 띄워 놨다면, Nexus가 자동으로 Schema를 생성한다.

~~~sql
### This file was autogenerated by GraphQL Nexus
### Do not make changes to this file directly


type Mutation {
    createTask(content: String!): Task!
    deleteTask(id: ID!): Task!
    ping: String!
    updateTask(content: String, id: ID!, isDone: Boolean): Task!
}

type Query {
    stage: String!
    task(id: ID): Task!
    tasks: [Task!]!
}

type Task {
    """Task 내용"""
    content: String!

    """Task 생성 시 자동 생성되는 Unique ID"""
    id: ID!

    """Task 완료 여부"""
    isDone: Boolean!
}
~~~

GraphQL Playground에서 테스트 해보자.

~~~sql
mutation {
    createTask(content: "Hello, World") {
    id
    }
}


>>>
{
    "data": {
    "createTask": {
        "id": "3fZAgMTo7Va7B2DHuZRXTN"
    }
    }
}

query {
    tasks {
    id
    content
    }
}


>>>
{
    "data": {
    "tasks": [
        {
        "id": "3fZAgMTo7Va7B2DHuZRXTN",
        "content": "Hello, World"
        }
    ]
    }
}
~~~
