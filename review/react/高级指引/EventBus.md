## 前言

React 的 EventBus没有官方库，一般是调用外部的EventEmitter库实现的

## 什么是EventBus

Event Bus 是一种设计模式，它利用了发布-订阅（Pub/Sub）模式实现组件、模块或者应用的不同部分之间的通信。Event Bus 允许不同的系统部分彼此独立，提高了代码的可维护性和可测试性。

## 为什么要用到EventBus

1. **组件间通信** ：当使用组件化开发时，不同的组件需要交换信息，但直接通信可能会收到组件结构影响，增大耦合度。通过Event Bus，组件可以发送事件而无需知道谁是监听者，监听者可以订阅感兴趣的事件并作出反应，而不需要了解事件的发送者。
2. **模块解耦** ：当系统分为多个模块或服务时，这些模块或服务可以订阅和发布事件而无需直接引用对方，降低了模块间耦合。
3. **跨界面通信** ：在前后端分离的应用程序中，后端可能会通过WebSocket或其他方式将事件发送给前端，前端的Event Bus可用来监听这些事件并更新界面。

## EventBus的实现

Event Bus 底层通过维护一个事件中心和一个事件监听器列表来实现。当事件发布者发布一个事件时，Event Bus 会查找所有订阅了该事件的监听器，并传递消息给它们。

```js
class EventBus {
  constructor() {
    this.listeners = {};
  }

  on(event, callback) {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event].push(callback);
  }

  off(event, callback) {
    const callbacks = this.listeners[event];
    if (callbacks) {
      const index = callbacks.indexOf(callback);
      if (index !== -1) {
        callbacks.splice(index, 1);
      }
    }
  }

  emit(event, ...args) {
    const callbacks = this.listeners[event];
    if (callbacks) {
      callbacks.forEach(callback => {
        callback(...args);
      });
    }
  }
}

```
