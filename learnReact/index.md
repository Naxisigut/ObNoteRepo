## 创建react项目
```js
npx create-react-app project-name
npm install
npm start
```

## 组件
#### 组件的创建
组件分为类式组件和函数式组件。
- 标准类式组件
```js
import React from 'react';

// 标准类式组件
// 1. render方法一定要写
// 2. render中的this指向组件实例，这是由类的语法决定的
// 3. 三大核心属性：refs state props
class MyComponent extends React.Component{
  /* constructor只在初始化时执行一次 */
  constructor(props){
    super(props)
    // 初始化state
    this.state = {
      count: 0
    }

    // 初始化方法
    // this.onClick指向MyCompontent原型对象上的方法onClick，
    // 通过bind生成一个新的方法，赋值给实例对象上新增的属性onClick
    this.onClick = this.onClick.bind(this) 
  }

  /* render在初始化和setState变化时执行 */
  render(){
    const { count } = this.state
    return (
      <>
        <div>count: { count }</div>
        <div>
          <button onClick={ this.onClick }>++</button>
        </div>
      </>
    )
  }
  onClick(){
    let { count } = this.state
    // 更新state必须通过setState
    this.setState({
      count: count+1
    })
  }
}

export default MyComponent
```
- 类式组件的简写
```js
import React from 'react';
/* 类式组件的简写 */
class MyComponent extends React.Component{
  // 初始化state：直接在类中写赋值语句
  state = {
    count: 0
  }
  render(){
    const { count } = this.state
    return (
      <>
        <div>count: { count }</div>
        <div>
          <button onClick={ this.onClick }>++</button>
        </div>
      </>
    )
  }
  // 初始化方法：直接在类中写赋值语句，并用箭头函数将this指向实例
  onClick = () => {
    let { count } = this.state
    this.setState({
      count: count+1
    })
  }
}

export default MyComponent
```
#### 组件的使用：props
- props的传递
```js
// 在标签上通过attr的形式传入props
<MyComponent name="lcc" age={18} sex="男" />

// 在标签上通过解构对象的形式批量传入props
const p = {
  name: 'lcc',
  age: 18,
  sex: '男'
}
<MyComponent {...p} />
```
- props的使用
```js
import React from 'react';
import PropTypes from 'prop-types';

class MyComponent extends React.Component{
  static propTypes = { // 注意是小写的propTypes
    name: PropTypes.string.isRequired, // 限制类型为字符串，必传
    sex: PropTypes.string,
    speak: PropTypes.func // 限制类型为函数
  }
  static defaultProps = {
    age: 18 // 限制默认prop值
  }
  render(){
    const { name, age, sex } = this.props
    ....
  }
}

```
#### 组件的使用：refs
refs