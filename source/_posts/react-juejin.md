---
title: 使用React构建精简版本掘金（一）
date: 2019-03-19 21:06:57
tags:
	- react
	- ant-design
categories: 'react'
---

最近正在学习react，就想着能不能用react做一个项目，平时浏览掘金，就拿掘金练手吧！

<!--more-->

![](https://user-gold-cdn.xitu.io/2019/3/19/169960b16660381f?w=1558&h=1036&f=png&s=155113)
是不是可以以假乱真呢！😂😂😂
## 初始化

* 使用create-react-app初始化项目结构

```
yarn create react-app react-juejin
```
这个脚手架会自动帮助我们搭建基础工程，同时安装React项目的各种必要依赖，如果在过程中出现网络问题，请尝试配置代理或使用其他 npm registry。  
进入项目并启动

```
cd react-juejin
yarn start
```

* 安装ant-design

```
yarn add antd
```

* 配置UI库懒加载样式  
  需要对整个项目重新配置，这里使用了[react-app-rewired ](https://github.com/timarney/react-app-rewired)（一个对 create-react-app 进行自定义配置的社区解决方案）。
```
yarn add react-app-rewired customize-cra
```
修改package.json 文件如下

![](https://user-gold-cdn.xitu.io/2019/3/18/1698f9da727831a1?w=575&h=172&f=png&s=23659)
在根目录中创建config-overrides.js，用于重写覆盖默认的配置

```
module.exports = function override(config, env) {
  // do stuff with the webpack config...
  return config;
};
```

* 使用 babel-plugin-import
  该插件用于按需加载plugins和样式

```
yarn add babel-plugin-import
```
修改上步创建的config-overrides.js

```
const { override, fixBabelImports } = require('customize-cra');

module.exports = override(
    fixBabelImports('import', {
        libraryName: 'antd',
        libraryDirectory: 'es',
        style: 'css',
    })
);
```

* 添加less-loader  
  个人习惯使用less，看个人喜好安装即可，不过查阅上面社区方案react-app-rewired，并没有提供比如sass的重写方案，故如果需要使用sass，可采用别的方案引入。

```
yarn add less less-loader
```
修改config-overrides.js

```
//const { override, fixBabelImports } = require('customize-cra');
const { override, fixBabelImports, addLessLoader } = require('customize-cra');

module.exports = override(
    fixBabelImports('import', {
        libraryName: 'antd',
        libraryDirectory: 'es',
        style: true,
    }),
    addLessLoader({
        javascriptEnabled: true,
    }),
);
```
以上详细配置的话可参考[ant-design官网](https://ant.design/docs/react/use-with-create-react-app-cn)

## 引入redux


* 安装
```
yarn add redux react-redux --save
```

* 使用方式
  考虑到之后可能会有多个reducer，开始就把结构弄好，做成日后可以方便合并使用多个reducer的方式  
  （1）创建一个reducer

```
// 建议使用这中结构

// 1.定义默认数据
let initialState = {
    notificationCount: 0
}

// 2.Reducer
const pageHeaderReducer = (state = initialState, action) => {
    switch (action.type) {
        case 'CHANGE_COUNT':
            return { ...state, notificationCount: action.notificationCount }
        default:
            return state
    }
}
// 3.导出
export default pageHeaderReducer;
```
（2）创建index.js,作为合并所有reducer的文件。

```
import {combineReducers} from 'redux';

import pageHeaderReducer from './pageHeader.js';

const appReducer = combineReducers({
    pageHeaderReducer
});
export default appReducer;
```
（3）App.js中使用定义好的reducer

```
import { createStore } from 'redux'
import { Provider } from 'react-redux'
import appReducer from './reducers/index.js';
// 使用合并后的那个Reducer
const store = createStore(appReducer);
class App extends Component {
  constructor(props){
    super(props);
  }
  ...
  render() {
    return (
      <Provider store={store}>
        <div className="App">
          ...
        </div>
      </Provider>
    );
  }
}
```
(4)在header/index.js中使用redux

```
import { connect } from 'react-redux';

class Header extends Component {
    ...
    render() {
        ...
        return (
            <Affix offsetTop={this.state.top}>
                ...
                <Badge count={this.props.count} overflowCount={10}>
                    <a href="/">
                        <Icon type="notification" />
                    </a>
                </Badge>
            </Affix>
        );
    }
}

const mapStateToProps = (state) => {
    return {
        count: state.pageHeaderReducer.notificationCount
    }
}

Header=connect(mapStateToProps)(Header)

export default Header;
```
到目前为止，就可以在外部修改notificationCount的值，通过redux，组件内部就可以正常获取到对应的count值。  
更详细的redux配置可以参考[redux中文文档](http://cn.redux.js.org/)

## 添加路由react-router
首页导航中存在5个tab切换，分别对应这不同的页面内容。接下来介绍如何通过react-router实现不同页面内容的跳转。

* 安装react-router

```
yarn add react-router-dom --save
```

* 使用方式

```
import { Switch, Route } from 'react-router-dom';
...
class Main extends Component {
    constructor(props) {
        super(props);
        this.state = {  }
    }
    render() {
        return (
            <div>
                <Switch>
                    <Route exact path='/' component={Home}/>
                    <Route path='/timeline' component={Home}/>
                    <Route path='/dynamic' component={Dynamic}/>
                    <Route path='/topic' component={Topic}/>
                    <Route path='/brochure' component={Brochure}/>
                    <Route path='/activity' component={Activity}/>
                </Switch>
            </div>
        );
    }
}
```
**上面的exact表示绝对匹配/,如果不注明exact,则/还会匹配/timeline等等上面代码实现了一个类似tabbar切换的效果**
* tab导航

```
render() {
        return (
            <ul>
                {this.state.navs.map((item,index)=>{
                    return <li key={item.path} className={item.isActived?'activeLi':''} onClick={this.handelClick.bind(this,index)}>
                                <Link to={item.path}>{item.text}</Link>
                            </li>
                })}
            </ul>
        );
    }
```
**react-router中提供了Link和NavLik两种方式，如果仅仅需要匹配路由,使用Link就可以了,而NavLink的不同在于可以给当前选中的路由添加样式, 比如上面写到的activeStyle和activeClassName**  
更详细的react-router配置可以参考[React-router中文文档](https://react-guide.github.io/react-router-cn/index.html)

到目前为止，基础结构就算是完成了，后续的就需要往各个页面添加实际内容了。


![](https://user-gold-cdn.xitu.io/2019/3/19/16993c311993d6ca?w=1398&h=322&f=gif&s=130255)


目前效果图如上所示，后续不断更新中。以上详细代码见[github](https://github.com/rocky-191/react-juejin),欢迎点赞，您的点赞是我的动力。