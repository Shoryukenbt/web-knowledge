## Vue-router

### 钩子函数

| 范围 | hook | 调用时机 | 说明 |
|--|--|--|--|
|全局|beforeEach((to, from, next) => {})|触发导航时按顺序调用|1. `return false`可终止导航，2. 必须确保`next`最多调用一次, 3. 异步解析， 4. 返回路由地址重定向|
|全局|afterResolve(async to=>{})|导航被确认之前，组件内守卫和异步组件被解析之后| 1. `return false`终止导航 2. 是获取数据操作的理想位置|
|全局|afterEach((to, from, failure) => {})|||
|单个路由|beforeEnter(to, from)|只在不同路由导航进入路由时触发|支持数组依次调用,`beforeEnter:[actionA, actionB]`|
|组件|beforeRouteEnter(to, from)|在渲染该组件的对应路由被验证前调用|不能调用`this`|
|组件|beforeRouteUpdate(to, from)|在当前路由改变，但是该组件被复用时调用||
|组件|beforeRouteLeave(to, from)|在导航离开渲染该组件的对应路由时调用||

### 完整的导航解析流程
1. 导航被触发。
2. 在失活的组件里调用 beforeRouteLeave 守卫。
3. 调用全局的 beforeEach 守卫。
4. 在重用的组件里调用 beforeRouteUpdate 守卫(2.2+)。
5. 在路由配置里调用 beforeEnter。
6. 解析异步路由组件。
7. 在被激活的组件里调用 beforeRouteEnter。
8. 调用全局的 beforeResolve 守卫(2.5+)。
9. 导航被确认。
10. 调用全局的 afterEach 钩子。
11. 触发 DOM 更新。
12. 调用 beforeRouteEnter 守卫中传给 next 的回调函数，创建好的组件实例会作为回调函数的参数传入。
