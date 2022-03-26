## Module?

JavaScript用'共享一切'的方法加载代码,在ECMAScript-6以前,在应用程序中每个JavaScript中定义的一切都共享一个全局作用域,造成命名冲突和安全问题,并且项目代码难以模块化.Module的出现,就是为了解决一系列问题.

**模块是一个个自动运行在严格模式下并且没有办法退出运行的局部作用域的代码块**

模块系统需要解决的主要问题

① 模块化的问题 

② 消除全局变量

③ 管理加载顺序

注意:

模块顶部创建的变量不会自动添加到全局作用域,未显式导出的变量,函数或类是模块私有的,外部无法访问.

模块的顶部,this的值是undefined.



## Module的导入和导出

- ### module的加载

  使用script标签加载模块时需要添加type='module'

  ```javascript
  <script type='module'> 
    import age from './modules.js';
  </script>
  ```

  

- ### export default 与import

  export 对外输出的值是动态绑定关系,可以取到模块内部实时的值.

  export可以出现在模块的任何位置,但必须是顶级作用域,在块级作用域内会报错

  

  ```javascript
  ///1.
  export default function add(x,y){
      return x + y;
  }
  ///2.也可以是匿名的
  export default function(x,y){
      return x + y;
  }
  ///3
  function add(x,y){
      return x + y;
  }
  export default add;
  ```

  一个module只能有一个export default,其本质是后面的声明,将其赋值给default的变量然后导出,所以可以导出字面量

  ```javascript
  export default 23;
  ```

  Import

  ```javascript
  <script type="module">
          import add from './base.js';
  </script>
  ```

  

- ### export与import

   export 导出后面是变量,函数,类的定义,或者先定义后通过{}包裹进行导出.

   export 一个模块可以有多个

   export输出的变量就是本来的名字,但是也可以用as关键字进行重命名

   import 时也需要{},进行名称绑定,所以不要求顺序一致,也可以用as起别名

```javascript
//1.
export var firstName = 'Michael';

//2.
var lastName = 'Jackson';
var year = 1958;
export {lastName , year};

//3.通常情况下,export输出的变量就是本来的名字,但是也可以用as关键字进行重命名
function v1 (){
    console.log('v1');
}
export{
    v1 as logv1,
    v1 as streamV1
};

import { firstName,lastName,year } from "./export.js";

//起别名
import { firstName as name} from "./export.js";
```



- ### 整体导入 

  ```javascript
  import * as name from './base.js';
  ```

  

- ### import与import()



- ### 导入导出复合写法

导入导出的复合写法

```javascript
export {age} from './profile.js';

//等同于
import {age} from './profile.js';
export {age};
```

复合写法在当前module是无法使用的.



