---
layout: post
title: 什么是Flight组件
---

Twitter Flight是基于组件的. 那么什么是组件了?
组件是一块自我包含的原子的功能片段.
一个组件由三个部分组成:

1. 组件定义
2. 组件工厂
3. 组件实例

## 组件定义

组件由一个命名方法定义. 包括定义事件监听器, 触发器, 默认配置和私有方法.
组件没有public的方法或成员变量. 所有与外界的交互使用事件来完成.

```js
function heartbeat () {
  this.defaultAttrs({
    bpm: 80
  });
  this.beat = function () {
    this.trigger('heartbeat'); // trigger heartbeat event
  };
  this.after('initialize', function () {
    setInterval(this.beat.bind(this), 60000/bpm);
  });
}
```

## 组件工厂

Flight的`defineComponent`方法接受一个组件定义函数并返回一个组件工厂.

* 组件工厂用来创建组件实例并绑定到DOM.
* 一个组件工厂可以创建无数组件实例,
但是只有一个实例可以绑定到一个DOM节点.
* 组件实例化的时候可以覆盖默认配置.

```js
HeartBeat = defineComponent(heartbeat);
HeartBeat.attachTo('body');
HeartBeat.attachTo('#child', {
  bpm: 90
});
```

## 组件实例

* 使用组件工厂的attachTo方法创建组件实例.
* attachTo不会返回实例的应用.
* 组件实例可以和它绑定的DOM节点以及该节点的子节点交互.
* 组件实例通过事件与外界交互.

```js
function heartbeatMonitor () {
  this.handleHeartbeat = function () {
    console.log('Still alive.');
  };
  this.after('initialize', function () {
    this.on('heartbeat', this.handleHeartbeat);
  });
}
```

--------

本文翻译自[What is a Flight component?](http://simplebutgood.net/what-is-a-flight-component/)