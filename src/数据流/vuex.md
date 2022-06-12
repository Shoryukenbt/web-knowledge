## Vuex

### Vuex是什么
Vuex是一个匹配Vue.js的状态管理模式，采用集中式的储存和管理组件的所有状态，并以相应的规则保证状态变化的可预测。

### Vuex解决了哪些问题
1. 多个组件共享状态的管理，不在需要层层传递
2. 使变更可追溯

### vuex的核心概念
1. `state`, vuex使用单一状态树，数据`data`必须是纯粹的跟Vue中的Data保持一致。可以使用`mapState`函数辅助绑定state到Vue的computed。
2. `getter`, 类似于store的计算属性，用于派生state中的状态，同样支持使用`.mapGetter`函数辅助绑定到computed。
3. `mutation`, vuex中唯一修改状态的途径就是提交`mutaion`, 每个mutation都包含一个名称和回调函数, 回调函数的参数是state, 通过`store.commit(mutationName, payload)`调用。 mutation必须是同步函数。 `mapMutation`辅助函数、支持将mutation映射到method。
4. `action`, action中可以执行异步操作，再通过调用mutaiton更新state, 通过`store.dispatch(actionName, payload)`调用，同样支持辅助函数`mapActions`
5. `module`, 为了避免store过于臃肿，可以将store分割成模块，每个模块拥有自己的state, getter, mutation, action, 通过`rootState`访问跟节点。mation, action, getter默认注册在全局命名空间，