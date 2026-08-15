---
name: migrate-to-shoehorn
description: 将测试文件从 `as` 类型断言迁移到 @total-typescript/shoehorn。当用户提到
  shoehorn、想要在测试中替换 `as` 或需要部分测试数据时使用。

---

# 迁移到 Shoehorn

## 为什么使用 shoehorn？

`shoehorn` 允许你在测试中传入部分数据，同时让 TypeScript 保持满意。它用类型安全的替代方案取代了 `as` 断言。

**仅限测试代码。** 切勿在生产代码中使用 shoehorn。

在测试中使用 `as` 的问题：

* 被教导不要使用它
* 必须手动指定目标类型
* 用于故意传入错误数据的双重 as（`as unknown as Type`）

## 安装

```bash
npm i @total-typescript/shoehorn
```

## 迁移模式

### 大型对象，仅需少数属性

Before:

```ts
type Request = {
  body: { id: string };
  headers: Record<string, string>;
  cookies: Record<string, string>;
  // ...20 more properties
};

it("gets user by id", () => {
  // Only care about body.id but must fake entire Request
  getUser({
    body: { id: "123" },
    headers: {},
    cookies: {},
    // ...fake all 20 properties
  });
});
```

After:

```ts
import { fromPartial } from "@total-typescript/shoehorn";

it("gets user by id", () => {
  getUser(
    fromPartial({
      body: { id: "123" },
    }),
  );
});
```

### `as Type` → `fromPartial()`

Before:

```ts
getUser({ body: { id: "123" } } as Request);
```

After:

```ts
import { fromPartial } from "@total-typescript/shoehorn";

getUser(fromPartial({ body: { id: "123" } }));
```

### `as unknown as Type` → `fromAny()`

Before:

```ts
getUser({ body: { id: 123 } } as unknown as Request); // wrong type on purpose
```

After:

```ts
import { fromAny } from "@total-typescript/shoehorn";

getUser(fromAny({ body: { id: 123 } }));
```

## 何时使用它们

| 函数              | 使用场景                      |
| --------------- | ------------------------- |
| `fromPartial()` | 传递仍能通过类型检查的部分数据           |
| `fromAny()`     | 传递故意错误的数据（保留自动补全功能）       |
| `fromExact()`   | 强制完整对象（之后可换用 fromPartial） |

## 工作流程

1. **收集需求** - 询问用户：
   * 哪些测试文件中的 `as` 断言导致了问题？
   * 是否涉及大型对象，其中只有部分属性重要？
   * 是否需要为错误测试传入故意错误的数据？

2. **安装并迁移**：
   * [ ] 安装：`npm i @total-typescript/shoehorn`
   * [ ] 查找包含 `as` 断言的测试文件：`grep -r " as [A-Z]" --include="*.test.ts" --include="*.spec.ts"`
   * [ ] 将 `as Type` 替换为 `fromPartial()`
   * [ ] 将 `as unknown as Type` 替换为 `fromAny()`
   * [ ] 从 `@total-typescript/shoehorn` 添加导入
   * [ ] 运行类型检查以验证
