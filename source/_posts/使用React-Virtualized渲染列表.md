---
title: 使用React Virtualized渲染列表(译)
date: 2019-04-20 22:05:57
tags:
---

[原文链接](https://css-tricks.com/rendering-lists-using-react-virtualized/)

使用React处理数据相对来说比较容易，因为React的设计就是把数据当作状态。但是当你需要处理的数据量很大的时候，麻烦就来了。比如你要处理一个包含500-1000条记录的数据集，这会产生巨大的计算量并导致性能问题。下面我们将学习如何使用虚拟列表来“看起来”渲染了一个长列表。

我们将使用[React Virtualized][]组件来实现我们的需求。它让我们可以以很小的代价渲染大集合数据。

## 设置

React Virtualized官方已经有很详细的介绍了，可以去它们的[github][React
Virtualized]去看看。

我们需要大量的数据，下面我们就来造一些。

```javascript
function createRecord(count) {
  let records = [];

  for (let i = 0; i < count; i++) {
    records.push({
      username: faker.internet.userName(),
      email: faker.internet.email()
    });
  }
  return records;
}
```

下面，我们设置一个数字来创造我们需要的数据：

    const records = createRecord(1000);

好了，现在我们有需要渲染的数据了。

## 创建一个虚拟列表

[这里](https://codepen.io/kinsomicrote/pen/NOQXeK)是我们创建的一个列表，我们引入使用了库提供的一些展示样式，本篇post不讨论这个。
现在开始感受一下这个[demo](https://codepen.io/kinsomicrote/pen/NOQXeK)，速度超级快，是不是？

你可能想知道这背后到底发生了什么，结果发现是一系列很疯狂和酷的sizing、positioning、transform和transitions，是这些技术让一条条记录进入/离开可视区。数据都在那里并渲染了，React Virtualized创建了一个window，当用户scroll的时候，一条条记录将滑入/出我们的视野。

为了渲染虚拟列表，我们需要`List`组件，它内部渲染了一个`Grid`组件。

首先，我们从设置`rowRenderer`，开始，它是负责渲染单挑数据的组件。

```javascript
rowRenderer = ({ index, isScrolling, key, style }) => {
    return (
      <div key={key} style={style}>
        <div>{this.props.data[index].username}</div>
        <div>{this.props.data[index].email}</div>
      </div>
    );
  };
```

它返回一个包含两个`div`的div`，里面的两个`div`分别是`username`和`email`。可以看出，这是一个简单的展示用户信息的列表。

`rowRenderer`接受几个参数，下面是这些参数的细节：

* `index`: 记录的数值ID
* `isScrolling`: 代表`List`组件是否发生scrolling
* `isVisible`: 代表这条数据是否在可视区内
* `key`: 这条记录在数组中的位置
* `parent`: 定义这个列表是否是另一个列表的parent/child
* `style`: 定位这条数据的style对象

下面我们再深入了解一些`rowRenderer`函数，我们把它放到`List`组件中：

```javascript
<List
  rowCount={this.props.data.length}
  width={width}
  height={height}
  rowHeight={rowHeight}
  rowRenderer={this.rowRenderer}
  overscanRowCount={3}
/>
```

你可能注意到这里的几个参数：

* `rowCount`: 接收代表列表长度的数字
* `width`: 列表的宽度
* `height`: 列表的高度
* `rowHeight`: 每条数据的高度
* `rowRenderer`: 用来渲染每条数据的模板，我们将传入之前定义的`rowRenderer`函数
* `overscanRowCount`:
  用来渲染用户scroll方向额外的数据，防止用户滑动太快，虚拟内容来不及渲染。

最后，代码应该是这样的：

```javascript
const { List } = ReactVirtualized

...

const height = 700;
const rowHeight = 40;
const width = 800;

class App extends React.Component {
  rowRenderer = ({ index, isScrolling, key, style }) => {
    return (
      <div key={key} style={style}>
        <div>{this.props.data[index].username}</div>
        <div>{this.props.data[index].email}</div>
      </div>
    );
  };

  render() {
    return (
      <div>
        <h2>Details</h2>
        <List
          rowCount={this.props.data.length}
          width={width}
          height={height}
          rowHeight={rowHeight}
          rowRenderer={this.rowRenderer}
          overscanRowCount={3}
        />
      </div>
    );
  }
}
```

## Cell measure

文档里介绍，cell
measure是一个高阶组件，用来暂时渲染列表。现在我们还看不到它，但数据已经在里面被处理并准备好展示了。

什么时候我们需要关心cell measure？最常见的use
case是当我们需要动态计算`rowHeight`的时候。React
Virtualized在渲染每一行的时候，会缓存它们的高度值，这样当数据滑出可视区的时候我们也不用再计算它的高度——不管里面的内容是什么，高度都是对的。

首先，我们创建自己的缓存`cache`，在我们组件的`constructor`里用`CellMeasureCache`：

```javascript
constructor() {
  super()
  this.cache = new CellMeasurerCache({
    fixedWidth: true,
    defaultHeight: 100
  })
}
```

当我们设置`List`组件的时候，把`cache`带上：

```javascript
<List
  rowCount={this.props.data.length}
  width={rowWidth}
  height={listHeight}
  deferredMeasurementCache={this.cache}
  rowHeight={this.cache.rowHeight}
  rowRenderer={this.renderRow}
  overscanRowCount={3}
/>
```

传给`deferredMeasurementCache`的值会被用来暂时渲染数据，接着——当`rowHeight`的计算结果出来的时候——额外的行会流入，就像它们一直在那里。

接着我们将在`rowRenderer`函数里使用`CellMeasure`替换我们之前的`div`：

```javascript
rowRenderer = ({ index, parent, key, style }) => {
  return (
    <CellMeasurer
      key={key}
      cache={this.cache}
      parent={parent}
      columnIndex={0}
      rowIndex={index}
    >
      <div style={style}>
        <div>{this.props.data[index].username}</div>
        <div>{this.props.data[index].email}</div>
      </div>
    </CellMeasurer>
  );
};
```

现在数据已经被获取、缓存并准备好展示在虚拟window里了！

## 虚拟table

虽然本片post主要说列表，但万一当我们需要渲染table怎么办？React
Virtualized也帮我们做了这件事情。这时我们需要使用`Table`和`Column`组件。

下面是代码：

```javascript
class App extends React.Component {
  render() {
    return (
      <div>
        <h2>Details</h2>
        <Table
          width={500}
          height={300}
          headerHeight={20}
          rowHeight={40}
          rowCount={this.props.data.length}
          rowGetter={({ index }) => this.props.data[index]}
        >
          <Column
            label='Username'
            dataKey='username'
            width={100}
          />

          <Column
            width={200}
            label='Email'
            dataKey='email'
          />
        </Table>
      </div>
    );
  }
}
```

这个`Table`组件包括下面的参数：

* `width`
* `height`
* `headerHeight`
* `rowHeight`
* `rowCount`
* `rowGetter`: 返回这行的数据

如果看一下`Column`组件，你会发现我们设置了一个`dataKey`参数。它是每条数据拥有的独一无二的id。

## 总结

希望这篇post可以帮你了解React
Virtualized可以做哪些事情，它如何让列表渲染变得很快，并如何在项目中使用它。

我们只讨论了皮毛，这个库覆盖了更多的use case，如在scroll的时候为记录generate
placeholders、实时获取/缓存数据的无限加载组件等等。

它将给你很多可以play with的东西！

此外，这个包维护的很好，实际上你可以加入[Slack group](https://react-virtualized.now.sh/)来跟踪这个项目，贡献它，和其他folks取得联系。

还有一条值得注意的是，React Virtualized在[StackOverflow上有它自己的标签](https://stackoverflow.com/questions/tagged/react-virtualized)，这是一个寻找问题的答案的地方，也可以po出你的问题。

[React Virtualized]: https://github.com/bvaughn/react-virtualized
