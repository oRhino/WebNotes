# vue3 笔记

## setup如何获取组件实例

- 在setup和其它composition api中是没有this的
- 可通过getCurrentInstance获取当前实例
- 若使用options api可照常使用this

## vue3为什么比vue2快?

- Proxy响应式
- PatchFlag
- HoistStatic 静态提升
- cacheHandler 事件监听缓存
- SSR优化
- tree-shaking

### PatchFlag
- 编译模板时,动态节点做标记
- 标记,分为不同的类型,如TEXT,PROPS
- diff算法时,可以区分静态节点,以及不同类型的动态节点 (只对比动态节点)
   . 只比对带有PF的节点
   . 并且通过Flag的信息得知当前节点要比对的具体内容


```
// https://vue-next-template-explorer.netlify.app/
<div>
  <span>Hello World</span>  
  <span>{{title}}</span>  
  <span :class="obj"></span>  
  <span :id="obj" :msg="message"></span>  
  <span :class="obj">{{hel}}</span>  
  <span :id="obj">{{xx}}</span>  
  <span :class="obj" :id="xx">{{hel}}</span>  
</div>

const _Vue = Vue

return function render(_ctx, _cache, $props, $setup, $data, $options) {
  with (_ctx) {
    const { createElementVNode: _createElementVNode, 
    toDisplayString: _toDisplayString, 
    normalizeClass: _normalizeClass,
    openBlock: _openBlock, 
    createElementBlock: _createElementBlock 
    } = _Vue

    return (_openBlock(), _createElementBlock("div", null, [
     //静态节点
      _createElementVNode("span", null, "Hello World"), 
      //动态节点 区分不同的类型,TEXT,PROPS,CLASS... ,用数组存放动态属性
      _createElementVNode("span", null, _toDisplayString(title), 1 /* TEXT */),
      _createElementVNode("span", {
        class: _normalizeClass(obj)
      }, null, 2 /* CLASS */),
      _createElementVNode("span", {
        id: obj,
        msg: message
      }, null, 8 /* PROPS */, ["id", "msg"]),
      _createElementVNode("span", {
        class: _normalizeClass(obj)
      }, _toDisplayString(hel), 3 /* TEXT, CLASS */),
      _createElementVNode("span", { id: obj }, _toDisplayString(xx), 9 /* TEXT, PROPS */, ["id"]),
      _createElementVNode("span", {
        class: _normalizeClass(obj),
        id: xx
      }, _toDisplayString(hel), 11 /* TEXT, CLASS, PROPS */, ["id"])
    ]))
  }
}

```
在packages/shared/src/patchFlags.ts

```
export const enum PatchFlags {
  
  TEXT = 1,// 1 动态的文本节点
  CLASS = 1 << 1,  // 2 动态的 class
  STYLE = 1 << 2,  // 4 动态的 style
  PROPS = 1 << 3,  // 8 动态属性，不包括类名和样式
  FULL_PROPS = 1 << 4,  // 16 动态 key，当 key 变化时需要完整的 diff 算法做比较
  HYDRATE_EVENTS = 1 << 5,  // 32 表示带有事件监听器的节点
  STABLE_FRAGMENT = 1 << 6,   // 64 一个不会改变子节点顺序的 Fragment
  KEYED_FRAGMENT = 1 << 7, // 128 带有 key 属性的 Fragment
  UNKEYED_FRAGMENT = 1 << 8, // 256 子节点没有 key 的 Fragment
  NEED_PATCH = 1 << 9,   // 512  表示只需要non-props修补的元素 (non-props不知道怎么翻才恰当~)
  DYNAMIC_SLOTS = 1 << 10,  // 1024 动态的solt
  DEV_ROOT_FRAGMENT = 1 << 11, //2048 表示仅因为用户在模板的根级别放置注释而创建的片段。 这是一个仅用于开发的标志，因为注释在生产中被剥离。
 
  //以下两个是特殊标记
  HOISTED = -1,  // 表示已提升的静态vnode,更新时调过整个子树
  BAIL = -2 // 指示差异算法应该退出优化模式
}
```

### HoistStatic 静态提升
- 将静态节点的定义,提升到父作用域,缓存起来
- 多个相邻的静态节点,会被合并起来
- 典型的拿空间换时间的优化策略


