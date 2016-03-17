# 前端面试题
---

## web前端优化
> * JS/CSS：
        JS会阻塞页面渲染，放在页面最下方；
        CSS在页面顶部加载。避免CSS表达式
        合并重用相同的代码块，对文件进行合并压缩，减少HTTP请求数
> * 图片：合并压缩图片，减少HTTP请求数
> * 页面优化：
        结构语义化，优化body里面的标签
        优化 `title`， `description`， `keywords` 等
> * 服务端：
        开启 `gzip` / `bzip2` 压缩
        使用HTTP的`keep-alive`减少连接数
        开启 `Etags`，实体标签，进行页面缓存
        开启 `APC`， `opcode` 缓存
        `memcache` / `redis` 数据缓存
        代码优化

---  
## ajax跨域
> * 代理
> * `dataTpye: "jsonp"`
> * `header("Access-Control-Allow-Origin:*");`
> * jquery  `getJson()`

---
## 事件捕获/事件冒泡
* 捕获由父级到子级
* 冒泡由子级到父级  
``` element.addEventListener(event, function, useCapture); ```    

> `useCapture`  
> true - 事件句柄在捕获阶段执行    
> false- false- 默认。事件句柄在冒泡阶段执行    

* 事件按顺序绑定

---
## 事件委托/事件绑定
* 委托在处理速度和内存占用上都优于事件绑定
* 但委托会在事件冒泡中造成性能损失  

> 委托对未来元素有效，事件绑定只对当前已有元素有效，对脚本后来生成的元素无效

``` on > delegate > live > bind ```   

> * `bind` 直接绑定，只对现有元素有效
> * `live` 通过冒泡匹配到对应的元素，对未来有效
> * `delegate` 相对于 `live` 更精确
> * `on` 是以上几种的综合体  

---  
## 立即执行函数  
``` (function(){}()) ```  
``` (function(){})() ```  
只用于函数表达式  例：  
``` var kof = function(){}() ```  
> 可以模仿出一个私有作用域，不会遗留全局变量，不会污染全局空间，用于JS模块化编程，又被称作“匿名包裹器”、“命名空间”  

---  
## 阻止冒泡  
> * ``` return false ```  
> * ``` event.target == event.currentTarget ```    
> * ``` event.stopPropagation ```    
> * ``` event.preventDefault ```    

---  
## 为什么使用闭包  
> 进行信息的隐藏和封装，模拟一些 OO 的特性，避免变量污染  

---  
## 变量提升（Hoisting）   
> 会将变量声明过程提升到顶部  
``` javascript  
    var a = 6;
    setTimeout(function () {
        alert(a);
        var a = 666;
    }, 1000);
    a = 66;  
    // undefined    
```   

---  
## 延迟加载  
**`setTimeout()` / `clearTimeout()`**  
> 加入到队列末执行  
``` javascript  
    var kof = 6;
    setTimeout(function () {
        alert(kof);
    }, 1000);
    a = 66;  
    // 66  
```
> 用于清除 jQuery 操作  

**`setInterval()` / `clearInterval()`**   
> 用于计时器操作  

---   
## 模块化编程思想  
> 模块化编程，低内聚，高耦合  
> 保证各模块之间的独立性，使各模块之间的依赖关系变的更加明确  
  
* 原始写法：  
``` javascript   
    function module () {
    
    }   
```  
> 污染全局变量，变量名可能冲突，各模块之间依赖关系不明确  

* 面向对象：  
``` javascript  
    var module = new Object({
        
    });  
```  
> 会暴露模块成员变量，能够从外部修改  

* 立即执行函数：  
``` javascript  
    var module = function () {
        var _sum = 0;
        var init = 100;
        
        var count = function () {
            return _sum + 1;
        };
        
        return {
            init: init;
            count: count;
        };
    }();  
```  
> 标准模块写法，只能访问 return 的值  

* 继承：  
``` javascript  
    var module = function (mod) {
        mod.k1 = function () {
        
        }();
        return mod;
    }(module);  
```    
* 防止空对象报错  
``` javascript  
    var module = function (mod) {
        mod.k1 = function () {
        
        }();
        return mod;
    }(module || {});  
```    
* 输入全局变量  
``` javascript  
    var module = function ($) {
        
    }(jQuery);  
```     
> 显示输入其他模块  

---   
## CommonJS 规范  
> 同步加载模块，如果模块太大会阻塞渲染  
`seaJS / CMD`  
  
---  
## AMD 规范  
> 异步模块定义  
> `require.js / curl.js`  
``` javascript  
    <script src="js/require.js" defer async="true"></script>  
```   
``` require([module], callback); ```

> `defer` : HTML4 中的异步加载
> `async` : HTML5 中的异步加载，不阻塞渲染  
  
  
  
  
---  

---  
## CSS 模块化  
> 配合 `less / sass / stylus` 实现 css 的封装继承多态  

---  
## 定位  
> 不要总是用 `float`  
> `float` 后注意 `clear`  
> `position: relative;`   相对于前一个父级元素，用 `margin` 调整  
> `display: inline` / `inline-block`  

---  
## 标签比较  
`<b>` 与 `<strong>`, `<i>` 与 `<em>`  
> 一个是物理标签，一个是逻辑标签，前者强调的是物理行为，后者强调了当前环境的语义，更符合`标签语义化`，更符合w3c标准   

`<h1>` 与 `<title>`  
> SEO 时，`title` 权重高于 `h1`  

`<alt>` 与 `<title>`  
> `alt` 主要是为图像未加载时做的说明（只用于 `img` `area` `input`）  
> `title` 用于文字或链接加注释  
 
---  
## 













