### 什么是生命周期：
从`Vue`实例创建、运行、到销毁期间，总是便随着各种各样的事件，这些事件，统称为生命周期

> 生命周期钩子 = 生命周期函数 = 生命周期事件

### 主要的生命周期函数分类：
#### 创建期间的生命周期函数
   - **`beforeCreate`:**实例刚被创建出来，data和methods属性还没有初始化好
   - **`created`：**data和methods已经创建好，此时还没有开始编译模板
   - **`beforeMount`：**完成了模板的编译，但是还没有挂载到页面中
   - **`mounted`：**已经将编译好的模板挂载到了页面指定的容器中显示

#### 运行期间的生命周期函数：
   - **`beforeUpdate`：**状态更新之前执行此函数，此时data中的状态是最新的，但是界面上数据还是旧的，因为此时还没有开始重新渲染DOM节点
   -  **`updated`：**实例更新完毕之后调用此函数，此时data中的状态值 和 页面上显示的数据都已经完成了更新，页面已经被重新渲染好了！
#### 销毁期间的生命周期函数：

   - **`beforeDestroy`**：实例销毁之前调用。在这一步，实例仍然完全可用
   - **`destroy`**：`Vue`实例销毁后调用。调用后，`Vue`实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190204154840188.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dhbmx1YmFiYTY2Ng==,size_16,color_FFFFFF,t_70)