1. 将静态节点的定义,提升到父作用域,缓存起来
```
<div>
  <span>Hello World</span>  
  <span>Hello World1</span>  
  <span>Hello World2</span>  
  <span>{{h}}</span>  
  
</div>

import { createElementVNode as _createElementVNode, toDisplayString as _toDisplayString, openBlock as _openBlock, createElementBlock as _createElementBlock } from "vue"

const _hoisted_1 = /*#__PURE__*/_createElementVNode("span", null, "Hello World", -1 /* HOISTED */)
const _hoisted_2 = /*#__PURE__*/_createElementVNode("span", null, "Hello World1", -1 /* HOISTED */)
const _hoisted_3 = /*#__PURE__*/_createElementVNode("span", null, "Hello World2", -1 /* HOISTED */)

export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createElementBlock("div", null, [
    _hoisted_1,
    _hoisted_2,
    _hoisted_3,
    _createElementVNode("span", null, _toDisplayString(_ctx.h), 1 /* TEXT */)
  ]))
}

// Check the console for the AST

```

2.多个相邻的静态节点,会被合并起来

```
<div>
  <span>Hello World</span>  
  <span>Hello World1</span>  
  <span>Hello World2</span>  
  <span>Hello World3</span>  
  <span>Hello World4</span>  
  <span>Hello World5</span>  
  <span>Hello World6</span>  
  <span>Hello World7</span>  
  <span>Hello World8</span>  
  <span>Hello World9</span>  
  <span>Hello World10</span>  
  <span>Hello World11</span>  
  
</div>

import { createElementVNode as _createElementVNode, createStaticVNode as _createStaticVNode, openBlock as _openBlock, createElementBlock as _createElementBlock } from "vue"

const _hoisted_1 = /*#__PURE__*/_createStaticVNode("<span>Hello World</span><span>Hello World1</span><span>Hello World2</span><span>Hello World3</span><span>Hello World4</span><span>Hello World5</span><span>Hello World6</span><span>Hello World7</span><span>Hello World8</span><span>Hello World9</span><span>Hello World10</span><span>Hello World11</span>", 12)
const _hoisted_13 = [
  _hoisted_1
]

export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createElementBlock("div", null, _hoisted_13))
}

// Check the console for the AST
```
### cacheHandler 事件监听缓存
- 缓存方法

 ```
 <div>
  <span @click="btnClick">Hello World</span>
</div>

import { createElementVNode as _createElementVNode, openBlock as _openBlock, createElementBlock as _createElementBlock } from "vue"

export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createElementBlock("div", null, [
    _createElementVNode("span", {
      onClick: _cache[0] || (_cache[0] = (...args) => (_ctx.btnClick && _ctx.btnClick(...args)))
    }, "Hello World")
  ]))
}

// Check the console for the AST
 ```


### SSR优化
- 静态节点直接输出,绕过了vdom
- 动态节点还是需要动态渲染

```
<div>
  <span>Hello World</span>
  <span>Hello World</span>
  <span>Hello World</span>
  <span>{{msg}}</span>
</div>

import { mergeProps as _mergeProps } from "vue"
import { ssrRenderAttrs as _ssrRenderAttrs, ssrInterpolate as _ssrInterpolate } from "vue/server-renderer"

export function ssrRender(_ctx, _push, _parent, _attrs, $props, $setup, $data, $options) {
  const _cssVars = { style: { color: _ctx.color }}
  _push(`<div${
    _ssrRenderAttrs(_mergeProps(_attrs, _cssVars))
  }><span>Hello World</span><span>Hello World</span><span>Hello World</span><span>${
    _ssrInterpolate(_ctx.msg)
  }</span></div>`)
}

// Check the console for the AST

```
### tree-shaking
- 编译的时候,根据不同的情况,引入不同的API

## Vite为什么启动快
- 开发环境使用ESMoudle,无需打包
- 生产环境使用rollup,并不会快很多


## composition api 和 react hooks 对比
- 前者setup只会调用一次,而后者函数会被多次调用
- 前者无需 useMemo useCallBack,因为setup只会调用一次;
- 前者无需顾虑调用顺序,而后者需要保证hooks的顺序一致
- 前者reactive + ref比后者useState,要难理解

