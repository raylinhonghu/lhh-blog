---

title: 用 GraphQL + Typeorm 实现登陆功能（1）

category: GraphQL

date: 2019-08-09

author: 林鸿鹄

---

## 搭建服务器和创建数据库

今天我要给大家带来的是用 TypeGraphQL 来实现的一个服务器。
首先我们要准备的 package 包括：

```
package.json

"dependencies": {
		"apollo-server-express": "^2.16.0",
		"bcryptjs": "^2.4.3",
		"class-validator": "^0.12.2",
		"connect-redis": "^5.0.0",
		"cors": "^2.8.5",
		"express": "^4.17.1",
		"express-session": "^1.17.1",
		"graphql": "^15.3.0",
		"graphql-query-complexity": "^0.6.0",
		"graphql-upload": "^11.0.0",
		"ioredis": "^4.17.3",
		"nodemailer": "^6.4.10",
		"pg": "^8.3.0",
		"reflect-metadata": "^0.1.13",
		"type-graphql": "^1.0.0-rc.3",
		"typeorm": "^0.2.25",
		"uuid": "^8.3.0"
}
```

- bcryptjs 用于加密密码
- class-validator 用于校验类
- connect-redis，ioredis，用于连接redis
- nodemailer 用于发邮件
- express 服务器的基础设施
- cors 解决跨域问题
- reflect-metadata 是 typegraphql 需要用的
- uuid 可以生成随机的id
- graphql-upload 是用来上传文件到服务器的

首先我们需要在本地连接 数据库 和 redis。
这里我将使用 docker 来开启实例， 数据库我用的是 postgres。

### Postgres

打开命令行 运行 pg 的镜像

```
docker run -name postgres-0 -e POSTGRES_PASSWORD=mypassword -p 5432:5432 

```

假如需要本地查看你的实例（数据库） 可以使用 bash


```
Docker exec -it (postgres-0 / docker-id) bash 

```

下面是一些关于 pg 的操作

```
Psql --help
Psql -U postgres 用户名
\du 数据库用户列表
create database test; 新建数据库
\l 展示数据库列表
\c test 链接到数据库test
\d 展示关系
Create table student(); 建表
\dt 显示当前数据库的表格
\d student 显示表格student 的列
```

最重要的是我们要在根目录下创建一个 ormconfig 文件，
这个文件是关于我们 typeorm 的一些配置， 用于连接数据库：

```
{
	"name": "default",
	"type": "postgres",
	"host": "localhost",
	"port": 5433,
	"username": "postgres",
	"password": "mypassword",
	"database": "typegrapqlexample",
	"synchronize": true,
	"logging": true,
	"entities": ["src/entity/**/*.ts"]
}

``` 
- synchronize 意味着同步
- logging 是是否打印日志
- entities 是我们之后写的一些 typeorm 的 Entity

那么连接完数据库，如何创建并载入我们要的数据呢？
此时我们先创建一个前面说的 Entity 文件。

在位于根目录下的 entity 我们创建一个 User.ts


```
import { Entity, PrimaryGeneratedColumn, Column, BaseEntity } from "typeorm";
import { ObjectType, Field, ID, Root } from "type-graphql";

@ObjectType()
@Entity()
export class User extends BaseEntity {

    @Field(() => ID)
    @PrimaryGeneratedColumn()
    id: number;

    @Field()
    @Column()
    firstName: string;

    @Field()
    @Column()
    lastName: string;

    @Field()
    @Column("text", { unique: true })
    email: string;

    @Field()
    name(@Root() parent: User): string {
        return `${parent.firstName} ${parent.lastName}`;
    }

    @Column()
    password: string;


    @Column('bool', { default: false })
    confirmed: boolean;
}
```

先不要慌，让我来逐步解释。

```
@Entity()
export class User extends BaseEntity {}
```
@Entity 装饰器来自typeorm， 他创建的类允许我们直接使用他来对数据库直接操作。例如

```
User.create({...}).save();
```
@Column 装饰器同样来自typeorm，可以给数据库里添加列。在上面例子中我们给 User 这个类添加了 id， firstname， lastname， email， password， confirmed 几个列。而且我们可以给这个列设置属性和类别如下所示。

```
@Column("text", { unique: true })
email: string;

```


### Redis

现在我们开启 redis 实例， 之后我们会使用到他。

```
Docker pull redis 拉取
Docker run --name myredis -p 6666:6379 -e
docker exec -it 5623bf562625 redis-cli    // 运行redis-cli

```


## 真正开始

准备工作都就绪了， 我们现在又了我们正在运行的数据库和redis，现在我们开始搭建服务器。

### 连接数据库

typeorm 会自动连接我们的数据库。
```
await createConnection();
```

创建 ApolloServer 和 schema 

```
const schema = await createSchema();
const apolloServer = new ApolloServer({
        schema,
        context: ({ req, res }: any) => ({ req, res }),
        uploads: false
})

```
- schema 之后我会解释， 是创建我们之后要创建的 typegraphql的集合
- context 载有运行时的信息
- upload 是取消原有的上传文件


这个是根据所有 typegraphql 创建 schema 的函数

```
import { buildSchema } from "type-graphql";

export const createSchema = () => buildSchema({
    resolvers: [__dirname + "/../modules/**/!(*.test).{ts,js}"],

})

```

创建一个 express app 为 apolloServer 所用

```
const app = Express();

```


首先解决服务器解决跨域问题

```
app.use(cors({
        credentials: true,
        origin: 'http://localhost:3000'
}));

```

再把 Redis 连接进来

```
 app.use(
        session({
            store: new RedisStore({
                client: redis,
            }),
            name: "qid", // name of cookie
            secret: "asd ",
            resave: false,
            saveUninitialized: false,
            cookie: {
                httpOnly: true, // js can access 
                secure: process.env.NODE_ENV === "production", // only works in https 
                maxAge: 1000 * 60 * 60 * 24 * 365 // 7 years
            }
        })
)

```

最后把 express app 丢进 apollo 大功告成

```

apolloServer.applyMiddleware({ app })

app.listen(4000, () => {
        console.log("server started on http://localhost:4000/graphql")
})

```


之后的章节我会开始介绍如何写 注册账号的 Resolver～ 😁
