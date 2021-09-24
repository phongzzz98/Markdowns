# GraphQL Basic

## 1. Difference between GraphQL and REST api
- GraphQL có khả năng query sâu và nhanh hơn RestAPI thông thường. 
- Để lấy dữ liệu GraphQL chỉ cần 1 lần query là có thể lấy đc những thông tin cần thiết. Còn RestAPI thường sẽ phải truy vấn nhiều lần

> VD: Giả sử có 1 app quản lý sách với 2 nguồn dữ liệu chính là sách và tác giả. Sách có thể bao gồm các trường như tên, chi tiết, số trang,... còn tác giả có thể gồm các trường như tên, tuổi, thông tin chi tiết,...
Khi làm frontend giả sử có trường hợp bạn sẽ phải lấy các tên các tác giả và tên những quyển sách họ viết
> - Trong RestAPI: ta cần truy cập 2 endpoint /author rồi đến /author/:id/books.
Khi gọi đến authors ta sẽ thu đc TẤT CẢ thông tin của TẤT CẢ các tác giả trong khi ta chỉ cần đến tên của tác giả. Hơn nữa sau khi có được id ta có thể truy cập vào các quyển sách của tác giả đó, nhưng ở đây ta sẽ nhận được TẤT CẢ thông tin của TẤT CẢ số sách này, trong khi ta chỉ cần đến tên các quyển sách.
Trường hợp này ta đã overfetch dữ liệu
> - Trong GraphQL: Ta sẽ chỉ cần dùng 1 query để lấy những thông tin cần thiết.
```javascript
query{
    authors{ //Gọi đến các tác giả
        name // Lấy tên các tác giả
        books{ // Lấy ra những quyển sách của tác giả đó
            name // Lấy tên các quyển sách đó
        }
    }
}
```
##### => Giúp bên FrontEnd dễ thở hơn. Nếu cần gì chỉ cần gọi 1 query là done!!

## 2. What is GraphQL?
Là một ngôn ngữ truy vấn giúp người dùng dễ dàng read và mutate dữ liệu trong API
- Đối với BE Dev: GraphQL cung cấp 1 hệ thống "type" (na ná như TypeScript), cho phép việc xây dựng kiểu cho dữ liệu. Ta có thể hình dung như sau:
```javascript
type Book{
    id: number
    name: string,
    pages: number,
    desctiption: string,
    authorID: number
}
type Author{
    id: number
    name: string
    age: number
    books: [Book]
}
```
- Đối với FE Dev: Với dữ liệu được định sẵn kiểu, điều này giúp việc truy vấn chính xác thông tin muốn được sử dụng. VD:
```javascript
query GetBookNameAndAuthorName{
    authors{ 
        name 
        books{ 
            name 
        }
    }
}
```
## 3. How to use GraphQL?
GraphQL cho phép FE và BE tương tác với nhau dù sử dụng nhiều ngôn ngữ khác nhau. Trong VD sau: FE sẽ dùng JS còn BE sẽ dùng NodeJS và Express

### 3.1. Terminology
Một số thuật ngữ cần biết khi sử dụng GraphQL:
- **Schema**: Là thứ miêu tả cấu trúc chính của dữ liệu được sử dụng. Nó định nghĩa các thông tin từ backend qua các "type" (kiểu) và "field" (trường). Hơn nữa, trong schema còn xác định rõ các "query" (truy vấn) và các "mutation" (biến đổi) mà phía client có thể sử dụng
- **Field**: Là các trường trong schema, thường sẽ có nhiều trường trong 1 schema
```javascript
// Kiểu Book có 2 trường: title và author
type Book {
  title: String  // returns a String
  author: Author // returns an Author
}
```
- **Query**: Là một kiểu object đặc biệt nhằm giúp client thực hiện truy vấn dữ liệu (Read data)
- **Mutation**: Cũng như Query nhưng dùng để biến đổi dữ liệu như thêm sửa xóa (Write data)
- **Resolver**: Là hàm chịu trách nhiệm trả về dữ liệu của một trường. Hàm này có thể có các tham số hoặc không

### 3.2. Setting things up
Để sử dụng GraphQL với NodeJS và Express ta cần tạo server và cài đặt vài dependencies cho nó qua các bước sau
- Tạo server Express:
```bash
npm init 
```
- Cài đặt dependencies:
```bash
npm i express express-graphql graphql
```
- Ví dụ mở đầu: Tạo 1 file server.js và viết đoạn code sau
```javascript
// Require các dependencies và tạo app
const express = require('express')
const expressGraphQL = require('express-graphql')
const app = express()
// Require các type sẽ được graphql sử dụng 
const {
  GraphQLSchema,
  GraphQLObjectType,
  GraphQLString,
} = require('graphql')
// Tạo schema (mọi thứ đều cần có kiểu)
const schema = new GraphQLSchema({
  query: new GraphQLObjectType({     
      name: "HelloWorld",
      fields: () => ({
          message: {
              type: GraphQLString,
              resolve: () => "Hello World!!!"
          }
      })
  })
})
// Tạo route để khi vào localhost:5000/graphql code trang này sẽ đc chạy
app.use('/graphql', expressGraphQL({
  schema: schema, // Cho schema vừa tạo vào đây
  graphiql: true // Tạo UI có sẵn để dễ dàng thao tác với graphql
}))
// Để app chạy ở cổng localhost 5000
app.listen(5000, () => console.log('Server Running'))
```
##### => Ta đã tạo ra một query tên là HelloWorld trả về trường message là một string.

