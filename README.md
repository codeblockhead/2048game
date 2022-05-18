# 项目总结

该项目是一个完全依赖原生 js 代码的文件,实现起来有一定难度;
直接使用 vscode 预览 index.html 即可运行;

## 核心代码:

2048 小游戏的核心是通过遍历顺序决定其移动
总结下如下两条规则（在不考虑方块合并的情况下）:

1. 规则 1：同一排或同一列的方块移动顺序跟随具体的方向，比如上图中：向上移动 2 先移动，4 后移动；向下移动 4 先移动，2 后移动。

2. 规则 2：每个方格都是移动到该方向的最后一个空白位置。
   ![Image text](https://style.youkeda.com/img/course/f10/4/2.jpeg)

### 代码实现-遍历顺序

我们设置一个方法，根据方向返回遍历路径，代码如下：

```js
//manager.js

Manager.prototype.getPaths(direction){
  let rowPath = [];
  let columnPath = [];
  return {rowPath, columnPath}
}
```

我们首先加入正常的左上到右下的遍历顺序，代码如下：

```js
Manager.prototype.getPaths = function (direction) {
  let rowPath = [];
  let columnPath = [];
  for (let i = 0; i < this.size; i++) {
    rowPath.push(i);
    columnPath.push(i);
  }
  return {
    rowPath,
    columnPath,
  };
};
```

- 当方向为向右的时候，遍历方向应该从右到左，columnPath 遍历方向应该反向。

- 当方向为向下的时候, 遍历方向应该从下到上，rowPath 遍历方向应该反向

这里解释一下遍历决定的原因:
假设第（1，1）（1，2）有两个 Tile，那么要先移动（1，1）Tile，再移动（1，2）Tile，否则第一个 Tile 会被挡住无法移动，所以必须从右到左遍历。
优化一下代码:

```js
Manager.prototype.getPaths = function (direction) {
  let rowPath = [];
  let columnPath = [];
  for (let i = 0; i < this.size; i++) {
    rowPath.push(i);
    columnPath.push(i);
  }

  // 向右的时候
  if (direction.column === 1) {
    columnPath = columnPath.reverse();
  }

  // 向下的时候
  if (direction.row === 1) {
    rowPath = rowPath.reverse();
  }
  return {
    rowPath,
    columnPath,
  };
};
```

### 代码实现-找寻目标位置

我们需要寻找方块移动的目标地址，根据规则 2，每个方格移动到此方向不能移动为止 :

```js
// 寻找移动方向目标位置
Manager.prototype.getNearestAvaibleAim = function (aim, direction) {
  // 位置 + 方向向量的计算公式
  function addVector(position, direction) {
    return {
      row: position.row + direction.row,
      column: position.column + direction.column,
    };
  }
  aim = addVector(aim, direction);

  // 获取grid中某个位置的元素
  let next = this.grid.get(aim);

  // 如果next元素存在（也就是此目标位置已经有Tile），或者是超出游戏边界，则跳出循环。目的：就是找到最后一个空白且不超过边界的方格
  while (!this.grid.outOfRange(aim) && !next) {
    aim = addVector(aim, direction);
    next = this.grid.get(aim);
  }

  // 这时候的aim总是多计算了一步，因此我们还原一下
  aim = {
    row: aim.row - direction.row,
    column: aim.column - direction.column,
  };

  return {
    aim,
    next,
  };
};
```

核心逻辑是如下，通过循环，找寻最后一个空白位置：

```js
// 如果next元素存在（也就是此目标位置已经有Tile，
// 或者是超出游戏边界，则跳出循环。
// 目的：就是找到最后一个空白且不超过边界的方格
while (!this.grid.outOfRange(aim) && !next) {
  aim = addVector(aim, direction);
  next = this.grid.get(aim);
}
```

## 小结

1. 整个项目其实还是有点庞大的。这个项目不是一个简单的项目，无论是在代码量，或者在数学判断逻辑上面都有一定的复杂度，对新手而言，可能难度比较偏大
2. 为了简化项目样式，使用 SCSS 拆分了 CSS 文件，并利用其变量，循环等特点，动态生成 Tile 位置代码
3. 同样，为了简化项目逻辑，利用对象，职责的思路拆分了 JS 代码，最终达到各司其职的作用

- ndex：入口文件
- Manager：游戏控制器
- Listener：游戏监听器
- Render：游戏渲染器
- Grid：游戏面板
- Tile：游戏方格
