---
layout: post
title:  "ReactRouter 离开页面的钩子"
date:   2017-10-25
desc: "ReactRouter 离开页面的钩子"
keywords: "mancoxu,javascript,react,react router"
categories: [Javascript]
tags: [Javascript]
icon: icon-html
---



```javascript

componentDidMount() {
    this.props.router.setRouteLeaveHook(
        this.props.route,
        this.routerWillLeave
    )
}

routerWillLeave(nextLocation) {
    return '确认要离开？';
}

```

### 用此方法需要注意：

1. 页面跳转需要由react-router api 方法实行，原生a标签跳转此方法会失效

2. 当前组件需要是react-router 里的链接页面一级组件


### 实用例

```js

import React from 'react';
import {connect} from 'react-redux';
import {bindActionCreators} from 'redux';
import {withRouter} from 'react-router'

class TestComp extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
        };
    }
 	
    componentDidMount() {
        this.props.router.setRouteLeaveHook(
            this.props.route,
            this.routerWillLeave
        )
    }

    routerWillLeave(nextLocation) {
	    return '确认要离开？';
    }
	
    handleChange(){
        this.props.router.push({
            pathname: 'http://mancoxu.me',
            query:{},
        });
    }

    render() {
        return (
            <div>
                <Button type="primary" 
                    onClick={this.handleChange.bind(this)} 
                >
                    点击跳转
                </Button>
                }
                // 失效 <p> <a href={`http://mancoxu.me`}>跳转</a> </p>
            </div>
        );
    }
}

TestComp = withRouter(TestComp);

const mapStateToProps = (state, ownProps) => {
    return {};
};

const mapDispatchToProps = (dispatch) => {
    return {
    	actions: bindActionCreators(actions, dispatch)
    };
}
export default connect(mapStateToProps, mapDispatchToProps)(TestComp)


```



报错解决链接：<https://stackoverflow.com/questions/39103684/getting-an-error-on-using-setrouteleavehook-withrouter>