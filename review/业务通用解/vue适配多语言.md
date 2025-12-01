# 背景

随着WP和WK的合并，WP需要接入WK这边的编辑器。

WP要做国际化，因此编辑器这边需要给网盘提供sdk包。

# 原理

i18n会读取浏览器navigator的语言配置，适配多语言文件。

把i18n的插件注入到项目中，在项目的代码中完成替换即可。

# 方案

我们的技术方案是vue-i18n。

### 多语言配置文件

配置语言文件：

例如： en-US.ts、 ja-JP.ts 和 zh-CN.ts

里面按照功能模块类型配置翻译的键值对

```ts
//src/locales
export default {
	common: {
		'save': 'Save',
		'add': 'Add,
	},
	toolbar: {
	},
	canvas: {
	},
}
```

### vue-i18n初始化配置

检测浏览器navigator的语言设置，来确定i18n需要去适配哪种语言。如果没有确定的语言，那就使用英语。

```ts
// /src/plugins/i18n.ts
import { createI18n } from 'vue-i18n'
import zhCN from '../locales/zh-CN'
import enUs from '../locales/en-us'

export type LocaleType = string

export interface I18nConfig {
  locale: LocaleType
  fallbackLocale: LocaleType
  messages: Record<LocaleType, any>
}

const defaultMessages = {
  'zh-CN': zhCN,
  'en-US': enUs
} as const

// 获取浏览器语言设置
const getBrowserLanguage = (): LocaleType => {
  const language = navigator.language
  if (language.includes('zh')) return 'zh-CN'
  return 'en-US'
}

// 创建默认配置
const defaultConfig: I18nConfig = {
  locale: getBrowserLanguage(),
  fallbackLocale: 'zh-CN',
  messages: defaultMessages
}

// 创建 i18n 实例
export const i18n = createI18n({
  ...defaultConfig,
  legacy: false // 使用 Composition API 模式
})

// 导出配置函数
// 支持模块化 & 懒加载
export const configureI18n = (config: Partial<I18nConfig>) => {
  // 更新 i18n 实例的配置
  if (config.locale) {
    i18n.global.locale.value = config.locale
  }
  
  if (config.fallbackLocale) {
    i18n.global.fallbackLocale.value = config.fallbackLocale
  }
  
  if (config.messages) {
    Object.keys(config.messages).forEach(locale => {
	// 支持动态合并包
      i18n.global.mergeLocaleMessage(locale, config.messages![locale])
    })
  }
}
```

### 入口配置

```ts
// src/main

import {i18n} from './plugin/i18n'

const initApp = (app)=>{
	app.use(i18n);
}
```

### 项目中的多语言使用方式

