

### 相比vue2新增

## 可复用 & 组合
#### 组合式API -- setup(props, context)
> 是为了更好地复用逻辑I, context = {atttrs, slots, emit, expose}

1. setup运行在实例创建之前，所以无法获取实例的属性
2. es6结构会失去props属性的`响应性`, 使用`解构`时，应使用toRefs, 但是对于props中不存在的属性，toRefs无法使其可响应，这时应该使用`toRef`, 如：

    > ```
    > setup(props, context){
    >   const { title } = props; // title无响应性
    >   const { title } = toRefs(props); // 正确写法
    >   const { title } = toRef(props, 'title'); //props中初始不存在的属性
    >   console.log(title.vaule/*或不使用解构：props.title*/);
    >}
    > ```
3. 在setup中使用 provide, inject
4. 调用生命周期： `on + 生命周期钩子`, eg: `onMount`
5. setup返回的的Ref与模板保持一致
6. 模板侦听器应该使用`flush: post`, 才能保持Dom是已经更新的

#### Mixin


#### diff算法
对于vnode的children，vue使用双端diff算法, 实现复用节点

