# 什么是 ViteTest

ViteTest 是一个为 Vite 生态系统设计的测试框架，可以提供单元测试和端到端测试。

# 为什么重要

1. **快速测试** ：ViteTest 利用 Vite 的快速构建和热更新特性，使得测试运行速度极快，提高了开发效率。
2. **与 Vite 无缝集成** ：ViteTest 直接与 Vite 配置兼容，简化了测试环境的设置，无需额外配置。
3. **支持现代特性** ：支持 ES 模块、TypeScript 和 JSX，使其适用于现代前端框架（如 Vue 和 React）。
4. **丰富的功能** ：提供断言、快照测试、模拟、异步测试等功能，满足不同的测试需求。

# 怎么做

## **配置**

在项目根目录下创建 `vitest.config.js`，可以配置测试选项。

```js
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
  },
});

```

## **编写测试** 

在 `tests` 目录下编写测试文件，例如 `example.test.js`。

```js
import { describe, it, expect } from 'vitest';

describe('Example Test', () => {
  it('should work', () => {
    expect(true).toBe(true);
  });
});

```


#### **运行测试** ：

使用以下命令运行测试：

```sh
npx vitest
```
