1. Web UI 中 DOM 节点跨层级的移动操作特别少，可以忽略不计。
2. 拥有相同类的两个组件将会生成相似的树形结构，拥有不同类的两个组件将会生成不同的树形结构。
3. 对于同一层级的一组子节点，它们可以通过唯一 id 进行区分。



Diff两个节点有三种情况：不同类型的节点、DOM节点(class相同)、component(class相同)类型。

tree diff , component diff

   **tree diff**  

- 不同类型的节点：React不会进行任何比对，直接将旧节点扔掉，嵌入新节点。
- 同类型的节点：线性时间内可以对比DOM节点的属性(attributes)变化。

 

**component diff**



- 不同类型的节点：直接删除 
- 节点单类型相同，属性不同 ：React会将新component的属性传递给旧component的component[Will/Did]ReceiveProps()参数，然后调用render来重新渲染。 

节点处于同一层级时，React diff 提供了三种节点操作，分别为：**INSERT_MARKUP**（插入）、**MOVE_EXISTING**（移动）和 **REMOVE_NODE**（删除）。 