- Khi truy vấn:
```javascript
// Client gọi
query{
    message
}
// Server trả lời
{
    "data":{
        "message": "Hello World!!!"
    }
}
```
### 3.3. Comprehensive example
Dưới đây là một VD chi tiết hơn của quản lý sách 

```javascript
const express = require('express')
const expressGraphQL = require('express-graphql')
const {
  GraphQLSchema,
  GraphQLObjectType,
  GraphQLString,
  GraphQLList,
  GraphQLInt,
  GraphQLNonNull
} = require('graphql')
const app = express()

// Khởi tạo dữ liệu thay vì dùng database
const authors = [
	{ id: 1, name: 'J. K. Rowling' },
	{ id: 2, name: 'J. R. R. Tolkien' },
	{ id: 3, name: 'Brent Weeks' }
]

const books = [
	{ id: 1, name: 'Harry Potter and the Chamber of Secrets', authorId: 1 },
	{ id: 2, name: 'Harry Potter and the Prisoner of Azkaban', authorId: 1 },
	{ id: 3, name: 'Harry Potter and the Goblet of Fire', authorId: 1 },
	{ id: 4, name: 'The Fellowship of the Ring', authorId: 2 },
	{ id: 5, name: 'The Two Towers', authorId: 2 },
	{ id: 6, name: 'The Return of the King', authorId: 2 },
	{ id: 7, name: 'The Way of Shadows', authorId: 3 },
	{ id: 8, name: 'Beyond the Shadows', authorId: 3 }
]

// Tạo type cho sách
const BookType = new GraphQLObjectType({
  name: 'Book',
  description: 'This represents a book written by an author',
  fields: () => ({
    id: { type: GraphQLNonNull(GraphQLInt) },
    name: { type: GraphQLNonNull(GraphQLString) },
    authorId: { type: GraphQLNonNull(GraphQLInt) },
    author: {
      type: AuthorType,
      resolve: (book) => { // Resolve trả về data cho trường author có kiểu AuthorType gồm thông tin của tác giả của cuốn sách này
        return authors.find(author => author.id === book.authorId)
      }
    }
  })
})

// Tạo type cho tác giả
const AuthorType = new GraphQLObjectType({
  name: 'Author',
  description: 'This represents a author of a book',
  fields: () => ({
    id: { type: GraphQLNonNull(GraphQLInt) },
    name: { type: GraphQLNonNull(GraphQLString) },
    books: {
      type: new GraphQLList(BookType),
      resolve: (author) => { // Như resolve trên nhưng trả về danh sách các sách của tác giả này
        return books.filter(book => book.authorId === author.id)
      }
    }
  })
})

// Tạo 1 Query gốc, nơi sẽ chứa tất cả các query client sử dụng
const RootQueryType = new GraphQLObjectType({
  name: 'Query',
  description: 'Root Query',
  fields: () => ({
    book: { // Lấy thông tin 1 quyển sách khi truyền vào tham số id
      type: BookType,
      description: 'A Single Book',
      args: {
        id: { type: GraphQLInt }
      },
      resolve: (parent, args) => books.find(book => book.id === args.id)
    },
    books: { // Lấy tất cả sách
      type: new GraphQLList(BookType),
      description: 'List of All Books',
      resolve: () => books
    },
    authors: { // Lấy tất cả tác giả
      type: new GraphQLList(AuthorType),
      description: 'List of All Authors',
      resolve: () => authors
    },
    author: { // Lấy thông tin 1 tác giả
      type: AuthorType,
      description: 'A Single Author',
      args: {
        id: { type: GraphQLInt }
      },
      resolve: (parent, args) => authors.find(author => author.id === args.id)
    }
  })
})

// Tạo 1 Mutation gốc, nơi sẽ chứa tất cả các mutation client sử dụng
const RootMutationType = new GraphQLObjectType({
  name: 'Mutation',
  description: 'Root Mutation',
  fields: () => ({
    addBook: { // Thêm sách
      type: BookType,
      description: 'Add a book',
      args: {
        name: { type: GraphQLNonNull(GraphQLString) },
        authorId: { type: GraphQLNonNull(GraphQLInt) }
      },
      resolve: (parent, args) => { // Hàm resolve sẽ thực hiện công việc thêm
        const book = { id: books.length + 1, name: args.name, authorId: args.authorId }
        books.push(book)
        return book
      }
    },
    addAuthor: { // Thêm tác giả
      type: AuthorType,
      description: 'Add an author',
      args: {
        name: { type: GraphQLNonNull(GraphQLString) }
      },
      resolve: (parent, args) => {
        const author = { id: authors.length + 1, name: args.name }
        authors.push(author)
        return author
      }
    }
  })
})
// Tạo schema chứa query và mutation vừa tạo
const schema = new GraphQLSchema({
  query: RootQueryType,
  mutation: RootMutationType
})

app.use('/graphql', expressGraphQL({
  schema: schema,
  graphiql: true
}))
app.listen(5000, () => console.log('Server Running'))
```