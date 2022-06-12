## 概览

### class组件

1. 组件构成

    `props`: 从父组件传递的参数，是响应式的

    `data`：组件自身的状态对象，为了避免污染原型，声明为Function, 并返回对象，该对象是响应式的，也仅当初始存在的`key`是响应式的，对于初始不存在的值，应该使用`null`, `undefined`站位，或者后期使用vm.$set(key, val)添加

    `methods`: 挂载与vm实例的一组方法，方法的this与当前实例绑定

    `computed`: 具有缓存特性的一组方法，调用时使用属性的方式调用

    > *对比methods, computed*: methods为普通方法，使用方法调用，如: `vm.fn()`, 每次调用都重新执行；而computed为属性调用, 如`vm.attr`, 并且结果根据依赖缓存，依赖为改变不会执行，直接返回缓存的结果  



2. 组件的生命周期

    beforeCreate, created, beforeMount, mounted, beforeUpdate, updated, actived, beforeDestroy, destroyed

3. 内容分发
    插槽slot, 作用于插槽scoped-slot(绑定了自组建的变量，使父组件能够访问到)

4. `provide`, `inject`解决数据嵌套传递的问题

### 重用与组合

1. 全局组合 Vue.mixin(obj)

2. 局部组合 class {mixins: [...mixins]}
    > 使用`Vue.config.optionMergeStrategies= function(to, from){}`来解决冲突属性

3. 插件 `{ install(Vue, options) {} }`, 插件必须包含install方法，Vue.use(plugin, opiton), 实际上是调佣plugin.install方法
    
## 响应式原理

1. 解析`data`中属性，使用Object.define劫持getter, setter, 生成响应式数据
2. 解析`template`获取对数据(props, data)的get, 注册对该属性的`watcher`, 当该属性值改变时通知watcher, 更新响应的vtrilDom, 最终更新dom
3. Vue更新Dom是使用异步队列更新，而是在下一个`tick`中完成更新，这里用到了`nextTick()`方法, 这是一个使用Promise.then, MutationObserver, setImmediate实现的

## Vue原理

### Dep 类说明
Dep类就是一个可观察对象，可以被多个`watcher`订阅, `notify`广播的时候依次调用订阅了该对象的`watcher`对象的  `update`方法
### Observer 类说明
1. 该具有value, dep, vmCount三个成员变量，调用过构造函数时就会对value根据其类型分别递归调用`observeArray`和`walk`发方法，对value的每一个key实现响应式; 同时为了对Array实现响应式，对Array的原型进行了重写，修改了`push`, `pop`, `shift`, `unshift`, `splice`, `sort`, `reverse`方法，使vue能够获取数组元素的变更，但是能力受到限制，导致我们操作Array除了使用上述方式，均不会被Vue接收变更无法实现响应式，例如使用下表直接操作元素，使用length删除元素之类的操作，
2. 对对象实现响应式的方法是对简单对象每一个`key`调用`defineReactive`函数, 函数使用Object.defineProperty定义了对该属性的getter, setter。调用`getter`时创建一个dep实例, 并让Dep.target(watcher)订阅该dep; 在掉用`setter`时, 掉用`dep.notify()`来通知watcher执行更新操作

### Watcher 类说明
1. 构造函数constructor接受一个vm实例，一个表达式或者函数，并在获取被观察对象时根据该表达式或者函数进行解析，一个回调函数，options, isRenderWatcher
2. 依据expOrFn的类型初始化`getter`函数，如果是`function`类型，直接复制给getter, 否则创建解析函数`parsePath`，获取value时就调用this.getter
3. 根据`lazy`的值获取value值或者延迟
4. watcher实例具有get函数，该函数执行时会将Dep.target赋值成watcher实例，并在执行完毕时恢复Dep.target, 这样做的作用是因为我们在调用响应式数据的getter时，会对当前getter的key创建一个dep实例, 并将该实例添加到Dep.target的`newDeps`中，并让Dep.target订阅dep实例，因为调用get的过程中会访问数据的gtter, 也就会重新订阅dep，及执行上述过程，所以执行完成后还需要调用cleanupDeps来清理旧的deps
4. watcher实例具有update函数，该函数的执行是将lazy-wacther的dirty设置为true, 或者直接调用sync-watcher的run方法，否则调用queueWatcher方法将watcher实例添加到scheduler的queen中，等待在nextTick中被执行，所以这也是vue的dom更新是异步的表面原因，同时因为入列的时候依据watcher的引用做了去重处理，那么同一个vue实例在一个宏任务内的多次修改也只会执行一个更新
5. watcher实例的`run`方法，执行过程是调用`get`方法获取最新的值，并判断和oldValue有差异则调用回调函数
### 响应式实现总结
原理是使用数组原型方法重写和Object.defineProperty对普通对象进行访问劫持，访问getter时收集依赖，访问setter时发出通知

