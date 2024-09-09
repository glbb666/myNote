### 是什么

React Native 框架用于构建原生移动应用。

在React Native中包含三部分。

1. **JavaScript 模块** ：这是你主要编写应用业务逻辑和 UI 的地方。使用 JavaScript 和 React 的语法，你创建组件，管理状态，处理用户输入，并定义应用的行为。
2. **原生代码（Native Code）** ：这包括预封装的原生模块和你自己可能编写的模块。原生模块提供了对平台特定功能的访问，如摄像头、地理位置服务、文件系统等，这些都是 JavaScript 无法直接访问的。原生代码也负责渲染原生 UI 组件，如按钮、文本框等。
3. **通信模块**：通信模块负责连接 JavaScript 和原生代码。它使得 JavaScript 能够调用原生模块的方法，同时允许原生代码触发 JavaScript 的事件处理器。

在 React Native 中，React 的 JavaScript 代码与原生代码之间的通信主要通过以下三种方式完成。

### JS Bridge

JS Bridge 是 React Native 最初使用的通信机制。它充当了 JavaScript 和原生代码之间的中介，使得 JavaScript 能够调用原生 API，以及原生代码能够触发 JavaScript 的事件处理器。

#### 工作原理：

1. **序列化和反序列化** ：当 JavaScript 调用原生方法时，参数会被序列化为 JSON 字符串，然后通过桥传送到原生侧。同样，当原生代码需要调用 JavaScript 函数时，也会将数据序列化为 JSON 字符串，然后通过桥传递回去。
2. **消息队列** ：JS Bridge 通常使用消息队列来处理 JavaScript 和原生代码之间的通信请求。这有助于确保通信的顺序性和一致性，即使在高并发的情况下也能正确处理。
3. **异步调用** ：由于序列化和反序列化的过程可能会带来性能开销，大部分调用都是异步的，以避免阻塞 JavaScript 执行线程或原生 UI 线程。

### JavaScript Interface (JSI)

随着 React Native 的发展，引入了 JavaScript Interface。

#### 工作原理：

1. **直接调用** ：JSI 允许原生代码直接调用 JavaScript 函数，而无需将数据转换为 JSON。这减少了不必要的数据转换，提高了通信速度。
2. **TurboModules** ：JSI 与 TurboModules 结合使用，后者是一种更高效的原生模块，它们可以被 JavaScript 直接调用。

### NativeEventEmitter

`NativeEventEmitter`是React Native提供的一个类，用于处理JavaScript和原生代码之间的事件通信。它允许原生模块通过事件的方式将数据传递给JavaScript，从而实现异步通信。

#### 工作原理

1. **注册事件** ：在JavaScript代码中，通过 `NativeEventEmitter`实例注册事件监听器。
2. **发送事件** ：在原生代码中，通过事件发送器发送事件。
3. **处理事件** ：JavaScript代码中的事件监听器接收到事件后，执行相应的处理逻辑。

### 原生监听JavaScript事件

在某些场景下，可以在原生代码中注册监听器，等待JavaScript的调用来触发这些监听器。

* **工作原理** ：原生代码监听特定的JavaScript事件，并在事件触发时执行相应的操作。JavaScript代码可以通过调用原生模块的方法来触发这些事件。
* **优点** ：适合需要原生代码主动监听某些JavaScript事件的场景。
* **缺点** ：实现和维护可能比较复杂，需要在原生代码中实现相应的监听逻辑。
