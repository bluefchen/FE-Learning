#### React child
1. 任何东西都能是一个child
2. API
   - React.Children.map
   - React.Children.forEach
   - 计数使用 React.Children.count
   - 转换为数组：React.Children.toArray
   - 克隆元素：React.cloneElement(元素，｛属性｝)

#### key
1. 碰到数组---->列表的映射，或者同级元素需要移位的情况，要给元素加上key属性
2. 使用唯一id作为key比使用数组的index作为key性能更好。用index的时候，[a,b]--->[b,a]的时候，发现key是不变的，但是实际的节点类型不同，接下来会新建两个dom节点去替换原来的节点，相当于key没有用上。

#### setState
1. setState不会立即更新state，也不会立马render
2. 多次的setAtate会合并，shoudComponentUpdate->componentWillUpdate->render->componentDidUpdate
3. 直到下次rende函数调用时，才会更新this.state
4. 原生事件中的setState / setTimeOut中进行setState与普通setState不同，前者会立马render.即使是同一个定时器中的多次setState
5. 函数式的setState，参数（当前state，当前props），使用函数式`setState(()=>{})`时，React会保证调用每次function时，state都已经合并了之前的状态修改结果。