---

title: GraphQL + Apollo [后端]

category: GraphQL 

date: 2019-07-07

author: 林鸿鹄

---

### Restful 方式
获取一本书的数据: 

```
domain.com/books/:id
```
获取一个作者: 

```
domain.com/authors/:id
```

太多请求, 太多 routes。再看看另外一种方式

### GraphQL 方式

graphql 就像一幅 graph ！

```
{
	books(id: 12) {
		name
		author {
			name
		}
	}
}
```

用 GraphQL 可以简单的去请求我们要的数据。

吹了这么多，让我们开始介绍 GraphQL 吧！

### package.json

```
  "dependencies": {
    "cors": "^2.8.5", //解决跨域
    "express": "^4.17.1", // node 服务器
    "express-graphql": "^0.11.0", 
    "graphql": "^15.3.0",
    "lodash": "^4.17.19", // 方便操作数据
    "mongoose": "^5.9.28", // 更好操作数据库
    "nodemon": "^2.0.4" // 可以 watch 文件变化
  }
```

### 建立服务器
这次我打算用 express 以及 mongoDB 来搭建服务器。

引入express：

```
const express = require('express');
const app = express();
```


解决前端跨域问题：

```
app.use(cors())

```

让 express 和 graphql 融合的库：

```
app.use(
	'/graphql',
	graphqlHTTP({
		schema, // relation? type? how graph look?
		graphiql: true,
	})
);
```

监听端口：

```
app.listen(4000, () => {
	console.log('listening to port 4000');
});
```

### 连接 mongo 数据库

这里连接的是 mlab.com 的数据库， 调用了mongoose 的方法： 

```
mongoose.connect("mongodb://qqq111:qqq111@ds151943.mlab.com:51943/gqldb")

mongoose.connection.once('open',()=> {
    console.log("connected to the db")
})
```

### Schema

在 graphqlHTTP 里面我们传入了一个叫做 schema 的东西， 那么 schema 是什么呢？ 

Schema 可以描述数据的类型， 描述 graph 是长什么样子。


### Object Type

我们先创建一个 book 的 graphql 类型， 我们希望他有书名，类型，其作者，和 id。

创建这个类我们需要先引入一个类： 

```
const { GraphQLObjectType } = require('graphql');
```

这个类有2个属性， name 和 fields。

- name 是这个对象的名字。
- fields 描述graphql是什么样的。可能是一个对象，也可能是一个返回对象的函数。对象里面有描述这个对象的类型。 
 - 	假如是对象：会直接复制这个对象到 fields
 - 假如是一个返回对象的函数： 当引用的时候，两个类要知道是引用哪个类，但是直接声明对象会立即执行导致 undefined， 当用函数写法，直到整个文件结束才会执行


```
const BookType = new GraphQLObjectType({
	name: 'Book',
	fields: () => ({
		id: {
			type: GraphQLID,
		},
		name: {
			type: GraphQLString,
		},
		genre: {
			type: GraphQLString,
		},
		author: {
			type: AuthorType,
			resolve(parent, args) {
				return Author.findById(parent.authorId);
			},
		},
	}),
});
```
resolve 可以让他通过 parent (也就是book里的数据) 去数据库里查找这个book 的 author。

上面还有几个比较特殊的关键字：


```
GraphQLObjectType, GraphQLString, GraphQLID, GraphQLInt, GraphQLList, GraphQLNonNull 
```

分别对应： 

- GraphQLObjectType => 生成 graphql 对象
- GraphQLString => graphql 字符串
- GraphQLID => graphql ID
- GraphQLInt => graphql 整型
- GraphQLList => graphql 数组
- GraphQLNonNull => graphql 不为空

同样的我们再创建一个作者对象，作者会有名字，年龄，他写的书和 id：

```
const AuthorType = new GraphQLObjectType({
	name: 'Author',
	fields: () => ({
		id: {
			type: GraphQLID,
		},
		name: {
			type: GraphQLString,
		},
		age: {
			type: GraphQLInt,
		},
		books: {
			type: new GraphQLList(BookType),
			resolve(parent, args) {
				return Book.find({ authorId: parent.id });
			},
		},
	}),
});
```

