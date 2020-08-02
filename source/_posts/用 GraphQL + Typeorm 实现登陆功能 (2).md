---

title: 用 GraphQL + Typeorm 实现登陆功能（2）

category: GraphQL

date: 2019-09-01

author: 林鸿鹄

---

## 注册用户 Resolver 

### 什么是 Resolver？
resolver 是在 graphql 里处理数据的地方。它是一个函数会对不同的 graphql query 做出响应。

### 什么是 Grapql Query？
就和一般的数据CRUD差不多， graphql 有自己的 query 形式。 比如 Query， mutation 等。


### 字段参数的选择

注册用户我们首先要思考我们需要什么用户信息？

我们从 entity 那里可以了解到我们需要用户的 email 地址， 密码， 还有姓名。

所以 
firstName,
lastName,
password,
email
是我们要的参数。

### 返回值？
我们拿到用户信息之后，我们在数据库创建一个账号数据，并给用户发送一个确认邮箱的邮件， 假如创建成功就返回一个用户的对象到前端。


### TypeGraphQL
typeGraphql 提供了几个装饰器


@Resolver() 可以把这个类装饰成一个 Resolver

```
@Resolver(User)

```

Query 是说明这个方法是 Query 类型的查询

```
@Query(() => String)

```

Mutation 是说明这个方法是 Mutation 类型的操作

```
@Mutation(() => User)

```

@Arg() 来表示传入的参数

```
@Arg("data")

```


这里我们使用 bycrypt 来加密用户传来的信息， 并加密密码再储存到数据库

```
const hashedPassword = await bcrypt.hash(password, 12);
const user = await User.create({
            email,
            firstName,
            lastName,
            password: hashedPassword
}).save()

```

之后我们要给用户发送一个邮件，让他确认刚刚的注册操作。并且之后要给原本 User 数据的 confirmed （是否确认）从 false 改成 true， 表示用户通过点击右键上的连接已经完成确认。

发送邮箱前我们要生成这个用户自己独特的URL。

```
export const createConfirmUrl = async (userId: number) => {

    // 生成 token last 1 day
    const token = v4();
    await redis.set(confirmUserPrefix + token, userId, "ex", 60 * 60 * 24); // 1 day
    console.log("create : ",confirmUserPrefix + token)

    return `http://localhost:3000/user/confirm/${confirmUserPrefix + token}`;
}
```

因为我只想确认注册的 URL 存活一天。所以不把 Session 存到数据库而是存到 redis。


```
await sendEmail(email, await createConfirmUrl(user.id));
```

URL 拼接好后我们就把这个 URL 通过邮箱发送给用户的邮箱地址。用户可以通过点击邮箱跳转到前端， 在发送一个请求到 ConfirmUser 的 Resolver。

最后完整的代码是这样的： 

```
import { Resolver, Query, Mutation, Arg, UseMiddleware } from "type-graphql";
import * as bcrypt from 'bcryptjs';
import { User } from "../../entity/User";
import { RegisterInput } from "./register/RegitserInput";
import { isAuth } from "../middleware/isAuth";
import { logger } from "../middleware/logger";
import { sendEmail } from "../utils/sendEmail";
import { createConfirmUrl } from "../utils/createConfirmationUrl";

@Resolver(User)
export class RegisterResolver {

    @Query(() => String)
    @UseMiddleware(isAuth, logger)
    async hello() {
        return "Hello World";
    }

    @Mutation(() => User)
    async register(@Arg("data")
    {
        firstName,
        lastName,
        password,
        email
    }: RegisterInput): Promise<User> {
        const hashedPassword = await bcrypt.hash(password, 12);
        const user = await User.create({
            email,
            firstName,
            lastName,
            password: hashedPassword
        }).save()

        await sendEmail(email, await createConfirmUrl(user.id));
        console.log("user!:", user)
        return user;
    }
}

```
