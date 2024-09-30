# coverage

## Providers

### 是什么

在 Vitest 中，`coverage.provider` 是控制测试覆盖率工具的配置选项。Vitest 支持多种工具来生成代码覆盖率报告，`provider` 就是用来指定哪个工具来收集和报告覆盖率数据。

### 有什么提供者

Vitest 提供两种覆盖率提供者：

#### **`c8`** （默认）：

* 性能较好。使用了 V8 引擎的内置功能，避免了一些额外的开销。
* 无需编译器插件即可收集覆盖率数据，性能较好。
* 与 Node.js 内置的工具集成紧密，适合对性能要求高的项目。

#### **`istanbul`** ：

* 在性能上稍慢一些，因为它进行更多的静态分析和处理，特别是在大型项目中。
* 提供 HTML、LCOV、text-summary 等多种格式的报告。
* 如果需要更多报告选项或集成不同的 CI 工具（如 Codecov、Coveralls），可以选择这个 provider。

#### 实际使用中

istanbul的效果更准确，可以把定义的箭头函数方法识别为function，而非变量。

### 怎么做

你可以在 Vitest 的配置文件（如 `vitest.config.js`）中通过 `coverage` 选项指定 `provider`，如下所示：

```js
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    coverage: {
      provider: 'istanbul', // 可以设置为 'c8' 或 'istanbul'
      reporter: ['text', 'json', 'html'], // 配置报告类型
      reportsDirectory: './coverage', // 报告生成的目录
    },
  },
});

```

#### 配置项

* **`eporter`** ：设置生成的报告类型（如 `text`, `json`, `html`, `lcov` 等）。
* **`reportsDirectory`** ：设置生成的覆盖率报告存放目录。
* **`include` 和 `exclude`** ：定义哪些文件/文件夹应被包含或排除在覆盖率报告之外。
* **`lines`, `functions`, `branches`, `statements`** ：设定最低覆盖率要求，低于这个阈值时测试会失败。
