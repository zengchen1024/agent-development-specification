# 编码规范 · TypeScript 专项

> 本文件是 `coding/base.md` 的 TypeScript 语言扩展，仅记录 TypeScript / JavaScript 特有约定。

---

## 1. 代码风格

### 1.1 格式化与 Lint

- 使用 `Prettier` 格式化，`ESLint` 静态分析（CI 强制）
- 配置文件由 team-standards 统一维护，各项目引用：

> [团队填写] 例如：
>
> ```
> extends: ["@team/eslint-config"]
> ```

### 1.2 命名约定

| 类型             | 约定                            | 示例                               |
| ---------------- | ------------------------------- | ---------------------------------- |
| 变量 / 函数      | camelCase                       | `userName`, `getUser()`            |
| 类 / 接口 / 类型 | PascalCase                      | `UserService`, `UserDTO`           |
| 常量             | UPPER_SNAKE_CASE                | `MAX_RETRY_COUNT`                  |
| 枚举             | PascalCase（成员也 PascalCase） | `enum Status { Active, Inactive }` |
| 文件             | kebab-case                      | `user-service.ts`                  |
| React 组件文件   | PascalCase                      | `UserCard.tsx`                     |

---

## 2. 类型系统

### 2.1 TypeScript 配置

- 必须启用 `strict: true`（包含 `strictNullChecks`、`noImplicitAny` 等）
- 禁止使用 `@ts-ignore`（允许 `@ts-expect-error` 并附注释说明原因）
- 禁止使用 `any`（使用 `unknown` + 类型守卫代替）

### 2.2 类型定义原则

- 优先使用 `interface` 定义对象结构，`type` 用于联合类型、交叉类型等
- 数据模型定义集中在 `types/` 或 `models/` 目录
- 使用 `zod` 在运行时验证外部输入，并从 schema 推导类型（Single Source of Truth）

```typescript
// 推荐：用 zod 定义 schema，推导类型
import { z } from "zod";

const UserSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1),
  email: z.string().email(),
});

type User = z.infer<typeof UserSchema>;
```

### 2.3 泛型

- 泛型参数使用描述性名称（`TEntity`、`TResponse`），单字母仅限简单场景（`T`、`K`、`V`）
- 避免过度使用泛型，优先保持代码可读性

---

## 3. 错误处理

### 3.1 错误处理原则

- 使用 `Result` 模式（或 `neverthrow` 库）替代 throw/catch 处理业务错误
- 在最外层（API handler、事件处理器）统一捕获未处理异常
- `async/await` 函数必须处理 Promise rejection

```typescript
// 推荐：明确错误返回，而非 throw
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

async function getUser(id: string): Promise<Result<User>> {
  try {
    const user = await db.findUser(id);
    if (!user) return { success: false, error: new Error("not found") };
    return { success: true, data: user };
  } catch (err) {
    return { success: false, error: err as Error };
  }
}
```

---

## 4. 模块系统

### 4.1 Import 规范

- 使用 ES Module（`import/export`），禁止混用 CommonJS（`require`）
- 使用路径别名替代深层相对路径（如 `@/services/user` 替代 `../../../../services/user`）
- import 顺序：外部包 → 内部绝对路径（别名）→ 相对路径（由 ESLint 强制）

### 4.2 导出规范

- 优先命名导出（named export），避免 default export（难以重命名追踪）
- React 组件例外，允许 default export

---

## 5. 异步编程

### 5.1 async/await 优先

- 统一使用 `async/await`，禁止混用 `.then().catch()` 链式调用
- 并发执行使用 `Promise.all()`，有序执行使用 `await` 串行

```typescript
// 推荐：并发执行
const [users, orders] = await Promise.all([getUsers(), getOrders()]);

// 不推荐：串行造成不必要等待
const users = await getUsers();
const orders = await getOrders();
```

### 5.2 防止 Unhandled Rejection

- 所有 `Promise` 必须被 `await` 或显式处理 rejection
- 启动时执行的异步操作（如 `main()`）必须顶层 `.catch()` 处理

---

## 6. 项目结构

> [团队填写] 区分 Node.js 后端和 React/Next.js 前端：

**Node.js 后端**

```
src/
├── domain/           # 实体和接口
├── service/          # 业务逻辑
├── repository/       # 数据访问
├── api/              # 路由和控制器
├── types/            # 共享类型定义
└── index.ts          # 入口
```

**React / Next.js 前端**

```
src/
├── app/              # Next.js App Router 页面
├── components/       # 通用 UI 组件
├── features/         # 按功能聚合（组件+hooks+types）
├── lib/              # 工具函数和 SDK 封装
├── types/            # 全局类型定义
└── hooks/            # 通用 hooks
```

---

## 7. 依赖管理

> [团队填写] 包管理器选型，例如：
>
> - 统一使用 `pnpm`（Monorepo 友好，更严格的依赖隔离）
> - `package.json` 中锁定 engines 版本：`"node": ">=20"`

---

## 8. 测试

- 使用 `vitest`（或 `jest`）作为测试框架
- 单元测试：`<file>.test.ts` 与源文件同目录
- 组件测试：使用 `@testing-library/react`
- E2E 测试：使用 `Playwright`
- 使用 `msw`（Mock Service Worker）mock 网络请求

> 详细测试策略参见 `docs/standards/testing.md`
