# TypeScript 模式

`SKILL.md` 中每条规则对应的代码示例。底层的原则（principle）是语言无关的。参见 **type-system-discipline** 与 **boundary-discipline** 原则技能（skill）。

## 品牌类型

给原始类型打品牌，让它们无法混淆。在边界处验证一次。下游代码信任这个类型。

```ts
type AgentId = string & { readonly __brand: "AgentId" };

function isAgentId(input: string): input is AgentId {
  return isUUID(input);
}

function parseAgentId(input: string): AgentId {
  if (!isAgentId(input)) throw new Error(`Invalid agent id: ${input}`);
  return input;
}

function focusAgent(id: AgentId): void {
  /* input is trusted */
}
```

照 `readonly __brand: 'X'` 这个形状来。别发明新约定。

## 可辨识联合

用字面量判别字段给变体建模。每个变体共用同一个字段名，而每个变体的取值互不相同，于是不可能的组合无法被表示。

```ts
// Don't. Boolean + optionals lets contradictory states exist.
type DiffState = { loading: boolean; diff?: GitDiff; error?: string };

// Do. Only valid states exist.
type DiffState =
  | { kind: "loading" }
  | { kind: "ready"; diff: GitDiff }
  | { kind: "error"; error: string };
```

选定一个判别字段名（`kind`、`type`、`tag`）并坚持使用。

## 构造式建模

用全都合法的部件把类型构造出来，而不是用运行时检查去约束一个宽松类型。

非空，用可变元组实现：

```ts
type NonEmpty<T> = [T, ...T[]];

// Don't: T[] plus a length check every caller must repeat
function pickWinner(entries: string[]): string {
  if (entries.length === 0) throw new Error("no entries");
  return entries[Math.floor(Math.random() * entries.length)];
}

// Do: an empty value of the type can't exist
function pickWinner(entries: NonEmpty<string>): string {
  return entries[Math.floor(Math.random() * entries.length)];
}
```

拿到普通的 `T[]` 时，用守卫收窄一次。此后这个事实就随类型一路传下去：

```ts
const isNonEmpty = <T>(arr: T[]): arr is NonEmpty<T> => arr.length > 0;
```

偶数长度，成对表示：

```ts
type Pairs<T> = [T, T][];
```

时间区间，用起点加时长表示：

```ts
// Don't: a comment holds the invariant
type TimeRange = { start: Date; end: Date }; // start <= end

// Do: a negative range can't be written; derive end when needed
type TimeRange = { start: Date; durationMs: number };
```

让 `durationMs` 保持普通数字。只有当裸数字可能被传进期望时长的位置时，才给它打品牌（照品牌类型（Branded types）一节），不要条件反射地打。选定让坏状态无法构造的表示，再在上层暴露你需要的读取方式（`pairs.flat()`、一个 `rangeEnd()` 辅助函数）。

## 最简单的全类型

不要什么都强化。所有操作都保持全函数时，就保留 `T[]`：

```ts
const sum = (xs: number[]) => xs.reduce((a, b) => a + b, 0); // [] is 0, fine
```

当宽松类型在使用处逼出谎言时才强化。破绽是 `!`、`arr[0] as T`，以及一个"should never happen"抛错：

```ts
// Don't: partiality smuggled past the compiler
function newestSession(sessions: Session[]): Session {
  return sessions.at(0)!;
}

// Do: strengthen the input; the assertion disappears
function newestSession(sessions: NonEmpty<Session>): Session {
  return sessions[0];
}
```

把结果弱化为 `Session | undefined` 是另一种全函数签名。

## 用 `unknown` 不用 `any`

外部数据永远是 `unknown`。使用前先收窄。

```ts
// Don't
function handle(input: any) {
  return input.foo.bar;
}

// Do
function handle(input: unknown) {
  if (typeof input === "object" && input !== null && "foo" in input) {
    // narrowed; compiler verifies access
  }
}
```

外部来源包括 RPC 载荷、`JSON.parse`、`postMessage`、IPC、文件内容、环境变量、数据库结果。

## 先 schema 后手写守卫

在为外部数据手写逐属性的类型守卫之前，先找仓库的运行时 schema 库和现成的 schema。让一个 schema 独占验证，TypeScript 类型从它派生。不要维护一份 schema、一个重复的 interface、和一个会彼此漂移的守卫。

```ts
import { z } from "zod";

const UserSchema = z.object({
  id: z.string().uuid(),
  role: z.enum(["admin", "member"]),
});

type User = z.infer<typeof UserSchema>;

function parseUser(input: unknown): User {
  return UserSchema.parse(input);
}
```

失败是预期内的分支时用 `safeParse`。仓库用别的 schema 库时用等价的推导辅助。不要为一个守卫新加 schema 依赖。这条规则偏向代码库已经信任的 schema 体系。

## 禁用 `as` 断言

