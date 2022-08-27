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

### vue2/vue3/react diff算法有什么不同

- diff算法
 给出两组数据,找出需要更新的数据
 比如github的pull request 中的代码diff

- 不同
 react 仅右移
 vue2 双端比较
 vue3 处理前置节点|后置相同节点,剩余生成最长递增子序列以减少最少移动次数
 
 
