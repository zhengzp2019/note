# 前端学习教程

## 学习路线

```
JavaScript 基础 → TypeScript → 框架（React/Vue） → Node.js → 工程化
```

---

## 一、JavaScript 基础

### 1.1 变量与类型

```javascript
// 三种声明方式
var name = "旧方式"      // 不推荐，有作用域问题
let age = 25             // 可重新赋值
const PI = 3.14          // 常量，不可重新赋值

// 基本类型
let str = "hello"        // string
let num = 42             // number
let bool = true          // boolean
let empty = null         // null
let notDefined           // undefined
let sym = Symbol("id")   // symbol
let big = 9007199254740991n // bigint

// 引用类型
let arr = [1, 2, 3]
let obj = { name: "Tom", age: 25 }
```

### 1.2 函数

```javascript
// 函数声明
function add(a, b) {
  return a + b
}

// 箭头函数（推荐）
const add = (a, b) => a + b

// 默认参数
const greet = (name = "World") => `Hello, ${name}!`

// 解构参数
const printUser = ({ name, age }) => {
  console.log(`${name} is ${age} years old`)
}
printUser({ name: "Tom", age: 25 })
```

### 1.3 数组常用方法

```javascript
const nums = [1, 2, 3, 4, 5]

// map - 映射（类似 Java Stream 的 map）
const doubled = nums.map(n => n * 2)  // [2, 4, 6, 8, 10]

// filter - 过滤
const evens = nums.filter(n => n % 2 === 0)  // [2, 4]

// reduce - 归约
const sum = nums.reduce((acc, n) => acc + n, 0)  // 15

// find - 查找第一个
const found = nums.find(n => n > 3)  // 4

// some / every - 是否存在 / 是否全部满足
nums.some(n => n > 4)   // true
nums.every(n => n > 0)  // true

// 展开运算符
const merged = [...nums, 6, 7]  // [1, 2, 3, 4, 5, 6, 7]
```

### 1.4 对象操作

```javascript
const user = { name: "Tom", age: 25, city: "Beijing" }

// 解构赋值
const { name, age } = user

// 展开运算符（浅拷贝）
const updated = { ...user, age: 26 }

// 可选链（避免 NPE）
const street = user?.address?.street  // undefined，不会报错

// 空值合并
const displayName = user.nickname ?? user.name  // "Tom"
```

### 1.5 异步编程

```javascript
// Promise - 类似 Java 的 CompletableFuture
const fetchData = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve("data"), 1000)
  })
}

// async/await - 推荐写法
async function getData() {
  try {
    const result = await fetchData()
    console.log(result)
  } catch (error) {
    console.error(error)
  }
}

// 并发请求
const [users, posts] = await Promise.all([
  fetch("/api/users").then(r => r.json()),
  fetch("/api/posts").then(r => r.json())
])
```

### 1.6 模块系统

```javascript
// 导出 (math.js)
export const add = (a, b) => a + b
export const multiply = (a, b) => a * b
export default class Calculator { /* ... */ }

// 导入
import Calculator, { add, multiply } from "./math.js"
```

---

## 二、TypeScript

TypeScript = JavaScript + 静态类型系统。编译时检查类型错误，写完编译成 JS 运行。

### 2.1 基础类型

```typescript
// 类型注解
let name: string = "Tom"
let age: number = 25
let isActive: boolean = true
let items: string[] = ["a", "b", "c"]

// 函数类型
function add(a: number, b: number): number {
  return a + b
}

// 箭头函数类型
const greet = (name: string): string => `Hello, ${name}`
```

### 2.2 接口与类型别名

```typescript
// interface - 定义对象结构（类似 Java 的 interface）
interface User {
  id: number
  name: string
  email: string
  age?: number           // 可选属性
  readonly createdAt: Date  // 只读
}

// type - 类型别名（更灵活）
type Status = "active" | "inactive" | "banned"  // 联合类型
type Point = { x: number; y: number }

// 使用
const user: User = {
  id: 1,
  name: "Tom",
  email: "tom@example.com",
  createdAt: new Date()
}

function updateStatus(userId: number, status: Status) {
  // status 只能是 "active" | "inactive" | "banned"
}
```

### 2.3 泛型

```typescript
// 类似 Java 的泛型
function first<T>(arr: T[]): T | undefined {
  return arr[0]
}

first<number>([1, 2, 3])   // number
first<string>(["a", "b"])  // string

// 泛型接口
interface ApiResponse<T> {
  code: number
  message: string
  data: T
}

type UserResponse = ApiResponse<User>
type ListResponse = ApiResponse<User[]>

// 泛型约束
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]
}
```

### 2.4 常用工具类型

```typescript
interface User {
  id: number
  name: string
  email: string
  age: number
}

// Partial - 所有属性变为可选
type UpdateUser = Partial<User>

// Pick - 选取部分属性
type UserPreview = Pick<User, "id" | "name">

// Omit - 排除部分属性
type CreateUser = Omit<User, "id">

// Record - 键值对映射
type UserMap = Record<string, User>
```

