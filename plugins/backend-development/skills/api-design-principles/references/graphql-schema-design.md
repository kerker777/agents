# GraphQL Schema 設計模式

## Schema 組織結構

### 模組化 Schema 結構
```graphql
# user.graphql
type User {
  id: ID!
  email: String!
  name: String!
  posts: [Post!]!
}

extend type Query {
  user(id: ID!): User
  users(first: Int, after: String): UserConnection!
}

extend type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
}

# post.graphql
type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
}

extend type Query {
  post(id: ID!): Post
}
```

## 型別設計模式

### 1. 非空值型別
```graphql
type User {
  id: ID!              # 必填
  email: String!       # 必填
  phone: String        # 可選（可為空值）
  posts: [Post!]!      # 非空值陣列，包含非空值文章
  tags: [String!]      # 可空值陣列，包含非空值字串
}
```

### 2. 多型介面
```graphql
interface Node {
  id: ID!
  createdAt: DateTime!
}

type User implements Node {
  id: ID!
  createdAt: DateTime!
  email: String!
}

type Post implements Node {
  id: ID!
  createdAt: DateTime!
  title: String!
}

type Query {
  node(id: ID!): Node
}
```

### 3. 異質結果聯集
```graphql
union SearchResult = User | Post | Comment

type Query {
  search(query: String!): [SearchResult!]!
}

# 查詢範例
{
  search(query: "graphql") {
    ... on User {
      name
      email
    }
    ... on Post {
      title
      content
    }
    ... on Comment {
      text
      author { name }
    }
  }
}
```

### 4. 輸入型別
```graphql
input CreateUserInput {
  email: String!
  name: String!
  password: String!
  profileInput: ProfileInput
}

input ProfileInput {
  bio: String
  avatar: String
  website: String
}

input UpdateUserInput {
  id: ID!
  email: String
  name: String
  profileInput: ProfileInput
}
```

## 分頁模式

### Relay 游標分頁（建議使用）
```graphql
type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type UserEdge {
  node: User!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

type Query {
  users(
    first: Int
    after: String
    last: Int
    before: String
  ): UserConnection!
}

# 使用方式
{
  users(first: 10, after: "cursor123") {
    edges {
      cursor
      node {
        id
        name
      }
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

### 偏移量分頁（較簡單）
```graphql
type UserList {
  items: [User!]!
  total: Int!
  page: Int!
  pageSize: Int!
}

type Query {
  users(page: Int = 1, pageSize: Int = 20): UserList!
}
```

## Mutation 設計模式

### 1. 輸入/回傳資料模式
```graphql
input CreatePostInput {
  title: String!
  content: String!
  tags: [String!]
}

type CreatePostPayload {
  post: Post
  errors: [Error!]
  success: Boolean!
}

type Error {
  field: String
  message: String!
  code: String!
}

type Mutation {
  createPost(input: CreatePostInput!): CreatePostPayload!
}
```

### 2. 樂觀回應支援
```graphql
type UpdateUserPayload {
  user: User
  clientMutationId: String
  errors: [Error!]
}

input UpdateUserInput {
  id: ID!
  name: String
  clientMutationId: String
}

type Mutation {
  updateUser(input: UpdateUserInput!): UpdateUserPayload!
}
```

### 3. 批次 Mutation
```graphql
input BatchCreateUserInput {
  users: [CreateUserInput!]!
}

type BatchCreateUserPayload {
  results: [CreateUserResult!]!
  successCount: Int!
  errorCount: Int!
}

type CreateUserResult {
  user: User
  errors: [Error!]
  index: Int!
}

type Mutation {
  batchCreateUsers(input: BatchCreateUserInput!): BatchCreateUserPayload!
}
```

## 欄位設計

### 參數與篩選
```graphql
type Query {
  posts(
    # 分頁
    first: Int = 20
    after: String

    # 篩選
    status: PostStatus
    authorId: ID
    tag: String

    # 排序
    orderBy: PostOrderBy = CREATED_AT
    orderDirection: OrderDirection = DESC

    # 搜尋
    search: String
  ): PostConnection!
}

enum PostStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

enum PostOrderBy {
  CREATED_AT
  UPDATED_AT
  TITLE
}

enum OrderDirection {
  ASC
  DESC
}
```

### 計算欄位
```graphql
type User {
  firstName: String!
  lastName: String!
  fullName: String!  # 在解析器中計算

  posts: [Post!]!
  postCount: Int!    # 計算而得，不會載入所有文章
}

type Post {
  likeCount: Int!
  commentCount: Int!
  isLikedByViewer: Boolean!  # 依據情境而定
}
```

## 訂閱

```graphql
type Subscription {
  postAdded: Post!

  postUpdated(postId: ID!): Post!

  userStatusChanged(userId: ID!): UserStatus!
}

