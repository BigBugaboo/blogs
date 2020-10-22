---
title: test
date: 2020-10-05 01:10:22
tags:
---

## 什么是泛型？

> 喜欢更多细节和原理的朋友，可以前往官网，阅读文档。[官网传送门](https://www.typescriptlang.org/docs/handbook/generics.html)  [中文版传送门](https://www.tslang.cn/docs/handbook/generics.html)
---

如果是科班出身，对编译型语言有一定了解的话，可以看看这句话，就能初步理解了个大概。
[泛型是程序设计语言的一种特性。允许程序员在强类型程序设计语言中编写代码时定义一些可变部分，那些部分在使用前必须作出指明。](https://baike.baidu.com/item/%E6%B3%9B%E5%9E%8B/4475207?fr=aladdin)**这里提及的可变部分必须作出声明，typescript 可以自动推断类型，所以声明不是必须的。**

然后，我们用通俗一点的语言，来进一步了解 typescript 的泛型，举个例子：
我是一个机器人，专门负责把我拿到的钱，变成硬币。我没加泛型之前，我只收漂亮国的钱。后来，我免费了，加入了泛型，可以处理各种不同国家的钱，再返回对应国家的硬币。你可以告诉我，你给我什么钱，你也可以不告诉我，我自己判断。

就从上面上面例子中，其实可以发现，在没加入泛型之前，为了低容错，会声明某些固定类型，比如，只转漂亮国的钱。当加入泛型后，我们可以让函数变为**纯函数**，依赖参数，用**泛型变量**来定义相关参数的类型，实现类似动态类型的概念，做出行为一致结果,**拿什么钱，转什么钱**。所以泛型更考验开发者的能力，因为在编写代码时，要保证其在定义的泛型场景下，能通过。当然，函数虽然可以转很多国家的钱，可能它不想转冥币，所以我们也必须声明好泛型的适用的场景，就要加**泛型约束**。

## 泛型怎么用？
这一部分，就用官方的例子来讲一下。

下面这个例子，一个函数，声明形参和函数返回值的类型`number`，那么这种情况下，函数只能接受对应的实参类型的调用。
```typescript
function identity(arg: number): number {
    return arg;
}
```
使用了泛型的函数，长这样
```typescript
function identity<T>(arg: T): T {
    return arg;
}

const a = identity('Hello');
const b = identity<string>('world');

console.log(a + ' ' + b); // Hello world
```
T 可以理解成一个形参变量，那么 `identity` 函数的 arg 和其返回值类型都是用 T 变量。回到这节内容，我们聊到怎么用泛型，在概念上，其实跟怎么用函数差不多，T 如果是形参，那么调用就会有实参，`identity<string>('world')`用`<>`标示 T 的类型，是不是有函数方法调用那味了？当你再看一遍，你会发现其实我们不一定要传类型，因为 T 相当于带了默认值的参数，不过这默认值是自动推断的，推断 `'Hello'` 的类型。



## 什么时候该用泛型？

在大部分的开发场景下，都没必要使用泛型，因为不用泛型也能正常写业务，正常写工具。泛型，更多是附加价值。比方，写个万金油的工具，和单一业务工具，功能上是单一的，但是应用场景更广了，在价值层面上不同。
所以我更推荐在公共工具、有输出行为的组件及函数等，个人看法，如果大家有其他等想法，可以在下方评论区留言。

我们看几个例子，第一个最经典的例子，如果使用react + typescript 进行开发，第一个接触的泛型，很可能是 useState 泛型函数。
```typescript
import * as React from "react";

const FunctionComponent: React.FC<{}> = () => {
  const [count, setCount] = React.useState(1); // 根据上面提及的，在 typescript 中的泛型不一定需要声明类型，它会自动推断，count: number
  // 声明类型
  // const [count, setCount] = React.useState<number>(1);

  const handleClick = (e: React.MouseEvent<HTMLDivElement>): void => {
    const newCount = count + 1;
    setCount(newCount);
  };

  return <div onClick={handleClick}>{count}</div>;
};

export default FunctionComponent

```

在上面的例子可以发现，useState 是有输出行为的函数，泛型可以保证 count 和 setCount 的类型校验，我们不用繁琐的声明 count 和 setCount 的类型。看完整个过程，是不是可以抽象理解成，泛型类似“**类型变量**”，useState 把类型当作变量进行使用传递。从这个点，我们再思考一下，如果有变量的概念，是不是会存在逻辑的可能。那么接下来我们介绍有逻辑的泛型工具，用逻辑操纵类型。
