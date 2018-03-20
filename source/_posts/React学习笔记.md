title: React 基本总结
date: 2017/6/13 20:46:25
---


## 基本操作

1. 进行表达式操作

   ```javascript
   const user = {
     firstName: 'tian',
     lastName: 'wucai'
   }

   const element = {
     <h1>
     	hello, {user.firstName}
     </h1>
   }
   ```
<!--more-->

2. 可以对标签传值

   ```javascript
   const element = <img src={user.avatarUrl}/>
   ```



3. 对字符串进行安全的输出

   ```Javascript
   const title = '<h1>12</h1>'
   const element = <h1>{title}</h1>
   ```



4. 创建html标签对象

   ```javascript
   const element = React.createElement(
   	'h1',
     	{className: 'top'},
     	'hello world'
   )
   ```

   - 第一个参数为标签名
   - 第二个参数为给予此标签的属性
   - 第三个是标签内包裹的内容

   也可以提前创建好标签的具体属性，随后再传给`React.createElement()`,如

   ```javascript
   const element_info = {
     type:'h1',
     props: {
       className: 'top',
       children: 'Hello,word'
     }
   }
   ```



## 显示Elements

1. 显示到指定dom

   ```html
   <div id='root'>Hello empty</div>
   ```

   如果没有设置react，则默认显示**Hello empty**。

   ```javascript
   const element = <h1>hello world</h1>
   ReactDOM.render(
   	element,
     	document.getElementById('root')
   )
   ```

   当使用了react，指定了显示到id为root的标签中，则页面返回的是**hello world**

   > 注意： react 的元素是不可变类型，当你创建完元素后不能改变其属性或者包裹的内容， 以下是官方的一个简单显示时间的demo

```javascript
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(
    element,
    document.getElementById('root')
  );
}
setInterval(tick, 1000);
```

> 注意：react 只会更新需要更新的节点，对其他节点是没有操作的

2. 细节

   ```html
   <div />

   <div></div>
   1
   <div>{false}</div>

   <div>{null}</div>

   <div>{undefined}</div>

   <div>{true}</div>
   ```

   对于如上的返回，只能显示1，其他会自动忽略

   > `false`, `null`, `undefined`, and `true` are valid children. They simply don't render

## 组件和参数的使用

1. 使用JavaScript function 方式定义简单组件

   ```javascript
   function Welcome(props){
      return <h1>Hello, {props.name}</h1>;
   }
   ```

2. 继承React.Component方式

   ```javascript
   class HelloWord extends React.Component {
     render(){
       return <h1>hello,{this.props.name}</h1>
     }
   }
   ```

> 注意：组件名必须首字母大写
>
> ​	    如果在组件内使用了this.props获取参数，则必须向组件传入对应名字的属性参数

3. 其他的使用方式

   ```javascript
   import React from 'react';

   const MyComponents = {
     DatePicker: function DatePicker(props) {
       return <div>Imagine a {props.color} datepicker here.</div>;
     }
   }

   function BlueDatePicker() {
     return <MyComponents.DatePicker color="blue" />;
   }
   ```



