# vue3 笔记

## setup如何获取组件实例

- 在setup和其它composition api中是没有this的
- 可通过getCurrentInstance获取当前实例
- 若使用options api可照常使用this

## vue3为什么比vue2快?

- Proxy响应式
- PatchFlag
- HoistStatic
- cacheHandler
- SSR优化
- tree-shaking

### PatchFlag
- 编译模板时,动态节点做标记
- 标记,分为不同的类型,如TEXT,PROPS
- diff算法时,可以区分静态节点,以及不同类型的动态节点

```
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
