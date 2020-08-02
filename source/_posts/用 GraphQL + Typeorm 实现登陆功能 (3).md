---

title: 用 GraphQL + Typeorm 实现登陆功能（3）

category: GraphQL

date: 2019-09-14

author: 林鸿鹄

---

## 确认邮箱 Resolver 

确认邮箱的步骤就是当用户点击我们发送的 URL， 他会被带到前端去，前端会发送一个 Mutation 来让我们对数据库的 confirmed 字段进行修改。

用户在发送请求的时候会同时给我们发送一个 token 也是我们之前在 redis 里生成的 token。

所以我们的 Argument 里会有一个 token 字段

```
@Mutation(() => Boolean)
async confirmUser(@Arg("token") token: string): Promise<boolean> {}
```

第一步，我们通过 token 从 redis 获取 UserId

```
const userId = await redis.get(confirmUserPrefix + token)
```

第二步，我们通过 UserId 把数据库相对应的用户修改成已确认

```
if (!userId) {
   return false;
}

await User.update(
	{ id: parseInt(userId, 10) },
	{ confirmed: true }
);

```

第三步， 当我们成功修改用户的状态， 我们可以从 redis 把之前存的 token 给删除。

```
await redis.del(confirmUserPrefix + token);
```

确认邮箱的完整的代码如下：

```
@Resolver()
export class ConfrimUserResolver {

    @Mutation(() => Boolean)
    async confirmUser(@Arg("token") token: string): Promise<boolean> {
        
        const userId = await redis.get(confirmUserPrefix + token)
        if (!userId) {
            return false;
        }

        await User.update({ id: parseInt(userId, 10) }, { confirmed: true });

        await redis.del(confirmUserPrefix + token);

        return true;
    }
}
```

## 登陆 Resolver 

关于登陆，我们希望校验用户的信息是否和注册的信息一致，以及在用户登陆之后保存用户的登录状态。

登陆我们需要用户填写他们的邮箱，密码。

存贮用户的 session 我决定存在 context 里方便全局取用。

所以在 login 里我们会得到以下参数： 

```
@Mutation(() => User, { nullable: true })
async login(
    @Arg("email") email: string,
    @Arg("password") password: string,
    @Ctx() ctx: MyContext,
): Promise<User | null> { }
```
因为我们可能在数据库查不到用户， 所以返回值可能为 null。

首先我们会通过 email 去数据库里找到相对应的 user。

```
const user = await User.findOne({ where: { email } });

if (!user) {
     return null;
}
```

然后我们比较加密后的密码和数据库的密码是否一致。

```
const valid = await bcrypt.compare(password, user.password);

if (!valid) {
    return null;
}

```

我们在确认这个用户是否已经确认过邮箱，假如没确认过，说明没有激活， 我们不会给予 user 的信息。

```
if(!user.confirmed) {
   return null;
}
```

当用户提供的信息通过以上的校验，我们将通过登陆，并且把 userId 存储到 context 里。最后返回 user 的数据。

```
ctx.req.session!.userId = user.id;

return user;
```

登陆的 Resolver 最后如下所示： 

```
import { Resolver, Mutation, Arg, Ctx } from "type-graphql";
import * as bcrypt from 'bcryptjs';
import { User } from "../../entity/User";
import { MyContext } from "src/types/MyContext";

@Resolver(User)
export class LoginResolver {

    @Mutation(() => User, { nullable: true })
    async login(
        @Arg("email") email: string,
        @Arg("password") password: string,
        @Ctx() ctx: MyContext,
    ): Promise<User | null> {
        
        const user = await User.findOne({ where: { email } });

        if (!user) {
            return null;
        }

        const valid = await bcrypt.compare(password, user.password);

        if (!valid) {
            return null;
        }

        if(!user.confirmed) {
            return null;
        }

        ctx.req.session!.userId = user.id;

        return user;

    }
}
```

## 登出 Resolver 
这个相对于登陆简单许多， 我们要做的只是把存在于 context 中的 userID 移除即可， 并且清除掉 cookie 就可以。

最终代码如下： 

```
@Resolver()
export class LogoutResolver {
    @Mutation(() => Boolean)
    async logout(
        @Ctx() ctx: MyContext
    ): Promise<Boolean> {

        return new Promise((res, rej) => {
            ctx.req.session!.destroy((err) => {
                if (err) {
                    console.log(err)
                    return rej(false)
                }
                ctx.res.clearCookie('qid')
                return res(true)
            })
        })
    }
}
```


## 忘记密码 Resolver 
忘记密码我们也会向用户发送一封邮件，当用户收到邮件后就会跳转至修改密码的页面。

首先当用户提供他们的邮箱地址， 我们要去数据库查找这个邮箱是否存在。

```
const user = await User.findOne({ where: { email } })
if (!user) {
   return true;
}
```

假如存在的话我们在 redis 里保存这个用户的 token 和 userID，并且生成 URL 发送给用户的邮箱。 

```
const token = v4();
await redis.set(forgotPasswordPrefix + token, user.id, "ex", 60 * 60 * 24); // 1 day

await sendEmail(email, `http://localhost:3000/user/change-password/${forgotPasswordPrefix + token}`);
```

最后代码如下： 

```
@Resolver()
export class forgotPasswordResolver {

    @Mutation(() => Boolean)
    async forgotPassword(@Arg("email") email: string): Promise<boolean> {

        const user = await User.findOne({ where: { email } })
        if (!user) {
            return true;
        }
        // 生成 token last 1 day
        const token = v4();
        await redis.set(forgotPasswordPrefix + token, user.id, "ex", 60 * 60 * 24); // 1 day

        await sendEmail(email, `http://localhost:3000/user/change-password/${forgotPasswordPrefix + token}`);
        return true;
    }
}
```

## 修改密码 Resolver 
修改密码我们首先会拿到用户传来的 token， 通过 token 去 redis 获取 userID

```
const userId = await redis.get(forgotPasswordPrefix + token);

if (!userId) {
   return null;
}

```

拿到 userID 后去数据库查找相对应的 user， 然后我们把新的密码进行加密，加密完成后把 user 的密码更新。

```
const hashedPassword = await bcrypt.hash(password, 12);

user.password = hashedPassword;

await user.save();
```

我还希望用户成功修改密码后， 我们可以把他的 session 保存到 context。这样用户就无需再登陆了。

```
ctx.req.session!.userId = user.id;
```

最后修改密码完成后， 我们可以吧 redis 中的 token 删除。

```
redis.del(forgotPasswordPrefix + token)
```

完整的修改密码代码如下： 

```
@Resolver()
export class changePasswordResolver {

    @Mutation(() => User, { nullable: true })
    async changePassword(
        @Arg("data") data: ChangePasswordInput,
        @Ctx() ctx: MyContext
        ): Promise<User | null> {
        const { token, password } = data;

        const userId = await redis.get(forgotPasswordPrefix + token);

        if (!userId) {
            return null;
        }

        const user = await User.findOne({
            where: {
                id: userId
            }
        })

        if (!user) {
            return null;
        }

        const hashedPassword = await bcrypt.hash(password, 12);

        user.password = hashedPassword;

        await user.save();

        ctx.req.session!.userId = user.id;

        redis.del(forgotPasswordPrefix + token)

        return user
    }
}
```
