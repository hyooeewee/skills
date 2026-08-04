---
name: migrate-to-shoehorn
description: 迁移测试文件中的 `as` 类型断言至 @total-typescript/shoehorn。当用户提到
  shoehorn、想要替换测试中的 `as`，或需要部分测试数据时使用。

---

# 迁移至 Shoehorn

## 为什么要使用 shoehorn？

`shoehorn` 允许你在测试中传入部分数据，同时保持 TypeScript 的类型检查通过。它用类型安全的方式替代了 `as` 断言。

**仅限测试代码。** 永远不要在生产代码中使用 shoehorn。

`as` 在测试中的问题：

* 已被教导不使用它
* 必须手动指定目标类型
* 使用双重 `as` (`as unknown as Type`) 来传入故意错误的数据

## 安装

```bash
npm i @total-typescript/shoehorn
```

## 迁移模式

### 属性较少的大对象

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

## 如何选择

| 函数              | 使用场景                        |
| --------------- | --------------------------- |
| `fromPartial()` | 传入仍然能通过类型检查的部分数据            |
| `fromAny()`     | 传入故意错误的数据（保持自动补全）           |
| `fromExact()`   | 强制使用完整对象（稍后替换为 fromPartial） |

## 工作流程

1. **收集需求** - 询问用户：
   * 哪些测试文件中有导致问题的 `as` 断言？
   * 他们是否正在处理只有部分属性重要的大对象？
   * 他们是否需要传入故意错误的数据来进行错误测试？

2. **安装并迁移**：
   * [ ] 安装：`npm i @total-typescript/shoehorn`
   * [ ] 查找包含 `as` 断言的测试文件：`grep -r " as [A-Z]" --include="*.test.ts" --include="*.spec.ts"`
   * [ ] 将 `as Type` 替换为 `fromPartial()`
   * [ ] 将 `as unknown as Type` 替换为 `fromAny()`
   * [ ] 添加来自 `@total-typescript/shoehorn` 的导入
   * [ ] 运行类型检查以验证
