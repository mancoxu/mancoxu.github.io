---
layout: post
title:  "关于Redux的一些总结(二)：组件拆分 & connect"
date:   2017-03-04
desc: "关于Redux的一些总结(二)：组件拆分 & connect"
keywords: "mancoxu,javascript,react,redux"
categories: [Javascript]
tags: [Javascript]
icon: icon-html
---

## 组件拆分
--------------------------

在 [关于Redux的一些总结(一)：Action & 中间件 & 异步](/javascript/2017/03/03/关于Redux的一些总结(一)-Action-&-中间件-&-异步.html) 一文中，有提到可以根据 reducer 对组件进行拆分，而不必将 state 中的数据原封不动地传入组件，可以根据 state 中的数据，动态地输出组件需要的（最小）属性。

在常规的组件开发方式中，组件自身的数据和状态是耦合的，这种方式虽能简化开发流程，在短期内能提高开发效率，但只适用于小型且复杂度不高的SPA 应用开发，而对于复杂的 SPA 应用来说，这种开发方式不具备良好的扩展性。以开发一个评论组件 Comment 为例，常规的开发方式如下：

```javascript

class CommentList extends Component {
    constructor(){
        super();
        this.state = {commnets: []}
    }

    componentDidMount(){
        $.ajax({
            url:'/my-comments.json',
            dataType:'json',
            success:function(data){
                this.setState({comments:data});
            }.bind(this)
        })
    }

    render(){
        return <ul>{this.state.comments.map(renderComment)}</ul>;
    }

    renderComment({body,author}){
        return <li>{body}-{author}</li>;
    }
}

```

随着应用的复杂度和组件复杂度的双重增加，现有的组件开发方式已经无法满足需求，它会让组件变得不可控制和难以维护，极大增加后续功能扩展的难度。并且由于组件的状态和数据的高度耦合，这种组件是无法复用的，无法抽离出通用的业务无关性组件，这势必也会增加额外的工作量和开发时间。

在组件的开发过程中，从组件的职责角度上，将组件分为 **容器类组件(Container Component)** 和 **展示类组件(Presentational Component)**。前者主要从 state 获取组件需要的（最小）属性，后者主要负责界面渲染和自身的状态(state)控制，为容器组件提供样式。

按照上述的概念，Comment应该有两部分组成：CommentListContainer和CommentList。首先定义一个容器类组件(Container Component)：

```javascript

//CommentListContainer
class CommentListContainer extends Component {
    constructor(){
        super();
        this.state = {commnets: []}
    }

    componentDidMount(){
        $.ajax({
            url:'/my-comments.json',
            dataType:'json',
            success:function(data){
                this.setState({comments:data});
            }.bind(this)
        })
    }

    render(){
        return <CommnetList comments={this.state.comments}/>;
    }
}

```

容器组件CommentListContainer获取到数据之后，通过props传递给子组件CommentList进行界面渲染。CommentList是一个展示类组件：

```javascript

//CommentList
class CommentList extends Component {
    constructor(props){
        super(props);
        this.state = {commnets: []}
    }


    render(){
        return <ul>{this.props.comments.map(renderComment)}</ul>;
    }

    renderComment({body,author}){
        return <li>{body}-{author}</li>;
    }
}

```

将Comment组件拆分后，组件的自身状态和异步数据被分离，界面样式由展示类组件提供。这样，对于后续的业务数据变化需求，只需要更改容器类组件或者增加新的展示类业务组件，极大提高了组件的扩展性。

### Container Component

容器类组件主要功能是获取 state 和提供 action，渲染各个子组件。各个子组件或是一个展示类组件，或是一个容器组件，其职责具体如下：

* 获取 state 数据；

* 渲染内部的子组件；

* 无样式；

* 作为容器，嵌套其它的容器类组件或展示类组件；

* 为展示类组件提供 action，并提供callback给其子组件。

### Presentational Component

展示类组件自身的数据来自于父组件(容器类组件或展示类组件)，组件自身提供样式和管理组件状态。展示类组件是状态化的，其主要职责如下：

