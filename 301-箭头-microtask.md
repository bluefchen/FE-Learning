### 301 302
1，301 永久重定向：资源切换位置，没带www跳到带www，换空间

2，302暂时重定向：网站劫持（搜索引擎，默认判断a--->b   ，判断a为高使用网站，从而劫持b的浏量



### http-only指的是cookies只读。

csrf方法：1，加token。2，将token放到cookies，设置cookies为http-only



### 箭头函数区别：
1. 不绑定this，获取它所在的上下文的this值，作为自己的this
2. 没有原型与constructor,不能用new
3. 没有arguments，通过rest参数...
4. 不能通过call , bind , apply 改变this的指向

### macrotast与microtast
1. microtask包含Promise, process.nextTick   当microtask被推到执行中，那主线程执行完后，会循环调用队列任务中的下一个任务。
	macrotask每一次事件循环只会执行一次，microtask会一直提取，直到microtask为空。
2. macrotast包含setTimeout，setInteral，setImmediate， I/O, UI rendering  执行完后
