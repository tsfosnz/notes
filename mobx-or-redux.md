## RN 的状态管理

状态管理的两种流行方式

### Mobx

阅读需求

在完成一个使用 Mobx 的项目之后，需要再次了解各方面

- Best practice
- Performance
- Coding style 
- Coding organizaiton

### Redux

Redux 保存所有的状态在一个地方。

不同的状态，通过不同的 reducer 分别管理。

每一类状态的保存的索引名字可以自定义。

每一个新的状态的流程是：

- dispatch 一个 action
- 这个 action 通过自己的 reducer 来返回新的状态
- 新的状态被 cast 到目标组件中
- 注意，action 的名字是全局的，所以每一个都不能相同

{ action, action type, reducer } 是一个组合，可以管理一个 Container

### 如何和组件链接

为了把状态传递到组件内部，传统的方法是：

subscribe store 

就是预定 store 的变化，如果 store 变化了，可以更新本组件

另外 dispatch action 将会带来 store 的变化

推荐的方法是：

把组件分为 Container 和 Presentational 

Container 是和 store 绑定的，类似于 subscriber

Presentational 是通过 Container 获得新的状态，实际还是使用 props 来获得

为了把 Container 和 store 链接起来

connect 支持 state 的变化 mapStateToProps ，以及 dispatch 的变化，方便组件发送 action 的时候用。

因为 store 只有一个，所以，Provider 只有一个，所有的 组件都通过 最外部的那个 Container 链接。

### 问题

- redux 里面可以使用多个 container ，每个 container 使用一个独立的 store 么？看似可以
- 对于 component 里面的内部 state ，可能就不需要 redux 管理了


### ReactJS + Semantic UI

试用了 Web UI 开发的组件，确实非常快。

很快就可以做出来很好的页面，只要按照良好的设计走，就能得到良好的体现。

并且，和后端代码的接口也非常简单。

这样，写 Admin 或者是什么就简单容易了，如果再加上 GraphQL ，就比较清晰。

### Parse + GraphQL


### React Relay



 