具体流程是：
1. `Vue`实例化时`mountElement`函数中，会生成`updateComponent`函数，初始化Vm实例的`_watcher`, 并将`updateComponent`函数作为`Watcher`构造函数的第二个参数`expOrFn`, 这样`updateComponent`函数马上会被执行，并且其运行`'_patch_`函数生成`VNode`时会访问数据的`getter`, 根据前面的描述，这样便完成了依赖的收集。
2. 收集后的状态是: a.VueComponent具有`_watcher`属性，这是一个`Watcher`实例，并且订阅了`template`中使用的每一个数据的key对应的dep; 这样当数据发生变化时会触发dep的notify,从而执行`_watcher`的`update`方法, `update`执行是又会调用已经被设置为`get`的`updateComponent`方法，所以也会调用`vm._update`完成更新，并同时完成新视图依赖的收集, 等待下一次收到通知再重复这个过程

### computed和watcher
1. computed的原理也是使用Watcher, 将key定义到vm实例上，并禁用setter, 和普通响应式对象不同的是，每次get都是先检测脏值，有脏值则计算最新的结果并返回，否则返回上次计算的结果
2. watcher原理是针对每一个key创建一个watcher实例，定阅解析`expOrFn`过程中产生的依赖，并在接收到`notify`时运行回调函数

### diff算法
1. oldHead和newHead对比，如果`sameNode(oldHead, newHead)`, 则调用patchNode(oldHead, newHead), 并且oldHead和newHead下移
2. oldTail和newTail对比，如果`sameNode(oldTail, newTail)`, 则调用patchNode(oldTail, newTail)，并且oldTail和newTail上移
3. oldHead和newTail对比，如果是`sameNode`, 则移动oldHead.elem到oldTail.elem之后, 并调用`patch(oldHead, newTail)`, 并且oldHead下移，newTail上移
4. oldTail和newHead对比，如果是`sameNode`, 则移动oldTail.elem到oldHead.elem之前, 并调用`pathNode(oldTail, newHead)`, 并且oldTail上移，newHead下移
5. 以上都没有匹配则遍历oldHead和oldTail之间的VNode, 找到和newHead匹配的OldNode, 若果找到匹配项则移动匹配的oldNode.elem到oldHead.elem之前, 并调用`patchVNode(matchOldNode, newHead)`, 设置old匹配位置为null, newHead下移; 否则调用`createElement`，并将创建的节点插入到`oldHead`之前，newHead下移
6. 遍历过程中会遇到第5部设置为`null`的`oldNode`, 此时执行`oldHead`下移或者`oldTail`上移
7. 当old或者new的head > tail时，终止遍历。此时如果old还有剩余，则对`oldHead`到`.oldTail`之间的VNode执行removeVNodes操作; 如果new还有剩余，则对`newHead`到`newTail`之间的节点执行`addVnodes`操作，若果当前`newTail`则将新创建elem使用`append`插入到parent.children的尾部，否则使用`insertBefore`插入到`newTail.elem`之前

### VNode
1. VNode是什么？ VNode是对虚拟Dom的一种抽象，描述真是dom的微结构以及和Vue Component的关系，所有对Dom的修改都先更新对应的Vnode, 在通过Vnode实现最小更新
2. VNode怎么创建的？ VueComponent调用render函数返回VNode
3. Vnode有什么用？ Vue在patch的时候，通过VNode创建(更新)真实DOM，并且在更新时通过比较VNode，达到最小化更新真实DOM的目的