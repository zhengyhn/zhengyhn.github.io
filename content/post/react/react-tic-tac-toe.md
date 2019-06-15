---
title: React入门-井字游戏实现与完善
date: "2019-06-15"
tags: ["React"]
categories: ["React"]
---

前段时间换了家外企工作, 空闲时间比较多。

虽然我是做Java后端的，但是老外喜欢搞敏捷开发和全栈，所以也要写前端，既然是老外，那肯定是喜欢用React的，然而我之前从来没写过React，只能从基础一步一步来了。

## **井字游戏**
井字游戏是React官方的入门实战教程的案例来的，在[这里](https://reactjs.org/tutorial/tutorial.html)。

这个游戏的规则是这样的:

- 一共9个格子的棋盘, 共2种棋子，分别是X和O
- 每次，一个玩家将棋子下到一个格子上
- 如果有3个相同棋子在横/竖/对角线上相连，那就赢了，否则就为和局

最后的效果是这样的：

{{< figure src="/images/tic_tac_toe_original.gif" title="original tic tac toe" >}}

## **实现**

我的实现就是完全跟着官方教程来的，最后挑了一些有趣的课后作业完成，分别新增了下面的功能：

- 点击历史列表某一项时，这一项的文字加粗
- 将原来硬编码写的九宫格改成用循环
- 如果是平局，在界面上显示出来

都是比较简单的功能，我还想到了一个功能，就是给一个回放的按钮，点击可以回放整个游戏过程。

所以，增加了上面的4个功能的实现，最终的效果变成:

{{< figure src="/images/tic_tac_toe_improved.gif" title="improved tic tac toe" >}}

### **点击历史列表某一项时，这一项的文字加粗**

在state里面加一个字段selectedIndex，表明现在选择的是哪一个历史项，这个字段会在渲染的时候使用。

```
    this.state = {
      history: [
        {
          squares: Array(9).fill(null),
        },
      ],
      currentStep: 0,
      selectedIndex: -1,
      xIsNext: true
    };
```

在渲染的时候，代码改成这样：

```
    const moves = history.map((item, index) => {
      const desc = index > 0 ? `Go to move ${index}` : `Go to game start`
      const textHtml = this.state.selectedIndex === index ? (<b>{desc}</b>) : desc
      return (
        <li key={index}>
          <button onClick={() => this.jumpTo(index)}>{textHtml}</button>
        </li>
      )
    })
```

在生成整个历史列表时，要遍历每一项，这个时候，判断状态里面保存的选择的下标是不是跟某一项的下标相等，如果相等就在文字外面包一个加粗的标签。

在点击某一项时，需要设置selectedIndex的值，代码变成这样子:

```
  jumpTo(i) {
    this.setState({
      currentStep: i,
      selectedIndex: i,
      xIsNext: (i & 1) === 0
    });
  }
```

### **将原来硬编码写的九宫格改成用循环**

这里有点坑，本来我想用双重循环的，比如这样:
```
    let board = []
    for (let i = 0; i < 3; ++i) {
      board.push(<div className="board-row">)
      for (let j = 0; j < 3; ++j) {
        board.push((j) => {this.renderSquare(i * 3 + j)})
      }
      board.push(</div>)
    }
    return (
      <div>
        {board}
      </div>
    );
```
但是在这一行:
```
board.push(<div className="board-row">)
```
它认为我没闭合标签，认为下面的代码还是内容，会报错, 所以最后我只能改成用map来写了，双重循环应该是可以的，刚入门，不熟悉。

最后用map实现的代码长这样：

```
    let board = []
    for (let i = 0; i < 3; ++i) {
      board.push(
        <div className="board-row">
        {[0, 1, 2].map((j) => this.renderSquare(i * 3 + j))}
        </div>
      )
    }
    return (
      <div>
        {board}
      </div>
    );
```

### **如果是平局，在界面上显示出来**

这个很简单，直接加一个判断，如果没有计算出来winner，并且全部棋子都摆满了棋盘，就认为是平局。代码如下：

```
    if (winner) {
      status = `Winner: ${winner}`
    } else if (history.length > 9) {
      status = `No Winner, it's draw`
    }
```

### **回放功能**

实现也比较简单，思路就是用history保存的状态，用setInterval重新设置一遍，在遍历到最后一个状态时用clearInterval停止。

首先，加个按钮:

```
        <div className="game-info">
          <div>{status}</div>
          <ol>{moves}</ol>
          <div><button onClick={() => this.playBack()}>Play back</button></div>
        </div>
```

然后看这个按钮调用的逻辑：

```
  playBack() {
    this.setState({
      currentStep: 0,
      selectedIndex: 0,
      xIsNext: true
    });
    var intervalID = setInterval(() => {
      this.setState({
        currentStep: this.state.currentStep + 1,
        selectedIndex: this.state.selectedIndex + 1,
        xIsNext: (this.state.currentStep & 1) === 0
      });     
      if (this.state.currentStep >= this.state.history.length - 1) {
        clearInterval(intervalID);
        this.setState({
          xIsNext: (this.state.currentStep & 1) === 0
        })
      }
    }, 1000)
  }
```

整个实现就是这样的了！忘了要把源码附上，在[这里](https://github.com/zhengyhn/react-practise/tree/master/tic-tac-toe)
