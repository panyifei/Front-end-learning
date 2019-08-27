---
layout: default
title: \{#\{ site.name }}
---
# Redux文档阅读
Redux是js的一个可预测的状态的管理者。他是来回来数据什么时候改变，数据来自哪里的。更多的约束得到了我们需要的东西。

# 介绍
## 要旨
 - 你的App的所有的状态存在一个单一store的对象树中。
 - 唯一的改变状态树的方法是发出一个action(一个对象描述发生了什么)
 - 通过写纯函数reducer来指定action是如何更改状态树的。

就这么多..

Redux没有一个dispather，并且不支持many stores。

他只拥有一个store和一个单一的reducer方法。

当项目不断演进的过程中，可以分割reducer成为更小的reducer来独立地操作状态树的不同部分。这点很像React app只对外一个根节点，但是它由很多小的部分组成的。

## 动机
随着单页应用的需求变得越来越复杂，我们的代码比起以前而言需要管理更多的状态。这下状态包括服务器的返回，缓存的数据，还有本地的一些数据。UI的状态也变得越来越复杂。

管理这个不断改变的状态是困难的，当我们失去了state的控制之后，我们基本也不懂程序发生了什么了。当一个程序是不透明的和不确定的，很难重现bug或者添加新特性。

问题变得难处理是因为我们把两个人脑很难区分的概念混在了一起，就是异变和异步。

Redux尝试着来让状态突变变得可预测。

## 3个原则
React可以用3个基本原则描述

### 单一来源的真理
你的整个app的状态存在一个单一store的对象树中。

这使得能够创建通用的app，你服务器的状态可以通过序列化传递给客户端，并且没有额外的代码。一个简单的状态树使得调试和反省整个项目变得容易，

### 状态是只读的
唯一的改变状态树的方法是发出一个action(一个对象描述发生了什么)

这点保证了无论是view还是网络的回调都不可能直接修改state。相替代的，他们表示了一个去转换state的意图，因为所有的改变都是按照一个严格顺序一步步执行的，他就没有一个竞态的顺序需要去注意。行动是简单的对象，他们是很容易被统计，存储，以及序列化的。

### 改变是通过纯函数执行的。
通过写纯函数reducer来指定action是如何更改状态树的。

recuder是纯函数，输入原来的状态和action，得到了新的状态。记住我们是返回新的状态对象，而不是在原来的state上进行改变。我们可以从一个单一的reducer开始，然后随着app grow，将它细分成更小的reducer。因为reducer仅仅只是函数，我们可以控制他们执行的顺序，传入外地的数据，甚至在分页这种普遍的任务中使用可重用的reducer。

## 现有技术
Redux是一个混合的遗产。他和一些模式和技术很像，但是也在一些重要的部分与他们是不同的。

### Flux
Redux可以被以为是一个Flux的实现吗？是也不是。Flux的创造者承认了这点。

Redux的灵感来源于Flux的几个重要特性。像Flux一样，Redux让你把model的更新逻辑放到了你的app的一个层去解决。(在Flux中为stores，在Redux中为reducers)。都让你通过一个叫做action的空对象来描述每次突变。

与Flux不同的是，Redux没有一个dispatcher的概念。这是因为他依赖于纯函数，而不是事件发射器。纯函数很容易管理，不需要一个额外的实体来管理他。你可以把它当做一个偏差或者一个实现细节。从(state,action) => state这个方面来看，Redux确实是Flux的架构，但是由于是纯函数，这个就更简单了。

另一个重要的区别是Redux假设你从不突变你的数据。就是说在reducer里面修改这个数据是很不被推荐的，我们应该返回新的数据，类似于Immutable。

我们从技术上是可以写非纯函数来变异数据的，但是我们劝你不要这么做。

### Elm
这是一门语言，被Haskell影响的。他的更新机制updater很像Redux的reducer。他最后会被编译为js。因为他是语言，所以我们可以强制他纯函数。

### Immutable
Immutable是一个js库，实现持久化数据结构。他有比较惯用的js api。

Redux不关心你如何存储一个对象，可以是个简单的对象，可以是个Immutable对象。

### Baobab
另一个为更新简单的JS对象实现了immutable api的库。与Redux一起用的好处不大。因为他的更新是给数据提供了游标修改的，而Redux希望的是只在dispatch一个action来修改数据。

### Rx
只能说他能配合RX做一些事情，毕竟RX是流式管理事件流顺序的，可以用RX管理多个异步事件流，然后再把他们交给store.dispatch。

## 生态系统
Redux是个小的库，但是他的概念和API都是很精心挑选来扩展成工具和扩展的生态系统。

# 基础
## actions
actions是从应用向store发送的信息。他们是store的唯一的信息来源，我们使用store.dispatch来发送信息。

action是简单的js对象，action必须有个type的属性来指示需要被执行的action。type必须被定义为string的类型，一旦app大了之后，我们可能需要把他们移到独立的模块。

除了type之外，action的结构随我们定义，这里也有个小规范啦。

这里引入了一个action creator的概念，不过没啥意思，小语法糖。

## reducers
action描述了到底发生了什么，但是没有指明应用的state作为结果是如何改变的，这是reducer的任务。

在redux中，所有的应用state都是存为了一个单一对象，在写代码之前去思考下他的形状很重要，如何把你的应用的状态最小展示为一个对象。

reducer是个纯函数，只拿取老的状态，action，然后返回一个新的状态。我们在reducer里面并不能干的事情：

 - 突变参数
 - 做一些边界影响，例如调用API，或者路由转换
 - 执行一些不纯洁的方法，例如random或者Date.now这种

在下面会有介绍边界影响的地方，目前记好了reducer必须是纯的。如果是flux的话，会有个dispatcher，会有个事件发射器。redux的话用出纯函数的reducer来取代了事件发射器。

## store
store是个将action和reducer结合到一起的地方，他有以下的任务：

 - 保存应用的state
 - 允许通过getState来拿到数据
 - 允许数据可以通过dispatch来更新
 - 通过subscribe注册监听
 - 通过subscribe返回的函数来取消注册监听

注意很重要的是你只有一个单一store，当你想要分割数据处理逻辑，你可以使用reducer composition而不是很多个store。

创建store很容易，在之前的模块，使用combineReducer来combine多个reducer成为一个，现在将他传给createStore就好了。

我们可以将createStore的第二个参数来指定初始状态。

## 数据流
redux的结构是严格的单向数据流。意味着应用中的数据都是遵循着同样的生命周期模式，让你的app更加可预测并且可理解。也鼓励了数据一致化。

## 与react的使用方法
他能很好的和react合作的原因是，react把state作为了UI的一种描述形式。

# 进阶
## 异步action
在之前的介绍中，我们创建的完全是同步的应用，每次一个action发出的时候，state都会立刻更新。现在开始介绍如果异步的请求来适应redux流。

主要的思想就是action除了返回对象之外还可以返回函数，这个时候action创建函数就变成了thunk了。

然后这时接入redux-thunk，这个中间件就会在action返回函数的时候执行。这个函数可以是不纯净的，可以去执行异步函数，也可以dispatch action。当然了，thunk并不是唯一方式，redux-saga使用感觉很不错啊。

## 异步数据流
默认情况下，createStore的Redux store没有用到middware，只支持同步数据流。

其实中间件的中间环节随便我们返回什么，但是到了最后只要返回一个简单的action对象就好了，saga估计也就是在这里面做了很多文章。

## 中间件
用过express或者koa的话，就会有一个已经熟悉的中间件的概念了。与之类似的，redux提供了一个第三方的点，就是从发出一个action到他到达reducer的时候。本文就是带我们了解moddleware。

就是从打log这一件事上，慢慢地去寻找到底补丁该打在哪里咯。

## 与React Router一起使用
react-router原来保留#的原因是为了适配IE9，params是react-router提供的功能啊。

# 技巧
## 转换到redux
redux并不是一个单一框架，是一系列的约定以及一些让他们一起运行的函数。

## 使用对象展开运行符
我们平常时候写Object.assign来复制一个新的对象，但是代码很冗余啊，于是我们用对象展开运算符，可以将对象的可枚举属性拷贝至另一个对象。

## 减少样板代码
## 服务器端渲染
## 计算衍生字段
Reselect库可以创建可记忆的，可组合的selector函数，可以用来高效的计算衍生数据。

这个挺有意思的，之前react的渲染可以使用shuoldComponentUpdate来判断是否重新渲染，如果不需要的话直接不渲染就好了。虽然可以不过粒度太大了。因为connet到store的时候会从真实的数据计算成衍生数据，也就是selector了，这个东西每次props改变的时候都会计算，如果计算量很大的话，很尴尬的。

于是用了这个库，他会比较传入的参数是否绝对相等，不过变化了就重新计算，如果没变化就不计算了，他的很大的一个好处是他可以将粒度拆的很细。

## 实现撤销重做
实现撤销操作在传统听难做的，你可能得克隆所有的model状态来追踪历史状态，

撤销重做的数据结构其实不难诶：

 - past:[1,2,3]
 - present:4
 - future:[5,6]

重做就是从future的头部推入present，present推入past，撤销就是past的尾部推入present，present推入future，当位于撤销的时候，新建了事件，就抛弃future。

实现的话用高等函数做？！好吧，其实就是接受了一个reducer，然后这个reducer被包装了下，返回了一个reducer，不过这个reducer认识UMDO和REDO。跟combineReducer其实做的是一样的事情，已经有redux-undo来做这件事情了。

## 子应用隔离
子应用的意思就是应用之间完全独立，相互之间不会共享数据或者action，不可见，不互相通信。

于是就完全当做几个app做，就是createStore，Provider封装在组件内，多次调用组件创建完全隔离的store。

## 构建reducer
这是redux的核心，redux是个很简单的设计模式。所有写的逻辑都进入了一个简单对象，运行的唯一方法就是传一个简单对象描述如何发生。redux store调用这段逻辑传入当前状态树和描述对象，这段写的逻辑返回新的状态树。Redux通知监听者，这颗状态树已经被更新了。

redux要求了reducer的格式，而且要求是纯函数，并且是可预测的。他只要你遵守他的限制，并不在乎你如何构建你的逻辑。既是自由之源，也是混乱之源。但是，还是有很多通用的模式。

## 必要的Reducer思想
注意必须生成一个新对象，不仅仅是最外层的对象，对于任何子对象都得做到。

突变不被推荐的原因是他通常打破了时差调试，还有redux的connect方法：意思是React的devtools会希望重新执行来得到一个值而不会产生影响。connet函数检查脏也就是检查引用是否改变了，如果没有改变的话那就不会重新渲染了。

其实是有数据库设计的问题的感觉。

## 基本的reducer结构
注意，整个app只有一个reducer，他是传入createStore的第一个参数。这个reducer需要做的事情有，第一次state为undefined的时候，reducer需要提供一个默认的state。在拿到action之后，对老的state进行处理得到新的数据后，返回一个新的对象。如果没有变化，返回老的数据。

这里就是说data大部分也就是3种分类，包含了数据的Domain Data，应用的状态App Data(就是比如现在正在获取数据？那个目前被选中着)，还有就是界面状态UI Data(比如modal现在打开着)。

## 拆分reducer逻辑
对于有意义的应用来说，将你的所有的更新逻辑放到一个reducer方法会很快变得不可维护。函数应该尽可能短，在理想状态下应该只做一件事。因此，将代码拆成一块块的是个好的编程习惯。

因为reducer也就是一个function，我们可以将reducer逻辑拆成多个方法，然后从敷方法调用新方法。

其实就只有combineReducers这个语法糖而已。

## 重构reducer的例子
这里的拆的整体的过程好有意思，反正最后要返回的是一个新对象，就对每个属性做单独的reducer，例如：

```javascript
function rootReducer(state = initialState, action){
  return {
    tudos: todosReducer(state.todos, action),
    visibilityFilter: visibilityFilterReducer(state.visibilityFilter, action),
  };
}
```

然后用一个高阶函数把switch去掉了，就会直接根据property匹配相应的函数了，也很神奇啊。高阶函数可以做好多事情，他可以对于一个函数进行自定义化。

然后上面的那坨代码可以使用内置的combineReducers来复写：

```javascript
const appReducer = combineReducers({
    visibilityFilter : visibilityReducer,
    todos : todosReducer
});
```

太开心了，太帅气了！终于搞清楚里面是啥了。

## 使用combineReducers
多个reducer响应同一个action，独立更新自己的部分，然后所有的部分被合成一个新的state对象。

因为这个模式很常见，redux提供了combineReducers来实现这个行为，他是一个更高级的reducer，拿那些部分reducer来生成一个新的reducer。

几个重要的点：

 - 首先，combineReducers是一个简化大多数情况下的功能，不一定需要，很有可能用不到也有可能不适用。
 - Redux本身不对你state的管理做限制，combineReducers强制执行几个规则。
 - Redux是否在发起一个action的时候调用了所有的Reducer？因为只有一个root reducer方法，所以答案是不调用，但是combineReudcer为了得到新的state树，会用当前的action和state再生成一次，给这些部分一个改变的机会，所以combineReducer执行了所有的
 - 我们可以在reducer的所有层级使用它，而不仅仅是根层次，很常用的，我们会有大量的combined reducer。

### 定义状态
两种方法来定义你store的state的初始形状和内容。首先，createStore会用preloadedState作为首参数，这主要是为了用之前在浏览器中初始化的状态，例如浏览器的localStorage。另一种方法是让root reducer在state为undefined的时候返回初始值。但是当用combineReducers的时候有些额外的关注点我们需要去看。

看意思就是我们如果不传入初始值，就会以reducer的名字为state的初始值，我们在用combineReducer的时候最好带上名字。

## beyond combineReducers
redux的combineReducers很有用，我们故意限制来处理一个简单的case：通过委托更新每一部分的state给特定的slice reducer来更新一个树(一个简单的js对象)。他不处理其他的case，例如一颗用Immutable的maps做的树，尝试传入部分state树作为额外的参数；或者slice reducer的排序；他也不关心如何slice reducer如何运行。

通常的问题是，我如何使用combineReducer来处理这些其他的使用。答案很简单，你可能要用些其他东西了。一旦你超越了核心使用case，就得用更自定义的reducer逻辑了，无论是一次性的用例，还是可以被广泛复用的功能。下面是处理这几种典型用例的建议，不过你自己使用的使用free一些。

## beyond combineReducers
redux的combineReducers很有用，我们故意限制来处理一个简单的case：通过委托更新每一部分的state给特定的slice reducer来更新一个树(一个简单的js对象)。他不处理其他的case，例如一颗用Immutable的maps做的树，尝试传入部分state树作为额外的参数；或者slice reducer的排序；他也不关心如何slice reducer如何运行。

通常的问题是，我如何使用combineReducer来处理这些其他的使用。答案很简单，你可能要用些其他东西了。一旦你超越了核心使用case，就得用更自定义的reducer逻辑了，无论是一次性的用例，还是可以被广泛复用的功能。下面是处理这几种典型用例的建议，不过你自己使用的使用free一些。

### 在slice reducer里用Immutable对象
目前combineReducer只能用空白的js对象。用immutable对象作为state的应用没法使用combineRuducer来管理map。因为许多开发者使用immutable，有一些已经发布的程序提供了等效的功能的，例如redux-immutable。这个package提供了自己实现的combineReducer知道如何迭代不可变的map而不是纯java对象。

### 在slice reducer之间分享数据
类似的，如果sliceA碰巧需要sliceB的部分数据，或者sliceB碰巧需要整个state作为参数，combineReducer并不自己处理这个。这可以通过写一个自定义的知道传入需要的数据作为额外的参数的函数来解决。

一种方法是重新处理下combineReducer，在某些action的时候时候多传些数据，然后在action接住的时候拿数据。

另一种是用高阶函数包一下创建action的函数，多穿些参数。

第三种是用一个combineReducer来处理最简单的情况，然后用一个reducer来处理特殊的需要相互之间交流数据的情况。然后一个包裹的方法，可以来同步运行过这两个reducer。

类似于一个叫做reduce-reducers的模块，他接收大量的reducer然后运行reduce，把之前的结果传给后面的reducer。

如果使用reduceReducers，我们要保证第一个处理了默认情况，因为后面的都会默认整个state已存在。

### 更多建议
理解redux reducer仅仅只是方法很重要。combineReducer很有用，但他只是一个工具。函数可以包含逻辑语句，可以包裹函数可以调用其他函数。你可能需要你的某个slice reducer来重置他的状态，并且只响应特殊的action，你可以用一些特殊的api，上面都提到过的。也就是我们可以有很大的选择权，我们也可以写自己的函数。

## Normalizing State Shape
许多应用处理自然嵌套或者相关的数据。比如，博客主有很多博客，每个博客有评论，所有的博客和评论都是由用户编写。

我们的数据结构可能会很庞大，并且有大量冗余。

 - 当一部分数据在多个地方复制的时候，同步的更新变得很难了。
 - 嵌套数据意味着相应的reducer的逻辑也变得嵌套而且复杂，特别是，尝试更新一个深入嵌套的字段会很难看
 - 因为immutable数据会要求他所有的祖先都被复制且被更新，一个新的对象引用会导致所有连接的UI重新渲染，

 由于这个原因，推荐的方法是在redux中处理关系数据或嵌套数据的时候把他处理为数据库，并将其保存为规范化形式。

 ### 规范化设计state
 基本概念是：

  - 每种类型的数据在state中有自己的表
  - 每个数据表应该将各个项目存在对象中，然后ID作为key，item本身作为值
  - 任何对于项目的引用都应该通过存ID来实现
  - 应该有个数组的IDs用来指示排序


然后给了个例子，给的状态结构总体上更加平坦，与原来的嵌套相比，下面几个方面有改进：

 - 因为item只定义在了一个地方，我们不需要在更新的时候改变多个地方了
 - reducer逻辑不用处理多层嵌套，所以看上去会简单
 - 检索和更新变得简单，我们不再需要深挖state，我们很容易就可以追踪了
 - 因为平层了，所以通常UI通常改动会更少了，我们不需要更新更深层的数据了

规范化的结构通常意味着更多的连接，而不是父组件连接所有向下传递。父元素简单的将ID向下传更容易做性能的优化，渲染上。所以结构化state是改善性能的好方法。

### 组织结构化的state
一个传统的应用会有关系数据和非关系数据的混合，并没有原则来确定描述如何处理，一个常用的模式是将关系的tables放在一个通用的父key下面，例如entities。

### Relationships and Tables
因为我们把我们redux的store当成是数据库了，所以数据库设计的很多原则我们可以使用上来，例如多对多的关系，我们可以存储一个中间表来存对应关系（通常叫做关联表或者连接表），为了保持一致性，我们会都是用byId和allIds来管理。数据库操作的性能也会比较好。

### Normalizing Nested Data
因为API返回的数据很多都是嵌套的，我们可能需要格式化之后再存入state中，Normalizr库就是干的这个事情，可以抽空看下Normalizr库是怎么写的~~

## Updating Normalized Data
我们使用Normalizr库来转换嵌套的数据变成格式化的数据。但是并没有解决在应用其他地方使用的格式化数据的更新问题。这边有一些推荐的方法。

一种是将action的内容合进state中。这在reducer做的事情最少，但是在action创建的地方我们需要拼装数据来适合放进来，也不能处理删除。

如果我们有一个嵌套的slice reducer，每个reducer都需要知道如何处理这个action，比如写一条评论，那些被combine的reducer要分开处理自己的部分。

### 其他的方法
基于任务的更新其实就是我们的update的写法，整个state更新。这样子写的话就得在reducer里面知晓整个state的结构。

### redux-orm
redux-orm提供了一个很有用的抽象来管理redux的格式化数据，它允许你声明modal class并且定义它们之间的关系，他可以为了你的数据类型生成空的table，充当一个专用查找数据的选择器工具，并对该数据执行不可变更新。

总的来说，redux-orm提供了一组非常有用的抽象，来定义数据类型的关系，来检索数据以及数据应用的不可变更新。

## Reusing Reducer Logic
随着应用的发展，reducer逻辑的常见模式开始出现，你会发现reducer逻辑的几个部分对不同类型的数据做相同的工作，并且希望通过为每种数据类型重用相同的公共逻辑来减少重复。或者，想在store中处理某种类型数据的多个实例。但是redux有一些取舍，他可以轻松跟踪程序的整体状态，但是很难定位特定的状态动作，尤其是combineReducer。

比如我们在combine的时候发起了一个action，所有的都发生了这个action，我们想单独指向一个reducer就比较困难。

### 高阶reducer自定义行为
高阶reducer是一个方法，接收一个reducer方法，返回一个新的reducer方法。也可以看做是reducer的工厂，combineReducers就是一个高阶reducer，我们可以用类似的方法来包装reducer，比如给action加上额外的前缀或后缀，或者在高阶中判断是谁在发生这个action。

我们也能在某种程度上改变方法，写一个通用的reducer包裹器，挺容易看懂的。

或者写成判断的形式。挺好玩的方法。

## Immutable Update Patterns
### 更新嵌套数据
更新嵌套数据的关键是嵌套的每一层都要被适当的复制和更新。更新嵌套对象可能会导致直接突变，应该避免。

 - 比如我们state.a = action.data；就是直接突变数据
 - 第二个错是我们只处理了最上面一层state的拷贝，对下面进行了突变，也是会挂的

正确的是拷贝所有层的嵌套数据

不幸的是，将不可变的更新应用于深层嵌套的过程很容易变得冗长而难以阅读。所以我们应该尽量使state平坦。

倒是可以抽空看下我们的immutable怎么写的。

## Initializing State
两种方法来初始化应用state，一种是createStore可以接受可选的preloadedState作为二参。或者reducer本身来判断，如果没有传个默认值或者直接通过ES6的语法。

## Using Immutable.JS with Redux
### Why should I use an immutable-focused library such as Immutable.JS
Immutable旨在克服js中固有的不变性问题，为应用程序所需的性能提供不变性的所有好处。你是选择库还是空白的js对象，取决于你是否舒服加入一个新的依赖和是否能处理js的突变。

### 为什么用Immutable.js
 - 保证不变性：这个和js不一样，js有些操作突变数据pop，push等等，有些并不，比如map，filter，concat，foreach。
 - 丰富的api，多种数据结构
 - Performance：使用不可变数据结构会涉及大量的昂贵的复制，尤其是复杂的嵌套数据会有许多的中间拷贝。消耗内存损耗性能。Immutable通过共享表面下的数据结构来避免，最大程度减少复制必要。

### 坏处
 - 很难与简单的js对象相互操作，你没法直接访问对象属性，你只能通过get或者getIn方法。这就很难与你自己的代码以及第三方库交流，比如lodash等等，他们其实都期望是简单的js对象。虽然提供了toJS方法，但是他很慢，广泛使用会损害性能优势。
 - Once used, Immutable.JS will spread throughout your codebase，一旦用了，你的代码库里会有很多get。而且很难删掉了
 - 没有解构或者扩展运算符
 - 当你的数据很少的时候优势不明显
 - 很难debug
 - 打破了对象引用，导致低性能

### 值得吗
值得，因为突变导致的问题非常难复查出来。但是追查一些规则：

 - 永远不要将js对象和Immutable混合
 - 让你的redux为一个Immutable对象
 - 除了展示型组件之外，全都用Immutable
 - 少用toJS
 - selector应该返回Immutable
 - 不要在mapStateToProps中使用toJS
 - 使用chrome的插件来阻止debug

## General
什么时候使用：

在你发现你有大量数据改变的时候用redux，当你发现你在top component管理着所有的数据的时候。他并不是写代码最快和最短的方法，他是来帮助回答“什么时候有状态的改变，数据来自哪里的”！！！他通过约束你来实现！！他是一个伟大的工具

redux只能被react用吗：
都可以啦，最好是能推断UI的变化从state的改变，例如React。

需要额外的构建工具吗：

不用，方便的

## Reducers
我在两个reducer之间如何分享数据，我是否需要combineReducer

redux的store的建议结构是通过键来将state分为多个切片，提供一个单独的reducer方法来管理每个独立的数据切片。这和Flux搞多个独立的store一样，redux提供combineReducer来让模式更简单。

很多用户想要在reducer之间分享数据，但是发现combineReducer不允许，有几种实现：

 - 如果一个reducer需要从另一个切片知道数据，可能状态树就需要重构了，让单个reducer处理更多数据。
 - 可能需要写替代combineReducer的东西来自定义了，类似于reduce-reducers
 - redux-thunk这种可以通过getState拿到整个state，然后通过action传入

原则就是reducer只是方法，我们可以管理他们并且随意拆分。

### 是否需要switch来管理actions
这个自愿的，可以if，可以是查找table，或者找个方法静态化

## Organizing State
### 是否需要传入所有的state进redux，是否该用setState
没有正确答案，有些希望redux管理每个部分的数据，来保持全部的序列化以及应用的可控的版本。其他人希望将非关键的或者是UI的状态，保存在组件的内部。

一些经验法则：

 - 是否应用的其他部分关心这些数据
 - 你是否需要从这原始数据创造衍生数据
 - 是否同样的数据来驱动多个组件
 - 将这数据恢复对你来说是否有价值
 - 你是否想要缓存数据

### 在stata里面能放方法，promise或者其他非序列化的东西吗
极力推荐放简单的序列化数据，技术上可以放非序列化的，但是破坏了持久化能力和数据的再补充

自己权衡吧，这是你的应用

### How do I organize nested or duplicate data in my state
具有ID，嵌套或者关系的数据应该以标准化的方式存储：每个对象只应该被存一次，任何的引用就应该存ID而不是对象的复制。

## Store Setup
### 是否可以多个store，是否可以直接引入store，然后在组件内部直接用
最原始的Flux是多个store的，每个有不同的domain。这就会导致一个问题，一个store需要等待另一个store更新。这在Redux中是不必要的，因为数据的分隔是被reducer变成了更小的reducer的。

在Redux中使用多个商店的有效原因可能包括：

 - 部分状态的更新频繁导致的性能问题
 - 在更大的应用程序中将Redux隔离为组件

同样的，传store也不是推荐模式

### 在我的store增强中使用多个中间件好吗？在中间件功能中next和dispatch有什么区别
中间件链式调用，每个中间件功能可以调用next来传递给下一个中间件，调用dispatch来重新触发链式的开头，或者什么都不做只是暂停处理。定义多条链式会有问题的，因为会有不同的dispatch实例。

### 我怎么只监听一小部分的state，我能不能拿到action的内容作为订阅的内容
监听只是为了根据值来处理事情，而不是针对action，如果非要的话，就用中间件。

## Actions
### 为什么type是string，为什么action type必须是常量
可序列化的action可以实现redux的功能，因为我们需要可序列化的，能记录历史的东西，才能时间调试，记录和回放action。用Symbol就做不到，因为序列化的结果不可控。因为其实我们不一定要存state的结果，我们可能只要一份原始数据和action的列表就行了。我们可以用Symbol，promise等等来做type，如果我们使用了中间件的话。

Redux只检查了action是个简单的对象并且有type，其他我们都是可以自定义的。

### 在reducer和action之间是否一定有个一对一的映射
并不一定，我们可以让all，some或者none来响应。取决于我们自己的设计。

### 如何表示ajax这样的副作用，为什么我需要action creator，thunk和中间件来做异步的行为。
现在的应用需要发起ajax这样的请求来获取数据，所以code不再是input进入一个function，而有了与外界的交互，也就是副作用。

redux受函数式编程影响，开箱即用，没有地方来执行副作用，特别是reducer要求是纯函数。但是Redux的中间件允许拦截发起的请求，并且在周围添加复杂行为，包括副作用。

一般，redux觉得将副作用的作为action创建过程中的事情来做。尽管我们可以在UI组件的内部来写这个，不过将逻辑拿出来可以更好的复用。

最简单的我们可以用thunk来写复杂的异步的逻辑，我们也可以用saga来写看上去更同步的逻辑。还有个Redux Loop在reducer里面声明副作用来响应state改变。

### 我应该从一个动作创建者分发多个action吗？
没有规范我们如何管理action。如果只发一个，我们就比较难debug到发生了什么，如果发多个的话，就会觉得是不是应该只发一个。

应该尽量避免同步发多个dispatch，有大量的插件和方法来做这件事。

## Immutable Data
### 不变性的好处
不变性能带来更高的性能，导致更简单的编程和调试，因为永不改变的数据比任意自由改变的数据更容易理解。

特别是，不变性使得能够简单的实现复杂的变化检测技术，确保更新DOM这种昂贵的操作只在必须发生时才发生。

### 为什么Redux要求不变性
React和redux都是浅检测，特别是combineReducer，react-redux的connect也是通过浅检测来判断是否需要re-render。

不变性让数据处理更安全。

time-travel调试需要reducer是纯函数。

### redux为什么使用浅检测
主要是为了性能考虑。

### react-redux是怎么使用shallow比较来决定是否重绘的
connect函数中传入的props一定不能是新对象，他会对props里面的所有值进行浅比较。如果是新对象，啧啧啧

### 如何处理数据的突变，非要使用Immutable.js吗？
不一定需要，简单的js对象，正确书写也可以完美解决。当然很容易出问题而且难以排查。

## Code Structure
### 逻辑放在reducer还是action
感觉reducer轻量更容易复用啊，action如果太轻了。reducer就会变得复杂，而且需要大量的输入。文档让我们自己选择。

## Performance
### redux在规模化了之后在性能和架构方面有多好
规模化之后必然有性能问题，这其实与redux无关。React-Redux还做了优化的。

许多独立的组件应该自己去连state。list应该传Id给他们的children。用可缓存的selector来做也是性能优化。

### 每个action call all reducers会有性能问题吗
react实际上只有一个reducer，其实没什么性能问题。如果担心的话，可以加一些redux-ignore来精准操作或者先计量一下时间。

### 我需要在reducer深克隆吗，会变慢吗
都是浅拷贝啦

### 如何减少store更新事件的数量
如果是react，我们可以用ReactDOM.unstable_batchedUpdates()来batch操作。

如果单纯的reducer，可以用redux-batched-actions来将多个action batch，然后在reducer里面解压。

## React Redux
### 为什么组件重渲，为什么跑了mapStateToProps？
意外地直接修改状态是目前为止组件在调度后不会重新呈现的最常见原因。Redux期望您的reducer不可变。意味着总是复制，并且将更改应用于副本。如果我们要改变某个嵌套的，那一顺溜的都要复制。

并不意味着我们一定需要Immutable.js，我们也可以用js对象来做，或者Object.assign或者spread操作符。

### 为什么我的组件经常的rerender
React Redux做了一些优化来保证你的组件在真正需要的时候重绘。包括在mapStateTpProps和mapDispatchToProps生成的对象做了shallow equal。但是在每次都得到新引用的情况下就挂了。

有一个好选择是用reselect来缓存或者你自己在shouldComponentUpdate中用_.equal来判断。还有的时候我们会传箭头函数，我们可以在constructor里面bind。

### 有的时候dispatch为什么没有了
因为你不写的话会有默认dispatch，但是你自己写的话，就会被覆盖了。

### 我应该只连顶层组件？我可以在我的树里连多个组件吗？
早期redux文档建议你只应该在接近顶层的地方连接。这会导致一些组件知道太多子组件的需求，并且传太多的props。

最佳实践是将组件分为展示或者容器组件。将展示组件分离。

而且连接较多的组件通常比较少的组件性能更好。

在数据流和你的组件职责之间找到一个平衡点。

## Miscellaneous

## Troubleshooting
### dispatch action的时候啥也没有发生
可能是突变了数据。你必须返回新的数据。注意可能是需要深拷贝的。

### 别忘了调用dispatch(action)
千万别只返回一个action，要记着真正去dispatch他。

## Glossary
### state
随意的数据，最好能容易的序列化。

### action
简单的对象，代表了改变状态的意图。唯一能改变store的东西，所以的东西都要变成action来改变。

最好用string作为type，因为他好序列化，Symbol不能序列化的。

除了type之外action对象完全取决于你自己。

### reducer
一个方法接收state和action来生成新的state。必须是纯函数。而且没有side effect。

### dispatching function

### action creator

### async action

### middleware

### store
包含了state树。有dispatch，getState，subscribe还有replaceReducer来热加载和代码分离

### Store creator
创建store

### Store enhancer
store增强，允许我们给store加新的接口，store并不是一个实例，而是一个对象集合，可以很容易被创建。

## API Reference
记住redux只关心管理状态，在一个真的应用中，我们还需要UI绑定，例如react-redux。

## createStore
一参为reducer，二参是初始state，三参是store的增强，

## store
store不是class，他是一个拥有一些方法的对象，