4. 参数设置

   当我们传递参数到组件时，可以对参数进行一些规定，如下:

   ```
   import PropTypes from 'prop-types';

   class Greeting extends React.Component {
     render() {
       return (
         <h1>Hello, {this.props.name}</h1>
       );
     }
   }

   Greeting.propTypes = {
     name: PropTypes.string
   };
   Greeting.defaultProps = {
     name: 'Stranger'
   };
   ```

   上面说明，我们要求传入的name为String类型，如果没有传，则默认为**Stranger**,以下是官方对于属性检验的一个例子:

   ```Javascript
   import PropTypes from 'prop-types';

   MyComponent.propTypes = {
     // You can declare that a prop is a specific JS primitive. By default, these
     // are all optional.
     optionalArray: PropTypes.array,
     optionalBool: PropTypes.bool,
     optionalFunc: PropTypes.func,
     optionalNumber: PropTypes.number,
     optionalObject: PropTypes.object,
     optionalString: PropTypes.string,
     optionalSymbol: PropTypes.symbol,

     // Anything that can be rendered: numbers, strings, elements or an array
     // (or fragment) containing these types.
     optionalNode: PropTypes.node,

     // A React element.
     optionalElement: PropTypes.element,

     // You can also declare that a prop is an instance of a class. This uses
     // JS's instanceof operator.
     optionalMessage: PropTypes.instanceOf(Message),

     // You can ensure that your prop is limited to specific values by treating
     // it as an enum.
     optionalEnum: PropTypes.oneOf(['News', 'Photos']),

     // An object that could be one of many types
     optionalUnion: PropTypes.oneOfType([
       PropTypes.string,
       PropTypes.number,
       PropTypes.instanceOf(Message)
     ]),

     // An array of a certain type
     optionalArrayOf: PropTypes.arrayOf(PropTypes.number),

     // An object with property values of a certain type
     optionalObjectOf: PropTypes.objectOf(PropTypes.number),

     // An object taking on a particular shape
     optionalObjectWithShape: PropTypes.shape({
       color: PropTypes.string,
       fontSize: PropTypes.number
     }),

     // You can chain any of the above with `isRequired` to make sure a warning
     // is shown if the prop isn't provided.
     requiredFunc: PropTypes.func.isRequired,

     // A value of any data type
     requiredAny: PropTypes.any.isRequired,

     // You can also specify a custom validator. It should return an Error
     // object if the validation fails. Don't `console.warn` or throw, as this
     // won't work inside `oneOfType`.
     customProp: function(props, propName, componentName) {
       if (!/matchme/.test(props[propName])) {
         return new Error(
           'Invalid prop `' + propName + '` supplied to' +
           ' `' + componentName + '`. Validation failed.'
         );
       }
     },

     // You can also supply a custom validator to `arrayOf` and `objectOf`.
     // It should return an Error object if the validation fails. The validator
     // will be called for each key in the array or object. The first two
     // arguments of the validator are the array or object itself, and the
     // current item's key.
     customArrayProp: PropTypes.arrayOf(function(propValue, key, componentName, location, propFullName) {
       if (!/matchme/.test(propValue[key])) {
         return new Error(
           'Invalid prop `' + propFullName + '` supplied to' +
           ' `' + componentName + '`. Validation failed.'
         );
       }
     })
   };
   ```

   ​

   ​



## state的使用

首先，下面是一个官方给出的demo

```javascript
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

下面对以这个例子进行一些说明:

- 在创建组件是，使用了构造函数constructor来对一些属性进行初始化，包括创建了state里面包含的值`{date: new Date()}`
- 返回的html中，获取的是state里面的值`{this.state.date.toLocaleTimeString()}`
- 使用了React组件中的生命周期函数，在组件加载后(`componentDidMount`)启动定时任务，每秒调用`tick()`函数，设置state里面的date属性
- 当组件卸载(`componentWillUnmount`)后，清空定时任务

> 注意：应该以正确的方式修改state值，如下:
>
> this.state.date = new Date() 错误
>
> this.setState({date:new Date()}) 正确



state还有另外一种更新方式，是可以防止数据不正确的问题，如下：

```javascript
this.setState((prevState, props) => ({
  counter: prevState.counter + props.increment
}));
or
this.setState(function(prevState, props) {
  return {
    counter: prevState.counter + props.increment
  };
});
```



## 事件处理

1. 添加事件

   普通html书写方式如下：

   ```html
   <button onclick="activateLasers()">
     Hello world
   </button>
   ```

   React 书写方式如下:

   ```html
   <button onClick={activateLasers}>
     Activate Lasers
   </button>
   ```



2. 防止标签的默认事件

   html代码如下:

   ```Html
   <a href="#" onclick="console.log('The link was clicked.'); return false">
     Click me
   </a>
   ```

   React代码如下:

   ```javascript
   function ActionLink() {
     function handleClick(e) {
       e.preventDefault();
       console.log('The link was clicked.');
     }

     return (
       <a href="#" onClick={handleClick}>
         Click me
       </a>
     );
   }
   ```

   > 在react中，必须通过e.preventDefault来使默认事件失效



## 条件判断

Example1:

```Html
function Greeting(props) {
  const isLoggedIn = props.isLoggedIn;
  if (isLoggedIn) {
    return <UserGreeting />;
  }
  return <GuestGreeting />;
}
ReactDOM.render(
  // Try changing to isLoggedIn={true}:
  <Greeting isLoggedIn={false} />,
  document.getElementById('root')
);
```

上面的这个代码块，是可以根据传入的`{false}`来判断返回内容



Example2:

```javascript
function Mailbox(props) {
  const unreadMessages = props.unreadMessages;
  return (
    <div>
      <h1>Hello!</h1>
      {unreadMessages.length > 0 &&
        <h2>
          You have {unreadMessages.length} unread messages.
        </h2>
      }
    </div>
  );
}

