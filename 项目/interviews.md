## HTML 
### HTML常见元素
- head  
  - meta  
  - title
  - style
  - link
  - script
  - base
  
- body
  - div/section/article/aside/header/footer
  - p
  - span/em/strong
  - table/thead/tbody/tr/td
  - ul/ol/li/dl/dt/dd
  - a
  - form/input/select/textarea/button

### HTML语义化
- 让人更容易读懂 增加代码可读性
- 让搜索引擎更容易读懂 SEO

### 块状元素 & 内联元素

- display:block/table div,h1-h6,table,ul,ol,p等
- display:inline/inline-block; span,img,input,button等


## 框架

### Vue每个生命周期都做了什么
- beforeCreate
  1. 创建一个空白的vue实例
  2. data,method尚未被初始化,不可使用
  
- created
  1. Vue实例初始化完成,完成响应式绑定
  2. data,method都已经初始化,可调用
  3. 尚未开始渲染模板

- beforeMount
  1. 编译模板,调用render 生成Dom
  2. 还没有开始渲染DOM

- mounted
 1. 完成组件渲染
 2. 组件创建完成
 3. 开始用'创建阶段' 进入 '运行阶段'

- beforeUpdate
 1. data发生变化之后
 2. 准备更新DOM(尚未更新DOM)

- updated
 1. data发生变化,且Dom更新完成
 2. 不要在(Updated中修改data,可能会导致死循环)

- beforeUnmount
 1. 组件进入销毁阶段(尚未销毁,可正常使用)
 2. 可移除,解绑一些全局事件,自定义事件.

- unmounted
1. 组件被销毁了
2. 所有的子组件也都被销毁了

- keep-alive组件
1. onActivated缓存组件被激活
2. onDeactivated缓存组件被隐藏

```js
// 挂载
父组件 beforeCreate
父组件 created
子组件 beforeCreate
子组件 created
子组件 mounted
父组件 mounted

// 更新
子组件 updated
父组件 updated

// 卸载
父组件 beforeDestroy
子组件 beforeDestroy
子组件 destroyed
父组件 destroyed

```

### Vue什么时候操作DOM比较合适
- mounted和update都不能保证子组件全部挂载完成
- 使用$nextTick渲染DOM

### vue2/vue3/react diff算法有什么不同

- diff算法
 给出两组数据,找出需要更新的数据
 比如github的pull request 中的代码diff

- 不同
 - react 仅右移
 - vue2 双端比较
 - vue3 处理前置节点|后置相同节点,剩余生成最长递增子序列以减少最少移动次数
 
### Vue/React为什么使用key
- diff会根据key是否相同判断元素是否要删除
- 匹配了key,则只移动元素,性能较好
- 未匹配到key,则删除重建,性能较差

### Vue组件通信方式
- props/$emits(父子组件)
- 自定义事件(eventbus,event-emitter,vue2-on/off/emits...)(全局组件) 
- $attrs/$listener(vue3移除了$listener,合并到$attrs)(父子)
- $parent/$refs(父子)
- provide/inject(跨层级)
- vuex(全局组件) 


 
