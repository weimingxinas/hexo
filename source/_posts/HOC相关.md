---
title: HOC相关
date: 2019-07-06 14:16:50
tags: components
---
1. 组合渲染

```javascript
// 通过属性代理实现
function stylHOC(WrappedComponent) {
  return class extends Component {
    render() {
      return (<div>
        <div className="title">{this.props.title}</div>
        <WrappedComponent {...this.props} />
      </div>);
    }
  }
}
```
2. 条件渲染  

```javascript
function visibleHOC(WrappedComponent) {
  return class extends WrappedComponent {
    render() {
      if (this.props.visible === false) {
        return null
      } else {
        return super.render()
      }
    }
  }
}
```
3. 操作props
```javascript
function proxyHOC(WrappedComponent) {
  return class extends Component {
    render() {
      const newProps = {
        ...this.props,
        user: 'ConardLi'
      }
      return <WrappedComponent {...newProps} />;
    }
  }
}

```
4. 获取refs

```javascript
function refHOC(WrappedComponent) {
  return class extends Component {
    componentDidMount() {
      this.wapperRef.log()
    }
    render() {
      return <WrappedComponent {...this.props} ref={ref => { this.wapperRef = ref }} />;
    }
  }
}
```
5. 状态管理  
将原组件的状态提取到HOC中进行管理，如下面的代码，我们将Input的value提取到HOC中进行管理，使它变成受控组件，同时不影响它使用onChange方法进行一些其他操作。基于这种方式，我们可以实现一个简单的双向绑定，具体请看双向绑定

```javascript
function proxyHoc(WrappedComponent) {
  return class extends Component {
    constructor(props) {
      super(props);
      this.state = { value: '' };
    }

    onChange = (event) => {
      const { onChange } = this.props;
      this.setState({
        value: event.target.value,
      }, () => {
        if(typeof onChange ==='function'){
          onChange(event);
        }
      })
    }

    render() {
      const newProps = {
        value: this.state.value,
        onChange: this.onChange,
      }
      return <WrappedComponent {...this.props} {...newProps} />;
    }
  }
}

class HOC extends Component {
  render() {
    return <input {...this.props}></input>
  }
}

export default proxyHoc(HOC);
```
6. 渲染劫持  
高阶组件可以在render函数中做非常多的操作，从而控制原组件的渲染输出。只要改变了原组件的渲染，我们都将它称之为一种渲染劫持。
实际上，上面的组合渲染和条件渲染都是渲染劫持的一种，通过反向继承，不仅可以实现以上两点，还可直接增强由原组件render函数产生的React元素。

```javascript
function hijackHOC(WrappedComponent) {
  return class extends WrappedComponent {
    render() {
      const tree = super.render();
      let newProps = {};
      if (tree && tree.type === 'input') {
        newProps = { value: '渲染被劫持了' };
      }
      const props = Object.assign({}, tree.props, newProps);
      const newTree = React.cloneElement(tree, props, tree.props.children);
      return newTree;
    }
  }
}
```
React.cloneElement()克隆并返回一个新的React元素，使用 element 作为起点。生成的元素将会拥有原始元素props与新props的浅合并。新的子级会替换现有的子级。来自原始元素的 key 和 ref 将会保留。

#### 实际应用
1. 日志打点
2. 双向绑定
3. 表单验证
4. Redux的connect

```javascript
export const connect = (mapStateToProps, mapDispatchToProps) => (WrappedComponent) => {
  class Connect extends Component {
    static contextTypes = {
      store: PropTypes.object
    }

    constructor () {
      super()
      this.state = {
        allProps: {}
      }
    }

    componentWillMount () {
      const { store } = this.context
      this._updateProps()
      store.subscribe(() => this._updateProps())
    }

    _updateProps () {
      const { store } = this.context
      let stateProps = mapStateToProps ? mapStateToProps(store.getState(), this.props): {} 
      let dispatchProps = mapDispatchToProps? mapDispatchToProps(store.dispatch, this.props) : {} 
      this.setState({
        allProps: {
          ...stateProps,
          ...dispatchProps,
          ...this.props
        }
      })
    }

    render () {
      return <WrappedComponent {...this.state.allProps} />
    }
  }
  return Connect
}
```
#### 注意事项
1. 静态属性拷贝  
当我们应用HOC去增强另一个组件时，我们实际使用的组件已经不是原组件了，所以我们拿不到原组件的任何静态属性，我们可以在HOC的结尾手动拷贝他们：

```javascript
function proxyHOC(WrappedComponent) {
  class HOCComponent extends Component {
    render() {
      return <WrappedComponent {...this.props} />;
    }
  }
  HOCComponent.staticMethod = WrappedComponent.staticMethod;
  // ...
  return HOCComponent;
}
```
如果原组件有非常多的静态属性，这个过程是非常痛苦的，而且你需要去了解需要增强的所有组件的静态属性是什么，我们可以使用hoist-non-react-statics来帮助我们解决这个问题，它可以自动帮我们拷贝所有非React的静态方法，使用方式如下

```javascript
import hoistNonReactStatic from 'hoist-non-react-statics';
function proxyHOC(WrappedComponent) {
  class HOCComponent extends Component {
    render() {
      return <WrappedComponent {...this.props} />;
    }
  }
  hoistNonReactStatic(HOCComponent,WrappedComponent);
  return HOCComponent;
}
```
2. 不要在render方法内使用高阶组件  
React Diff算法的原则是：  
    - 使用组件标识确定是卸载还是更新组件  
    - 如果组件的和前一次渲染时标识是相同的，递归更新子组件
    - 如果标识不同卸载组件重新挂载新组件

每次调用高阶组件生成的都是是一个全新的组件，组件的唯一标识响应的也会改变，如果在render方法调用了高阶组件，这会导致组件每次都会被卸载后重新挂载。
3. 不要改变原始组件
4. 透传不相关的props  
使用高阶组件，我们可以代理所有的props，但往往特定的HOC只会用到其中的一个或几个props。我们需要把其他不相关的props透传给原组件，如下面的代码

```
function visible(WrappedComponent) {
  return class extends Component {
    render() {
      const { visible, ...props } = this.props;
      if (visible === false) return null;
      return <WrappedComponent {...props} />;
    }
  }
}
```
