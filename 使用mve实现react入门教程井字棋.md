# 使用mve实现react入门教程井字棋



我找到的教程在这里 [入门教程: 认识 React](https://react.docschina.org/tutorial/tutorial.html)

 

## 环境准备

参考 [搭建mve的webpack开发环境](https://react.docschina.org/tutorial/tutorial.html)



## 游戏规则

摸索了一下，这个游戏是横、竖、斜线上三个连续相同，则取胜利的标准应该是一个算法——教程里直接提供了。

另外教程有历史状态，可以穿越状态，我感觉直接保存全局状态在这个场合下实现很便利，但现实中（如大型数据库）可能很难，所以要实现undo-redo栈。