type UserStatus {
  userId: ID!
  online: Boolean!
  lastSeen: DateTime!
}

# 客戶端使用方式
subscription {
  postAdded {
    id
    title
    author {
      name
    }
  }
}
```

## 自訂純量

```graphql
scalar DateTime
scalar Email
scalar URL
scalar JSON
scalar Money

type User {
  email: Email!
  website: URL
  createdAt: DateTime!
  metadata: JSON
}

type Product {
  price: Money!
}
```

## 指令

### 內建指令
```graphql
type User {
  name: String!
  email: String! @deprecated(reason: "Use emails field instead")
  emails: [String!]!

  # 條件包含
  privateData: PrivateData @include(if: $isOwner)
}

# 查詢
query GetUser($isOwner: Boolean!) {
  user(id: "123") {
    name
    privateData @include(if: $isOwner) {
      ssn
    }
  }
}
```

### 自訂指令
```graphql
directive @auth(requires: Role = USER) on FIELD_DEFINITION

enum Role {
  USER
  ADMIN
  MODERATOR
}

type Mutation {
  deleteUser(id: ID!): Boolean! @auth(requires: ADMIN)
  updateProfile(input: ProfileInput!): User! @auth
}
```

## 錯誤處理

### 聯集錯誤模式
```graphql
type User {
  id: ID!
  email: String!
}

type ValidationError {
  field: String!
  message: String!
}

type NotFoundError {
  message: String!
  resourceType: String!
  resourceId: ID!
}

type AuthorizationError {
  message: String!
}

union UserResult = User | ValidationError | NotFoundError | AuthorizationError

type Query {
  user(id: ID!): UserResult!
}

# 使用方式
{
  user(id: "123") {
    ... on User {
      id
      email
    }
    ... on NotFoundError {
      message
      resourceType
    }
    ... on AuthorizationError {
      message
    }
  }
}
```

### 回傳資料中的錯誤
```graphql
type CreateUserPayload {
  user: User
  errors: [Error!]
  success: Boolean!
}

type Error {
  field: String
  message: String!
  code: ErrorCode!
}

enum ErrorCode {
  VALIDATION_ERROR
  UNAUTHORIZED
  NOT_FOUND
  INTERNAL_ERROR
}
```

## N+1 查詢問題解決方案

### DataLoader 模式
```python
from aiodataloader import DataLoader

class PostLoader(DataLoader):
    async def batch_load_fn(self, post_ids):
        posts = await db.posts.find({"id": {"$in": post_ids}})
        post_map = {post["id"]: post for post in posts}
        return [post_map.get(pid) for pid in post_ids]

# 解析器
@user_type.field("posts")
async def resolve_posts(user, info):
    loader = info.context["loaders"]["post"]
    return await loader.load_many(user["post_ids"])
```

### 查詢深度限制
```python
from graphql import GraphQLError

def depth_limit_validator(max_depth: int):
    def validate(context, node, ancestors):
        depth = len(ancestors)
        if depth > max_depth:
            raise GraphQLError(
                f"Query depth {depth} exceeds maximum {max_depth}"
            )
    return validate
```

### 查詢複雜度分析
```python
def complexity_limit_validator(max_complexity: int):
    def calculate_complexity(node):
        # 每個欄位 = 1，列表會相乘
        complexity = 1
        if is_list_field(node):
            complexity *= get_list_size_arg(node)
        return complexity

    return validate_complexity
```

## Schema 版本控制

### 欄位廢棄
```graphql
type User {
  name: String! @deprecated(reason: "Use firstName and lastName")
  firstName: String!
  lastName: String!
}
```

### Schema 演進
```graphql
# v1 - 初始版本
type User {
  name: String!
}

# v2 - 新增可選欄位（向後相容）
type User {
  name: String!
  email: String
}

# v3 - 廢棄並新增欄位
type User {
  name: String! @deprecated(reason: "Use firstName/lastName")
  firstName: String!
  lastName: String!
  email: String
}
```

## 最佳實踐摘要

1. **可空值 vs 非空值**：從可空值開始，確定時才設為非空值
2. **輸入型別**：Mutation 必須使用輸入型別
3. **回傳資料模式**：在 mutation 回傳資料中返回錯誤
4. **分頁**：無限捲動使用游標分頁，簡單情況使用偏移量分頁
5. **命名**：欄位使用小駝峰命名，型別使用大駝峰命名
6. **廢棄**：使用 `@deprecated` 而非直接移除欄位
7. **DataLoaders**：關聯資料必須使用以防止 N+1 問題
8. **複雜度限制**：防範高成本查詢
9. **自訂純量**：用於領域特定型別（Email、DateTime）
10. **文件**：為所有欄位加上說明文件