### 继续 Schema
定义完了Book 和 Author 的对象， 我们可以把他们用在 Schema里。

GraphQLSchema 需要 RootQueryType 和 Mutation 来生成 schema： 

```
module.exports = new GraphQLSchema({
	query: RootQuery,
	mutation: Mutation,
});
```

### RootQuery
那么我们先来写 Root Query。

同样的这个对象也有 name 和 fields： 
- name 是 root query 的名字
- 这个 fields 里面包括了不同的 query。

我们先来写一个获取书的 query：

- 书的类型是我们之前声明的 BookType
- 我们可以通过书的 id 来获取书
- 最后我们需要描述如何通过数据库获取这本书

```
	book: {
			type: BookType,
			args: {
				id: { type: GraphQLID },
			},
			resolve(parent, args) {
				return Book.findById(args.id);
			},
		}
```

最后的 Root Query 如下所示： 

```
const RootQuery = new GraphQLObjectType({
	name: 'RootQueryType',
	// 不用函数包裹因为我们 root query 不在意顺序，没有相互引用
	fields: {
		book: {
			type: BookType,
			args: {
				id: { type: GraphQLID },
			},
			resolve(parent, args) {
				return Book.findById(args.id);
			},
		},
		books: {
			type: new GraphQLList(BookType),
			resolve(_, __) {
				return Book.find({});
			},
		},
		author: {
			type: AuthorType,
			args: {
				id: { type: GraphQLID },
			},
			resolve(parent, args) {
				return Author.findById(args.id);
			},
		},
		authors: {
			type: new GraphQLList(AuthorType),
			resolve(_, __) {
				return Author.find({});
			},
		},
	},
});
```

### Mutation

现在我们来写一个添加作者信息的 mutation： 
- 作者的类型是我们之前声明的 AuthorType
- 我们可以添加作者的名字和年龄到数据库
- 最后我们需要描述如何在数据库添加这位作者

```
	addAuthor: {
			type: AuthorType,
			args: {
				name: { type: new GraphQLNonNull(GraphQLString) },
				age: { type: new GraphQLNonNull(GraphQLInt) },
			},
			resolve(_, { name, age }) {
				const newAuthor = new Author({ name, age });

				return newAuthor.save();
			},
		}
```

最后的 mutation 如下所示： 

```
const Mutation = new GraphQLObjectType({
	name: 'Mutation',
	fields: {
		addAuthor: {
			type: AuthorType,
			args: {
				name: { type: new GraphQLNonNull(GraphQLString) },
				age: { type: new GraphQLNonNull(GraphQLInt) },
			},
			resolve(_, { name, age }) {
				const newAuthor = new Author({ name, age });

				return newAuthor.save();
			},
		},
		addBook: {
			type: BookType,
			args: {
				name: { type: new GraphQLNonNull(GraphQLString) },
				genre: { type: new GraphQLNonNull(GraphQLString) },
				authorId: { type: new GraphQLNonNull(GraphQLID) },
			},
			resolve(_, { name, genre, authorId }) {
				const newBook = new Book({
					name,
					genre,
					authorId,
				});

				return newBook.save();
			},
		},
	},
});

```

### mongoose

上面的数据库操作我们分别定义了两个 table。

AuthorModal

```
const moogoose = require('mongoose');

const Schema = moogoose.Schema;

const authorSchema = new Schema({
	name: String,
	age: Number,
	// 没 id 因为会被自动创建
});

module.exports = moogoose.model('Author', authorSchema);

```

BookModal

```
const moogoose = require('mongoose');

const Schema = moogoose.Schema;

const bookSchema = new Schema({
	authorId: String,
	name: String,
	genere: String,
	// 没 id 因为会被自动创建
});

module.exports = moogoose.model('Book', bookSchema);

```