### 2.5 类型收窄

```typescript
// typeof 收窄
function format(value: string | number): string {
  if (typeof value === "string") {
    return value.toUpperCase()  // 这里 TS 知道是 string
  }
  return value.toFixed(2)       // 这里 TS 知道是 number
}

// 判别联合类型（类似 Java sealed interface）
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "rect"; width: number; height: number }

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2
    case "rect":
      return shape.width * shape.height
  }
}
```

---

## 三、Node.js 基础

Node.js 是 JS 的服务端运行环境，让 JS 能做文件操作、网络服务等。

### 3.1 环境搭建

```bash
# 推荐使用 nvm 管理 Node 版本
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

nvm install 20     # 安装 Node 20 LTS
nvm use 20
node -v            # 验证
npm -v             # npm 包管理器
```

### 3.2 项目初始化

```bash
mkdir my-project && cd my-project
npm init -y                    # 生成 package.json
npm install typescript -D      # 安装 TS 为开发依赖
npx tsc --init                 # 生成 tsconfig.json
```

**package.json 核心字段：**

```json
{
  "name": "my-project",
  "scripts": {
    "dev": "ts-node src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js"
  },
  "dependencies": {},
  "devDependencies": {
    "typescript": "^5.0.0",
    "ts-node": "^10.0.0"
  }
}
```

### 3.3 常用内置模块

```typescript
import fs from "fs/promises"
import path from "path"

// 文件操作
const content = await fs.readFile("./data.txt", "utf-8")
await fs.writeFile("./output.txt", "hello")

// 路径拼接
const filePath = path.join(__dirname, "data", "config.json")
```

### 3.4 HTTP 服务

```typescript
import http from "http"

const server = http.createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "application/json" })
  res.end(JSON.stringify({ message: "Hello World" }))
})

server.listen(3000, () => {
  console.log("Server running at http://localhost:3000")
})
```

---

## 四、包管理与工具链

### 4.1 包管理器对比

| 工具 | 特点 |
|------|------|
| npm | Node 自带，最通用 |
| yarn | Facebook 出品，速度快 |
| pnpm | 最省磁盘空间，速度最快，推荐 |

```bash
# pnpm 安装
npm install -g pnpm
pnpm install express    # 安装依赖
pnpm add -D vitest      # 安装开发依赖
```

### 4.2 常用工具

| 工具 | 用途 |
|------|------|
| ESLint | 代码规范检查 |
| Prettier | 代码格式化 |
| Vite | 开发服务器 + 构建工具 |
| Vitest | 单元测试框架 |

---

## 五、前端框架入门（React）

### 5.1 创建项目

```bash
pnpm create vite my-app --template react-ts
cd my-app
pnpm install
pnpm dev    # 启动开发服务器
```

### 5.2 组件基础

```tsx
// 函数组件
interface Props {
  name: string
  age?: number
}

function UserCard({ name, age = 18 }: Props) {
  return (
    <div className="card">
      <h2>{name}</h2>
      <p>Age: {age}</p>
    </div>
  )
}

// 使用
<UserCard name="Tom" age={25} />
```

### 5.3 状态管理（Hooks）

```tsx
import { useState, useEffect } from "react"

function Counter() {
  // useState - 组件内状态
  const [count, setCount] = useState(0)

  // useEffect - 副作用（类似生命周期）
  useEffect(() => {
    document.title = `Count: ${count}`
  }, [count])  // count 变化时执行

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </div>
  )
}
```

### 5.4 数据请求

```tsx
import { useState, useEffect } from "react"

interface User {
  id: number
  name: string
}

function UserList() {
  const [users, setUsers] = useState<User[]>([])
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    fetch("/api/users")
      .then(res => res.json())
      .then(data => {
        setUsers(data)
        setLoading(false)
      })
  }, [])

  if (loading) return <p>Loading...</p>

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

---

## 六、与 Java 对比速查

| 概念 | Java | TypeScript |
|------|------|-----------|
| 包管理 | Maven / Gradle | npm / pnpm |
| 类型系统 | 强类型 | 强类型（编译时） |
| 接口 | `interface` | `interface` / `type` |
| 泛型 | `<T>` | `<T>` |
| 空安全 | `Optional<T>` | `?.` 可选链 + `??` 空值合并 |
| 异步 | `CompletableFuture` | `Promise` / `async await` |
| 集合操作 | Stream API | `map/filter/reduce` |
| 依赖注入 | Spring | 无框架级 DI，手动或用库 |
| 构建工具 | Maven | Vite / esbuild / webpack |
| 测试 | JUnit | Vitest / Jest |

---

## 七、推荐学习资源

- [MDN Web Docs](https://developer.mozilla.org/zh-CN/) — JS/CSS/HTML 权威文档
- [TypeScript 官方手册](https://www.typescriptlang.org/docs/) — TS 最佳入门
- [React 官方文档](https://react.dev/) — 新版交互式教程
- [JavaScript.info](https://javascript.info/) — 现代 JS 深入教程
