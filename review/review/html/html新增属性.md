
# HTML5新增属性
https://juejin.im/post/5d1b43edf265da1bae3914ac

因为input新增的属性比较重要

## HTML5input属性

- **type**：要展示的控件类型

- **accept**：如果该元素的 type 属性的值是file,则该属性表明了服务器端可接受的文件类型；否则它将被忽略。该属性的值必须为一个逗号分割的列表,包含了多个唯一的内容类型声明：
  - 以 STOP 字符 (U+002E) 开始的文件扩展名。（例如：".jpg,.png,.doc"）
  - 一个有效的 MIME 类型，但没有扩展名
    - `audio/*` 表示音频文件
    - `video/*` 表示视频文件
    - `image/*` 表示图片文件

  - 
  - 
  - **multiple**：可一次性选择多个值，适用于<input>中type为email和file
    用法：<input type="file" multiple="multiple" />![1568906621919](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\1568906621919.png)
  - **placeholder**：适用于<input>中type为：text, search, url, telephone, email, password
  - **required**：规定不能为空，适用于以下类型的 <input> 标签：
    text, search, url, telephone, email, password, date pickers, number, checkbox, radio, file
    用法：
```html
    <input type="text" required="required" />
```
- **autocomplete**：自动完成，适用于 <form> 标签，以及以下类型的 <input> 标签：
  text, search, url, telephone, email, password, datepickers, range, color。
  用法：<form autocomplete="on“></form>或者单独在input中用off

- **autofocus**：自动地获得焦点，适用于所有 <input> 标签的类型
```html
<input autofocus="autofocus" />
```

- **autoSave**：当**type=search**时，之前的搜索值将在页面加载的下拉列表中保留

- **disabled**：禁用此控件，提交表单忽略此组件，当type=hidden时，此属性将被忽略

- **from**: input元素与其关联的表单元素。 属性的值必须是同一文档中元素的id。若不指定id,则input必须是from的后代

- **formenctype**：覆盖enctype属性

- **formmethod**：覆盖method属性

- **formnovalidate**：当input是提交按钮或者图片，则此属性指定在提交表单时需不需要验证。覆盖novalidate属性

- formatarget

  ：指示在提交表单后显示接收响应的位置，将覆盖target属性

  - _self：缺省值，在当前页面重新加载返回值
  - _blank：在新窗口加载返回值
  - _parent：在父级浏览上下文中打开，当没有父级，和_self一致
  - _top：HTML 4文档时清空当前文档，加载返回内容；HTML5时在当前文档的最高级内加载返回值，如果没有父级，和_self的行为一致
  - iframename: 返回值在指定frame中加载

- **height**：当type=image时定义图片高度

- **width**：当type=image时定义图片宽度

- **list**：预定义选项列表，值为同一文档的datalist的id，当type=hidden,checkbox,radio,file，此属性被忽略

- **multiple**：当type=file,email指示用户能否选择多个值。

- **name**：控件的名称。和值一起被提交

- **pattern**：当type=text,search,tel,url,email时检验控件值的正则表达式

- **placeholder**：当type=text.search,url,email,tel时指示输入框的作用

- **readonly**：指示用户无法修改控件的值

- **required**：在提交表单前必须先填充值，当type=hidden,image,reset,submit,button时忽略

- **selectionDirection**：选择发生的方向

- **size**：控件的初始大小。当type=text,password时，表示输入的字符的长度。从HTML5开始，适用于当type=text, search, tel, url, email,password。否则会被忽略。 值必须大于0。 缺省值20。浏览器显示不一致

- **spellcheck**：元素需要检查其拼写和语法

- **src**：当type=image为图片路径

- **step**：设置数字或日期时间值的增量

- **tabindex**：元素在当前文档的tab导航顺序中的位置

- **value**：控件的值

## HTML其他属性

- defer:加载完脚本后并**不执行**，而是等整个**页面加载完之后再执行**
```html
<script defer src=“URL”></script>
```
- async:加载完脚本后**立刻执行**，不用等整个页面都加载完，属于**异步执行**。
```html
<script async src=“URL”></script>
```
- Start —— 起始值 Reversed —— 倒叙排列
```html
<ol start="10" reversed>
<li>Html</li>
<li>Css</li>
<li>JavaScript</li>
</ol>
```

![1568981990317](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\1568981990317.png)

- manifest=“cache.manifest”（定义页面离线应用文件
```html
<html manifest=“cache.manifest”>
```

## type属性(要呈现的控件类型)

- **button**：无缺省行为按钮

- **checbox**：复选框。必须使用 value 属性定义此控件被提交时的值。使用 checked 属性指示控件是否被选择。也可以使用 indeterminate 指示复选框在一种不确定状态（大多数平台上，显示为一条穿过复选框的水平线）。

- **radio**：单选框，value为选定的值，当在同一个单选按钮组中，所有单选按钮的name值相同时，同一时间只能有一个按钮被选中

- **email**：用于输入e-mail字段，可以搭配CSS伪类:valid和:invalid使用

- **url**：编辑URL的字段，可以使用pattern和maxlength来约束输入的值

- **file**：用户选择文件，使用accept属性定义上传文件的可选择类型

- **hidden**：页面不显示，但是值会提交到服务器
  image：图片提交按钮，使用src属性来说明图片来源，使用alt来说明图片的信息，也可以使用width，height来定义图片大小

- **number**：数字控件。输入浮点数可以使用step属性来指明几位小数

- **password**：值为*的控件，可以使用maxlength指定可以输入的值的最大长度

- **range**(滑块)，不精确值的控件，缺省值为![img](https://img-blog.csdn.net/201804222020033)

  - min：0
  - max： 100
  - value：min+(max-min)/2，当max小于min时使用min
  - step：1

- **Date picker**:
  
  - Date —— 选取日、月、年![1568904533170](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\1568904533170.png)
  - Month —— 选取月、年![1568904801064](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\1568904801064.png)
  - Week —— 选取周和年![1568904818074](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\1568904818074.png)
  - Time —— 选取时间（小时和分钟）![1568904835769](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\1568904835769.png)
  - Datetime —— 选取时间、日、月、年（UTC 时间），废弃，用Datetime-local代替
- Datetime-local —— 选取时间、日、月、年（本地时间）![1568904882120](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\1568904882120.png)
  
- **search** 用来实现搜索框，可以和**autoSave**配合使用，之前的搜索记录会自动保存![1568905963880](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\1568905963880.png)

  > 输入文字之后可以删除

- **color**![1568904930898](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\1568904930898.png)

  > 若type="color"，那么input.value就是颜色的十六进制颜色码

- **tel**：输入电话号码的控件，可以使用**pattern**和**maxlength**来约束输入的值

- **text**：缺省值

- **reset**：将表单内容设置为缺省值

- **search**：输入搜索字符串的单行文本字段，换行会被从输入的值中自动移除。
- **submit**：提交表单的按钮