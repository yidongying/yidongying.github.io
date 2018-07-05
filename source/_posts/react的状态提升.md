---
title: react的状态提升.md
date: 2017-08-23 11:02:39
tags:
categroies: react的概念理解
---
**1.概念理解**
在react中是单向数据流的设计, 即 只有父组件可以传递数据给子组件,而没有子组件传递数据给父组件的概念. 以正确的技术说明，是 拥有者组件 可以设置 被拥有者组件 中的资料，也就是主人与仆人的关系。

那么子组件要传递数据给父组件该如何沟通呢?

换句话说就是, react 如何将子组件的值暴露让父组件获取到?

可以采用一种迂回的方法, 在父组件中设置一个方法(函数), 将其通过props传递给子组件, 然后在子组件中更新state的状态,并调用父组件中传过来的方法, 将state数据作为参数传递给父组件. 这样, 改变父组件的状态，从而改变受父组件控制的所有子组件的状态. 这就是状态提升的概念.   用官方的原话就是:    '共享 state(状态) 是通过将其移动到需要它的组件的最接近的共同祖先组件来实现的。 这被称为“状态提升(Lifting State Up)'。

**官方参考网址: http://www.css88.com/react/docs/lifting-state-up.html**

**2.举例说明**
<br>
下面举个例子说明:

App.js文件   

```
import  React, { Component } from 'react'

import Item from './Item'

export default class App extends Component {

    constructor(props) {

        super(props)

        this.state = {

            options: [

                {name:'（1）免费行李', value: 1 },

                {name:'2', value: 2 },

                {name:'3', value: 3 } ],

            price: 0

        }

   }

    changePrice = (value) => {

        let price = 800

        if(value === 1)  price = 0

        else

        price *= value - 1

        this.setState({price: price}) }

    render() {

    return (

        {this.state.price}

)  }}
```

Item.js文件

```
import React, { Component } from 'react'

export default class Item extends Component {

    constructor(props) {

        super(props)

        this.handleChange = this.handleChange.bind(this)

    }

    handleChange(e){

    this.props.changePrice(e.target.value)

    }

    render() {

        var options = []

        var optionArray = this.props.optionArray

        options = optionArray.map(function(item, index){

            return ( {item.name} )

        })

        return ( {options} )

    }

}
```

说明:

 **2.1. 先绑定(bind)住render有用到的方法**

在父组件与子组件各有用到一个自己的方法changePrice，并在render中作赋值，在React中需要bind过才会把this对住，因为在render的return语句中使用，它在重渲染(re-render)时会再次建立新的方法(函数)内容值，这样会有效能上的影响，所以要先作绑定的事，然后再render的return里面用。关于bind(this)的一些理解,在我的另一篇文章,可以参考:https://www.jianshu.com/p/f7f2636d16a9

先绑定要在类的contructor里作，像下面这样，我这写一个父组件而已，子组件一样:

constructor(props) {super(props)this.state = {price:0}//先bind类中方法this.changePrice =this.changePrice.bind(this)  }

之后在render的return要改成这样:

```
render() {
  return(
	<div>{this.state.price}</div>
  );  
}
```



**2.2 校正state(状态)里的资料，以及提升到父组件去**

在子组件中的state(状态)中的资料是不是有那么必要放在子组件中，如果你还有第二个子组件、第三、第四…，它们都要用例如这里的选中资料，你放在这个子组件是违反了上面说的应用领域全局资料思维的。

先看一下子组件目前的state，是长这个样子:

	this.state = {names: ['（1）免费行李','2','3'],values: ['1','2','3'],selectName:'',prices:'0'}

这里要先校正一下，names与values是代表选项中的名与值，它们是有关联的，所以应该是这样的下面结构才是好些的，value如果是要用来代表数字，就用数字就行不需要用字串:

```
options: [  {name:'（1）免费行李',value:1},  {name:'2',value:2},  {name:'3',value:3}]
```

选中了哪个选项这个状态资料，还是要先放在子组件中，因为子组件中有选项盒，与触发的更动方法，但选项的资料可以移到上层的父组件中:

这是上层App.js中的状态:

```
this.state = {
	options: [
		{name:'（1）免费行李',value:1},
		{name:'2',value:2},  
		{name:'3',value:3}],
	price:0
 }
```

父组件也改用把state里面的选项值，用props值给子组件，所以在render里语句改成下面这样:

```
render() {
  return(
    <div>{this.state.price}</div>
  );  
}
```

子组件中这时可以用this.props.optionArray接到传入的选项值，所以在render方法中，改用这个来代替之前的this.state.names与this.state.values，简单改写如下:

varoptions = []varprices =this.state.pricesvaroptionArray =this.props.optionArrayfor(vari =0; i< optionArray.length; i++) {      options.push({optionArray[i].name})  }

注: 这里不用for...in语句而用for语句，是因为for...in语句是个不建议用在数组资料的语法，它并不会保证取到数组成员里的顺序。for...in只会用在对象的寻遍中。

更精简的写法是用Array.map，如下:

```
var options = []
var prices =this.state.prices
var optionArray =this.props.optionArray
options = optionArray.map(function(item, index){return({item.name})})
```

接着，如果依选项选中然后计算价格这件事，规划中应该是整个应用来作的，例如有可能还有其他的组件中也有其他的选项，最后统一要来算价格，所以计算价格这件事，也应该放到父组件去，所以如同上面的改写一样，把子组件的prices状态与相关计算的代码，都提到父组件，这个子组件纯用来当选项盒用而已。子组件此时连state都可以不用有。

因为整个改写过的代码会多些，所以我把父组件与子组件中的代码整个贴上。

父组件App.js:

```
import React, { Component } from 'react';
import Item from'./Item'
export default class App extends Component{
constructor(props) {
	super(props)
	this.state = {options: [ {name:'（1）免费行李',value:1},        {name:'2',value:2},        {name:'3',value:3}   ],price:0}
	this.changePrice =this.changePrice.bind(this)  
}  
changePrice(value){
	var price =800;
	if(value ===1) 
	price =0 
	else
	price = (value -1) * price
	this.setState({price: price})  }  
render() {
	return(
	<div>{this.state.price}</div>
	)  
}}
```

子组件Item.js:

```
import React, { Component }from'react'
export default class Item extends Component{
	constructor(props) {
		super(props)
	}  
	handleChange(e){
		this.props.changePrice(e.target.value)  
	}  
	render() {
	var options = []
	var optionArray =this.props.optionArray    
	options = optionArray.map(function(item, index){return({item.name})    })return(

<div>{options}</div>

)  }}
```

**2.3. 目前最终进化版本**

这个版本有几个改进如下，供参考:

用let/const取代var。

不用分号(;)作为语句结尾。

Item子组件改用函数定义方式，取代原先的组件定义方式。

能合并的语句都合并。

函数全用箭头函数(注意需额外加装babel-plugin-transform-class-properties)。

App.js

```
import React, { Component }from'react'
import Item from'./Item2'
export default class App extends Component{
	constructor(props) {
		super(props)
		this.state = {options: [ 
		{name:'（1）免费行李',value:1},
		{name:'2',value:2}, 
		{name:'3',value:3}],
  changePrice =(value) =>{
	  let price =800
	  if(value ===1) 
		  price =0
	  else
	  price *= value -1
render() {
	return(

	<div>{this.state.price}</div>
	
	)  
}}
```

Item.js

```
import React from'react'
const Item =(props) =>{
const optionArray = props.optionArray
const options = optionArray.map((item, index) =>{return({item.name})  })return(

{props.changePrice(e.target.value)}}>  {options}

)}
export default Item
```

<br>
**3.参考文档**
<br>
参考文档: 1. https://www.cnblogs.com/zhangbob/p/6962138.html?utm_source=itdadao&utm_medium=referral

2. http://www.css88.com/react/docs/lifting-state-up.html官方文档

3.https://blog.csdn.net/YQXLLWY/article/details/73481063