* 接受props传递的数据；
* 接受props传递的callback；
* 定义style；
* 使用其它的展示类组件；
* 可以有自己的状态(state)。



## 连接器：connect
------------------------------

react-redux 为 React 组件和 Redux 提供的 state 提供了连接。当然可以直接在 React 中使用 Redux：在最外层容器组件中初始化 store，然后将 state 上的属性作为 props 层层传递下去。

```javascript

class App extends Component{

  componentWillMount(){
    store.subscribe((state)=>this.setState(state))
  }

  render(){

    return <Comp state={this.state}
                 onIncrease={()=>store.dispatch(actions.increase())}
                 onDecrease={()=>store.dispatch(actions.decrease())}/>
  }
}

```

但这并不是所推荐的方式，相比上述的方式，更好的一个写法是结合 react-redux。

首先在最外层容器中，把所有内容包裹在 Provider 组件中，将之前创建的 store 作为 prop 传给 Provider。

```javascript

const App = () => {
  return (
    <Provider store={store}>
      <Comp/>
    </Provider>
  )
};

```

Provider 内的任何一个组件（比如这里的 Comp），如果需要使用 state 中的数据，就必须是「被 connect 过的」组件——使用 connect 方法对「你编写的组件（MyComp）」进行包装后的产物。

```javascript

class MyComp extends Component {
  // content...
}

const Comp = connect(...args)(MyComp);

```
`connect` 会返回一个与 store 连接后的新组件。那么，我们就可以传一个 Presentational Component 给 connect，让 connect 返回一个与 store 连接后的 Container Component。

connect 接受四个参数，返回一个函数：

```javascript

export default function connect(mapStateToProps, mapDispatchToProps, mergeProps, options = {}){
        //code

        return function wrapWithConnect(WrappedComponent){
            //other code
            ....
            //merge props
            function computeMergedProps(stateProps, dispatchProps, parentProps) {
                    const mergedProps = finalMergeProps(stateProps, dispatchProps, parentProps)
                    if (process.env.NODE_ENV !== 'production') {
                           checkStateShape(mergedProps, 'mergeProps')
                    }
                   return mergedProps
             }

                  ....

          render(){

                  //other code
                   ....

                   if (withRef) {
                             this.renderedElement = createElement(
                                     WrappedComponent, {
                                     ...this.mergedProps,
                                     ref: 'wrappedInstance'
                             })
                   } else {
                            this.renderedElement = createElement(
                                    WrappedComponent,
                                     this.mergedProps
                            )
                    }

                    return this.renderedElement
             }

        }
}

```

`wrapWithConnect` 接受一个组件作为参数，在 `render` 会调用 React 的 `createElement` 基于传入的组件和新的 props 返回一个新的组件。

以 connect 的方式来改写Comment组件：

```javascript

//CommentListContainer
import getCommentList '../actions/index'
import CommentList '../comment-list.js';

function mapStateToProps(state){
    return {
        comment: state.comment,
        other: state.other
    }
}

function mapDispatchToProps(dispatch) {
    return {
        getCommentList:()=>{ 
            dispatch(getCommentList());
        }
    }
}

export default connect(mapStateToProps,mapDispatchToProps)(CommentList);

```

在Comment组件中，CommentListContainer 只作为一个连接器作用，连接 CommentList 和 state：

```javascript

//CommentList
class CommentList extends Component {
    constructor(props){
        super(props);
    }

    componentWillMount(){
        //获取数据
        this.props.getCommentList();
    }

    render(){
        let {comment}  = this.props;

        if(comment.fetching){
            //正在加载
            return <Loading />
        }

        //如果对CommentList item的操作比较复杂，也可以将item作为一个独立组件
        return <ul>{this.props.comments.map(renderComment)}</ul>;
    }

    renderComment({body,author}){
        return <li>{body}-{author}</li>;
    }
}

```

关于 connect 比较详细的解释可以参考：[React 实践心得：react-redux 之 connect 方法详解](http://taobaofed.org/blog/2016/08/18/react-redux-connect/)

-------------------------------

原文链接：<https://github.com/dwqs/blog/issues/38>