每个 `as` 都是潜在的运行时崩溃。只有类型系统验证过那个声称之后才做断言。

```ts
import { z } from "zod";

// Don't
const user = data as User;

// Don't
function isUser(data: unknown): data is User {
  return typeof data === "object" && data !== null && "id" in data;
}

// Do
const userSchema = z.object({ id: z.string(), name: z.string() });
type User = z.infer<typeof userSchema>;

function parseUser(data: unknown): User {
  return userSchema.parse(data);
}
```

类型先行时，用它证明的类型标注验证器。这样编译器就会拒绝证明力弱于该类型的验证器。把下面对象里的 `name` 删掉，赋值就通不过编译。

```ts
type User = { id: string; name: string };

const userSchema: z.ZodType<User> = z.object({ id: z.string(), name: z.string() });
```

把 `as` 从现有代码里重构掉时，先弄清 TypeScript 为什么推断不出来：

- 缺判别字段。补一个，换成可辨识联合。
- 源类型过宽（例如 `Record<string, unknown>`）。把它收窄。
- 无类型边界。用拥有该形状的 schema 解析。只在确实没有 schema 的地方新加 schema。
- 确实表达不出来。用品牌类型或 `satisfies`。

## 收窄层级

从最优到兜底：

1. **可辨识联合的 switch / if。**编译器自动收窄。
2. **`in` 运算符。**`"key" in obj` 收窄到包含该键的变体。
3. **`typeof` / `instanceof`。**用于原始类型和类实例。
4. **用户自定义类型守卫。**上面这些不够用时。
5. **`as` 断言。**只在验证之后。

```ts
function area(s: Shape): number {
  if ("radius" in s) return Math.PI * s.radius ** 2; // narrowed to circle
  return s.width * s.height; // narrowed to rect
}
```

## 类型守卫

守卫必须真的验证它声称的东西。撒谎的守卫比 `as` 更糟。

```ts
function isCircle(s: Shape): s is Shape & { kind: "circle" } {
  return s.kind === "circle";
}
```

可能的话优先用判别字段收窄。

## 穷尽性

在 default 分支里，把判别字段赋给一个 `never` 类型的局部变量。

```ts
// Value-returning switch
function area(s: Shape): number {
  switch (s.kind) {
    case "circle":
      return Math.PI * s.radius ** 2;
    case "rect":
      return s.width * s.height;
    default: {
      const _exhaustive: never = s;
      return _exhaustive;
    }
  }
}

// Void switch
function handle(s: Shape): void {
  switch (s.kind) {
    case "circle":
      drawCircle(s);
      break;
    case "rect":
      drawRect(s);
      break;
    default: {
      const _exhaustive: never = s;
      void _exhaustive;
    }
  }
}
```

值返回的 switch 用 return 风格，语句型 switch 用 void 风格。

## 用 `satisfies` 不用 `as`

`satisfies` 在验证的同时不拓宽字面量类型。

```ts
// Don't. Widens, loses literal types.
const config = { theme: "dark", cols: 3 } as Config;

// Do. Validates AND preserves literal types.
const config = { theme: "dark", cols: 3 } satisfies Config;
// config.theme is "dark" (literal), not string
```

## 边界验证

在数据跨界进入处验证一次。内部信任类型。参见 **boundary-discipline** 原则技能。

- **传输格式**（proto、JSON-RPC）。用 `ignoreUnknownFields` 解析，向前兼容的改动就不会弄坏旧客户端。
- **持久化的 JSON。**带版本号的数据块，解析处包一层 try/catch。
- 在调用链深处**不要重复验证**。

## 从 schema 派生类型

当 `.proto`、OpenAPI 规范、GraphQL schema 或数据库迁移已经定义了某个形状时，从生成的类型派生，不要重复定义。

```ts
// Don't. Duplicate shape, drifts when the schema changes.
type CheckSummary = {
  totalCount: number;
  checks: { name: string; status: string }[];
};
function renderChecks(s: CheckSummary) {
  /* ... */
}

// Do. Derive from the generated schema type.
import type { ChecksMessage } from "<generated module>";
function renderChecks(s: Pick<ChecksMessage, "totalCount" | "checks">) {
  /* ... */
}
```

写新 interface 之前先找 `Pick`、`Omit`、`Parameters`、`ReturnType`、`Awaited`、`typeof`。

## 对象参数

```ts
// Don't. Swap two args, still compiles.
openFile(uri, {
  startLineNumber: 10,
  startColumn: 1,
  endLineNumber: 10,
  endColumn: 1,
});

// Do. Order-independent, self-documenting.
openFile({
  uri,
  selection: {
    startLineNumber: 10,
    startColumn: 1,
    endLineNumber: 10,
    endColumn: 1,
  },
});
```

热路径上跳过这个做法：逐帧渲染、词法分析器、解析器，以及其他任何分配开销要紧的紧循环。