1. 在Vue组件中应该通过$t(）方法来获取翻译文本

   ```html
   <template>
   	<div>{{t('toolbar.colorPicker.themeColor')}}</div>
   </template>
   <script>
   import {useI18n} from 'vue-i18n';
   const {t} = useI18n();

   return {
   	t
   }
   </script>

   ```
2. 在ts代码中通过i18n替换文本

   ```ts
   import {i18n} from '@/plugins/i18n'

   const {t} = i18n.global;

   export default ()=>{
   	const {t} = i18n.global;

   	reader.onerror = ()=>{
   		message.error(t(`upload.fileReadError`))
   	}

   	return {
   		reader;
   	}
   };

   ```

### 通过组件实现语言切换功能

```html
// src/components/LanguageSwitcher.vue
   <template>
     <div class="language-switcher">
       <Button
         :checked="currentLanguage === 'zh-CN'"
         type="radio"
         @click="switchLanguage('zh-CN')"
       >
         中文
       </Button>
       <Button
         :checked="currentLanguage === 'en-US'"
         type="radio"
         @click="switchLanguage('en-US')"
       >
         English
       </Button>
     </div>
   </template>

   <script lang="ts" setup>
   import { ref } from 'vue'
   import Button from './Button.vue'
   import { configureI18n } from '../plugins/i18n'

   const currentLanguage = ref('zh-CN')

   const switchLanguage = (locale: 'zh-CN' | 'en-US') => {
     currentLanguage.value = locale
     configureI18n({ locale })
   }
   </script>


```

### 检测没有国际化的代码

```ts
const fs = require('fs');

// 引入逐行读取模块，用于大文件处理
const readline = require('readline');

const path = require('path');

// 配置要搜索的目录
const searchDir = '/modules/core/src'; // 修改为你的目录路径

// 定义一个正则表达式来匹配中文字符
const chinesePattern = /[\u4e00-\u9fa5]/;

// 不需要国际化的文件路径
const BLACK_PATH = [
    // 语言配置目录不检查
    'modules/core/src/locales',
    // 字体配置，不用检查
    // mock 数据
    // 忽略移动端
    // BToolbar相关的配置
    // 没有被使用的配置
    // 中文标记，未影响
    // 导出助手，不用修改
];

// 递归函数，用于遍历目录并搜索含有中文的文件
function searchFiles(dir) {
    // 读取目录内容，withFileTypes: true 返回fs.Dirent对象而非字符串
    fs.readdir(dir, { withFileTypes: true }, (err, files) => {
        if (err) {
            console.error('Error reading directory:', err);
            return;
        }

        files.forEach(file => {
            const filePath = path.join(dir, file.name);
  
            if (file.isDirectory()) {
                // 如果是目录，则递归搜索
                searchFiles(filePath);
            } else if (
                file.isFile() &&
		// 只处理JavaScript、TypeScript和Vue文件
                (filePath.endsWith('.js') || filePath.endsWith('.ts') || filePath.endsWith('.vue')) &&
		// 检查文件路径是否在黑名单中（some方法返回布尔值）
                !BLACK_PATH.some(blackPath => filePath.includes(blackPath))
            ) {
		// 符合条件的文件，进行中文检测
                readFileForChinese(filePath);
            }
        });
    });
}


// 读取单个文件并检测中文字符
function readFileForChinese(filePath) {

    // 创建可读流，指定UTF-8编码避免乱码
    const fileStream = fs.createReadStream(filePath, { encoding: 'utf8' });


    // 创建逐行读取接口
    // crlfDelay: Infinity 表示将\r\n和\n都视为换行符
    const rl = readline.createInterface({
        input: fileStream,
        crlfDelay: Infinity
    });
  
    let lineNumber = 0;
  

    // 注册行读取事件处理函数
    rl.on('line', (line) => {
        const newLine = line.trim();
        lineNumber++;
  

        // 确保只检测需要国际化的中文文本
        if (!newLine.includes('//') && // 排除单行注释
            !newLine.startsWith('*') && // 排除多行注释中的*行
            !newLine.startsWith('/*') && // 排除多行注释开始
            chinesePattern.test(newLine) && // 核心：检测是否包含中文字符
            !newLine.includes('console') && // 排除控制台输出（日志等）
            !newLine.includes('// ignore i18n') && // 排除特殊忽略标记
            !newLine.startsWith('al ')) { // 排除特定前缀（可能是代码别名）
            console.log(`File: ${filePath} (${lineNumber}), Line: ${newLine}`);
            // console.log('Found Chinese characters');
        }
    }).on('close', () => {
        // console.log(`File ${filePath} processed`);
    });
}

// 开始搜索
searchFiles(searchDir);
```

### 难点

**主要的难点不在于i18n项目的设计。因为这些配置其实比较标准化，其实通过网上的一些案例，ai都可以找到一些参考。**

难度主要在于，代码有几十万航，如何系统的去识别哪些代码没有进行i18n的配置，如果人工去识别的话，耗费周期比较长。

因此我实现了一个扫描的脚本去做这件事。

#### 难点1：**精准识别与误报过滤**

```js
// 需要过滤各种特殊情况
!newLine.includes('//') &&           // 注释
!newLine.startsWith('*') &&          // 文档注释  
!newLine.includes('console') &&      // 日志输出
!newLine.includes('// ignore i18n') // 特殊标记
```

 **挑战** ：平衡检测的准确性和覆盖率，减少人工复核成本

#### 难点2：**性能与内存优化**

```js
// 使用流式读取避免大文件内存溢出
const fileStream = fs.createReadStream(filePath);
const rl = readline.createInterface({
    input: fileStream,
    crlfDelay: Infinity  // 处理不同系统的换行符
});
```

 **挑战** ：项目代码量庞大，需要保证扫描效率

#### 难点3：**配置化与可维护性**

```js
const BLACK_PATH = [
    'modules/core/src/locales', // 语言文件本身
    // mock 数据、字体配置等特殊目录
];
```

 **挑战** ：设计灵活的排除机制，适应不同模块的特殊情况

### 第三层：解决方案的价值

```
node scripyts/findch.cjs
```

**这个脚本的执行时间是秒级别的，通过这个工具，我们：**

* **将人工排查时间从一周缩短到两天。**
* **实现了95%以上的准确率。**
* **建立了可复用的代码扫描基础设施**
* **为后续的代码质量检查奠定了基础**