const messages = ['React', 'Re: React', 'Re:Re: React'];
ReactDOM.render(
  <Mailbox unreadMessages={messages} />,
  document.getElementById('root')
);
```

可以在`{}`里面包裹所有JSX内置的表达式，还有JavaScript的操作符。

> 注意：{ condition && expression}  当左边condition为true时，则会把右边expression的值输出。若condition为false，则会忽略



Example3:

```javascript
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      {isLoggedIn ? (
        <LogoutButton onClick={this.handleLogoutClick} />
      ) : (
        <LoginButton onClick={this.handleLoginClick} />
      )}
    </div>
  );
}
```

> 使用了if-else结构的三目操作符



Example4:

```Html
function WarningBanner(props) {
  if (!props.warn) {
    return null;
  }

  return (
    <div className="warning">
      Warning!
    </div>
  );
}

class Page extends React.Component {
  constructor(props) {
    super(props);
    this.state = {showWarning: true}
    this.handleToggleClick = this.handleToggleClick.bind(this);
  }

  handleToggleClick() {
    this.setState(prevState => ({
      showWarning: !prevState.showWarning
    }));
  }

  render() {
    return (
      <div>
        <WarningBanner warn={this.state.showWarning} />
        <button onClick={this.handleToggleClick}>
          {this.state.showWarning ? 'Hide' : 'Show'}
        </button>
      </div>
    );
  }
}

ReactDOM.render(
  <Page />,
  document.getElementById('root')
);
```

上面的官方demo的功能是显示或隐藏组件，通过事件来管理**warn** 的值来选择性的返回内容，实现了组件显示或隐藏的功能。



## 列表的使用

Example1:

```javascript
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((number) => number * 2);
console.log(doubled);
>>[2, 4, 6, 8, 10]
```



Example2:

```Javascript
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li>{number}</li>
);

ReactDOM.render(
  <ul>{listItems}</ul>,
  document.getElementById('root')
);
```



Example3:

```Javascript
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li>{number}</li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

> 当使用上面的代码时，会出现warn警告，表示没有设置key，正确的设置应该如下:

```Javascript
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li key={number.toString()}>
      {number}
    </li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```



## 表单处理

Example1 :

```Javascript
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: ''};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

Example2:

```Javascript
class EssayForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: 'Please write an essay about your favorite DOM element.'
    };

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('An essay was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <textarea value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```



Example3:

```Javascript
class FlavorForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: 'coconut'};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('Your favorite flavor is: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Pick your favorite La Croix flavor:
          <select value={this.state.value} onChange={this.handleChange}>
            <option value="grapefruit">Grapefruit</option>
            <option value="lime">Lime</option>
            <option value="coconut">Coconut</option>
            <option value="mango">Mango</option>
          </select>
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```



```Javascript
class Reservation extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      isGoing: true,
      numberOfGuests: 2
    };

    this.handleInputChange = this.handleInputChange.bind(this);
  }

  handleInputChange(event) {
    const target = event.target;
    const value = target.type === 'checkbox' ? target.checked : target.value;
    const name = target.name;

    this.setState({
      [name]: value
    });
  }

  render() {
    return (
      <form>
        <label>
          Is going:
          <input
            name="isGoing"
            type="checkbox"
            checked={this.state.isGoing}
            onChange={this.handleInputChange} />
        </label>
        <br />
        <label>
          Number of guests:
          <input
            name="numberOfGuests"
            type="number"
            value={this.state.numberOfGuests}
            onChange={this.handleInputChange} />
        </label>
      </form>
    );
  }
}
```

> [name]:value 是使用了es6的一些特性,如果是使用ES5的话可以如下

```Javascript
var partialState = {};
partialState[name] = value;
this.setState(partialState);
```

