## JS基本类型，位置

5种基本类型: Number, String, Boolean, Undefined, Null

基本类型在栈中，还包括 引用类型的地址

引用类型在堆中： （Ojbect , Array, Function)

栈中：读取速度更快，但受到空间和生命周期的限制。占空间小，大小固定。属于被频繁使用的数据。

堆中：主要存储引用对象。 占据空间更大，大小不固定。 引用数据类型在栈中存储了指针，查找也快。



## css所有居中问题
### 1. flex布局：
     这种只能是父级为弹性容器， `justify-content:center; align-items:center;display:flex;    `
       居中元素设置为 `margin: auto;`
### 2. 宽高不定时：
     父元素`display:relative;`
       居中元素`display:absolute;top:50%;left:50%;**transform:translate**(-50%,-50%);`
### 3. 宽高定时：

 #### 方法1：使用margin 50%
- 父元素：`display:relative;`

  居中元素：`display:absolute;top:50%;left:50%;height:100px;width:100px;margin-left:-50px;margin-top:-50px;`
 #### 方法2：使用calc
- 父元素： `display:relative;`

  居中元素：`display:absolute; width:100px; height:100px; left:calc(50%-50px); top:calc(50%-50px);`margin-top/left的值为父级元素值的50%-自身宽高的50%;

4. 使用margin/padding值去设置居中；

