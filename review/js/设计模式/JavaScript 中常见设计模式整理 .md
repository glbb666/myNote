### JavaScript 中常见设计模式

- [单例模式](./单例模式.md)
- [策略模式](./表单验证——策略模式和ES6的Proxy代理模式.md)
- [代理模式](./代理模式.md)
- [迭代器模式](./迭代器模式.md)
- [发布-订阅模式](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fblob%2Fmaster%2FBasicSkill%2F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%2F%E5%8F%91%E5%B8%83%E8%AE%A2%E9%98%85%E6%A8%A1%E5%BC%8F.md)
- [命令模式](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fblob%2Fmaster%2FBasicSkill%2F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%2F%E5%91%BD%E4%BB%A4%E6%A8%A1%E5%BC%8F.md)
- [组合模式](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fblob%2Fmaster%2FBasicSkill%2F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%2F%E7%BB%84%E5%90%88%E6%A8%A1%E5%BC%8F.md)
- [模板方法模式](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fblob%2Fmaster%2FBasicSkill%2F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%2F%E6%A8%A1%E6%9D%BF%E6%96%B9%E6%B3%95%E6%A8%A1%E5%BC%8F.md)
- [享元模式](./享元模式.md)
- [职责链模式](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fblob%2Fmaster%2FBasicSkill%2F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%2F%E8%81%8C%E8%B4%A3%E9%93%BE%E6%A8%A1%E5%BC%8F.md)
- [中介者模式](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fblob%2Fmaster%2FBasicSkill%2F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%2F%E4%B8%AD%E4%BB%8B%E8%80%85%E6%A8%A1%E5%BC%8F.md)
- [装饰者模式](./装饰者模式.md)
- [状态模式](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fblob%2Fmaster%2FBasicSkill%2F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%2F%E7%8A%B6%E6%80%81%E6%A8%A1%E5%BC%8F.md)
- [适配者模式](./适配者模式.md)
- [观察者模式](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fblob%2Fmaster%2FBasicSkill%2F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%2F%E8%A7%82%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8F.md)

### 各设计模式关键词

看完了上述设计模式后，把它们的关键词特点罗列出来，以后提到某种设计模式，进而联想相应的关键词和例子，从而心中有数。

| 设计模式      | 特点                                                       | 案例                                                         |
| ------------- | ---------------------------------------------------------- | ------------------------------------------------------------ |
| 单例模式      | 一个类只能构造出唯一实例                                   | [创建菜单对象](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fblob%2Fmaster%2FBasicSkill%2F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%2F%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F.md) |
| 策略模式      | 根据不同参数可以命中不同的策略                             | [动画库里的算法函数](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fblob%2Fmaster%2FBasicSkill%2F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%2F%E7%AD%96%E7%95%A5%E6%A8%A1%E5%BC%8F.md) |
| 代理模式      | 代理对象和本体对象具有一致的接口                           | [图片预加载](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fblob%2Fmaster%2FBasicSkill%2F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%2F%E4%BB%A3%E7%90%86%E6%A8%A1%E5%BC%8F.md) |
| 迭代器模式    | 能获取聚合对象的顺序和元素                                 | `each([1, 2, 3], cb)`                                        |
| 发布-订阅模式 | PubSub                                                     | [瀑布流库](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fwaterfall%2Fblob%2F0f229c1a2881d26166b92aa746b7f892af59c28f%2Fwaterfall.js%23L8) |
| 命令模式      | 不同对象间约定好相应的接口                                 | [按钮和命令的分离](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fblob%2Fmaster%2FBasicSkill%2F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%2F%E5%91%BD%E4%BB%A4%E6%A8%A1%E5%BC%8F.md) |
| 组合模式      | 组合模式在对象间形成一致对待的树形结构                     | [扫描文件夹](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fblob%2Fmaster%2FBasicSkill%2F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%2F%E7%BB%84%E5%90%88%E6%A8%A1%E5%BC%8F.md) |
| 模板方法模式  | 父类中定好执行顺序                                         | [咖啡和茶](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fblob%2Fmaster%2FBasicSkill%2F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%2F%E6%A8%A1%E6%9D%BF%E6%96%B9%E6%B3%95%E6%A8%A1%E5%BC%8F.md) |
| 享元模式      | 减少创建实例的个数                                         | [男女模具试装](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fblob%2Fmaster%2FBasicSkill%2F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%2F%E4%BA%AB%E5%85%83%E6%A8%A1%E5%BC%8F.md) |
| 职责链模式    | 通过请求第一个条件，会持续执行后续的条件，直到返回结果为止 | [if else 优化](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fblob%2Fmaster%2FBasicSkill%2F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%2F%E8%81%8C%E8%B4%A3%E9%93%BE%E6%A8%A1%E5%BC%8F.md) |
| 中介者模式    | 对象和对象之间借助第三方中介者进行通信                     | [测试结束告知结果](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fblob%2Fmaster%2FBasicSkill%2F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%2F%E4%B8%AD%E4%BB%8B%E8%80%85%E6%A8%A1%E5%BC%8F.md) |
| 装饰者模式    | 动态地给函数赋能                                           | [天冷了穿衣服，热了脱衣服](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fblob%2Fmaster%2FBasicSkill%2F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%2F%E8%A3%85%E9%A5%B0%E8%80%85%E6%A8%A1%E5%BC%8F.md) |
| 状态模式      | 每个状态建立一个类，状态改变会产生不同行为                 | [电灯换挡](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fblob%2Fmaster%2FBasicSkill%2F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%2F%E7%8A%B6%E6%80%81%E6%A8%A1%E5%BC%8F.md) |
| 适配者模式    | 一种数据结构改成另一种数据结构                             | [枚举值接口变更](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fblob%2Fmaster%2FBasicSkill%2F%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%2F%E9%80%82%E9%85%8D%E8%80%85%E6%A8%A1%E5%BC%8F.md) |
| 观察者模式    | 当观察对象发生变化时自动调用相关函数                       | [vue 双向绑定](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FMuYunyun%2Fblog%2Fissues%2F11) |

