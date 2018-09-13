#### js url编码三种方法

escape, encodeRUI, encodeRUIComponent
unescape, decodeRUI, decodeRUIComponent

编码为有效的统一资源标识符 (URI) 的字符串

#### sessionStorage API
~~~javaScript
sessionStorage.getItem('oneItem') = sessionStorage.oneItem
sessionStorage.setItem(''oneItem',value') = sessionStorage.oneItem = value
sessionStorage.removeItem('')
sessionStorage.clear()
~~~
作用域的不同：localStorag只要在相同协议、相同主机名、相同的端口下，就能读取、修改到同一份localStorage数据。
sessionStorage还需要在同一窗口下
生存期不同：localStorage永久，sessionStorage只要关闭浏览器就没了