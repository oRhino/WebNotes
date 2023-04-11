## 与 HTML 的区别

- 首字母大小写的区别，大写是自定义组件
- 标签必须闭合，如 `<input＞` 在 JSX 中是非法的
- 每段 JSX 只能有一个根节点,根节点可以是 Fragment,即 <> </>

## 属性

- class 要改为 className
- style 要使用 JS 对象（不能是 string）而且 key 用驼峰写法
- for 要改为 htmlFor

## 事件

- 使用 onxxx 的形式
- 必须传入一个函数（是 fn 而非 fn0）
