---
title: typescript 工具泛型
date: 2020-08-31 22:10:30
tags: typescript
---

**提要**
> 1. 什么是工具泛型？
> 2. 学习工具泛型前置条件？
> 3. 学习官方提供的工具泛型的实现方式
> 4. 加深回顾
> 5. 总结
> 6. 题目

**背景**
一直没有深入学习更多内容，后来加入观麦前端团队，有辛认识到有着 `TS` 丰富经验的前辈，开始意识到自己在 `TS` 方面还有很多盲区，所以选择跳出来，往更深的内容学习。现在与大家分享工具泛型相关的学习，有什么不足的地方，希望能和大家共同交流。如果不想看的，可以直接请官网学习
[官网传送门](https://www.typescriptlang.org/docs/handbook/utility-types.html)

## 什么是工具泛型？

处理一些类型相关的工具泛型，有些人也叫它`工具类型`，因为它具有一定的工具性质，就是类似纯函数的概念。

## 学习工具泛型前置条件？

### type 类型别名

> 顾名思义，你可以理解成，给类型取个名字

```typescript
type Person = {
  name: string
  age: number
}

const me: Person = {
  name: 'front-end developer',
  age: 66,
}
```

同时我们也可以用来声明工具泛型
```typescript
type TPickKey<T> = keyof T

interface Person {
  name: string
  age: number
}

const myKey: TPickKey<Person> = 'name'
// TPickKey<Person> = 'name' | 'age'
// 这个泛型主要获取传入类型的 key，并放回key的联和类型
```

### 条件类型

在`TS` 2.8 加入了条件类型,逻辑是三元运算符，所以比较好理解
```typescript
T extends U ? X : Y
```

### infer

声明一个变量，并对其进行使用,可以看后面的`ReturnType`进行理解

### extends 继承

下面的一些内容会经常用`extends`关键字,`extends`类似数学集合里的子集概念,可以看一下这篇文章，会对`TS`有进一步对理解。[深入Typescript 的类型系统](https://zhuanlan.zhihu.com/p/38081852)

### keyof

> keyof 操作符是在 TypeScript 2.1 版本引入的，操作符可以用于获取某种类型的所有键，其返回类型是联合类型。

```typescript
interface Person {
  name: string
}

type Keys = keyof Person // Keys = "name"
```

### typeof

> typeof 操作符可以用来获取一个变量或对象的类型

```typescript
interface Person {
  name: string
}

const me: Person = {
  name: 'bugaboo'
}
type MyType = typeof me // MyType = Person
type MyNameType = typeof me.name // MyNameType = string
```

### in
遍历联合类型
可以查看文章中的实现机制

### - （横杠）
逻辑取反，参考Required<T>
对比文章中Partial<T>的Required<T>两个的代码，可以非常快速理解。

## 学习官方提供的工具泛型的实现方式

在编写复杂的工具泛型前，我们必须要掌握基础工具及语法，才能更好的理解和实现。

*请先理解上一节的内容*

### 首先我们来认识一些常用的命名规则，这里参考 `java` 的泛型命名规范

- E - Element (used extensively by the Java Collections Framename)
- K - Key
- N - Number
- T - Type
- V - Value
- S, U, V etc. - 2nd, 3rd, 4th types
- R - Result
- A - Accumulator

### Partial<T>

> 把传入的类型，变为可选的

```typescript
interface Person {
  name: string
  age: number
}

const you: Person = {
  name: 'yyds',
  age: 18,
}

const me: Partial<Person> = {
  name: 'bugaboo',
}
//  typeof me = {
//   name?: string
//   age?: number
// }
```
当我在认识一个未知的东西时，我需要对它尽可能的全面了解，那么它的遍历会很深吗？

```typescript
interface HobbyType {
  title: string
  desc: string
}

interface Person {
  name: string
  age: number
  hobby: HobbyType[]
}

const you: Person = {
  name: 'yyds',
  age: 18,
  hobby: [{ title: 'play game', desc: 'many games' }]
}

const me: Partial<Person> = {
  name: 'bugaboo',
  hobby: [{}] // 类型“{}”缺少类型“HobbyType”中的以下属性: title, descts(2739)
}
```

实现

```typescript
type MyPartial<T> = {
  [K in keyof T]?: T[K]
}

type MyType = MyPartial<{
  name: string
  age: number
}>
```
解析
1. (`type MyPartial<T>`) --- 首先，我们声明个工具泛型
2. (`K in keyof T`，这里`K`代表着`T`类型中的`key`) --- 遍历 `T`
3. (`T[K]`) --- 获取对应 `key` 的类型
4. (`?:`) --- 设置成可选

### Readonly<T>

> 把传入的类型变为只读状态

```typescript
interface Person {
  name: string
}

let me: Readonly<Person> = {
  name: 'bugaboo'
}

me.name = 'yyds' // 无法分配到 "name" ，因为它是只读属性。ts(2540)
```

实现

```typescript
type MyReadonly<T> = {
  readonly [K in keyof T]: T[K]
}

type MyType = MyReadonly<{
  name: string
}>
```
解析
1. (`type MyReadonly<T>`) --- 首先，我们声明个工具泛型
2. (`K in keyof T`，这里`K`代表着`T`类型中的`key`) --- 遍历 `T`
3. (`T[K]`) --- 获取对应 `key` 的类型
4. (`readonly`) --- 设置成只读


### Record<K, T>

> 将类型应用在联合类型上

```typescript
interface Person {
  name: string
}

type Peoples = Record<'汉族' | '少数民族', Person>
// type Peoples = {
//   汉族: Person
//   少数民族: Person
// }
```

实现。

```typescript
type MyRecord<K extends keyof any, T> = {
  [S in K]: T
}

type MyType = MyRecord<'汉族' | '少数名族', { name: '' }>
```
解析
1. (`type MyRecord`) --- 首先，我们声明个工具泛型
2. (`<K extends keyof any, T>`) --- 传入两个参
3. (`K extends keyof any`) --- `K` 继承 `any`,就是随便，爱传啥传啥，看上面我例子，我都传中文了。
4. (`S in K`) --- 遍历 `K`

### Pick<T, S>
> 在 T 中，过滤掉非 S 的类型

```typescript
interface Person {
  name: string
  age: number
}

type MyType = Pick<Person, 'name'>
// type MyType = {
//   name: string
// }
```

实现

```typescript
interface Person {
  name: string
}

type MyPick<T, S extends keyof T> = {
  [K in S]: T[K]
}

type MyType = MyPick<Person, 'name'>
```
解析
1. (`type MyPick`) --- 首先，我们声明个工具泛型
2. (`<T, S extends keyof T>`) --- 传入两个参
3. (`S extends keyof T`) --- 之前聊到 `keyof` 是获取联合类型,所以`S`是继承 `T` 的 `key` 值组成的联合类型。
4. (`K in S`) --- 遍历 `S`
5. (`T[K]`) --- 获取`T`中对应的类型

### Exclude<T, S>

```typescript
type T = Exclude<1 | 2, 2 | 3>
// type T = 1
```

实现
```typescript
type MyExclude<T, S> = T extends S ? never : T
```

解析
利用条件类型，判断`T`是否属于`S`的子集，然后返回`never`或`T`
对于联合类型会自动分发条件
```typescript
type T = Exclude<1 | 2, 2 | 3>
(1 extends 2 | 3 ?  : 2 | 3) | (2 extends 2 | 3 ? never : 2 | 3)
```
所以在传入联合类型时，会返回到两个类型集合的补集

### Omit<T, K>

在`T`中删除对应的`K`

```typescript
interface Person {
  name: string
  age: number
}

type Me = Omit<Person, 'name'>
// type Me = { age: number }
```

实现
```typescript
type MyOmit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>
```

解析
利用上面的 `Pick` 和 `Exclude`进行组合
1. 先找出补集的`keys`
2. 然后从类型集合里，挑它出来。

### ReturnType<T>

获取函数返回值的类型

```typescript
function foo(a: string): Array<string> {
  return [a]
}

type FooReturnType = ReturnType<typeof foo>
// type FooReturnType = string[]
```

实现
```typescript
type MyReturnType<T> = T extends (...arg: any[]) => infer R ? R : any
```

解析
利用`infer`代指函数的返回类型，然后通过 extends 判断是否是函数类型，再返回对应的`infer`代指的类型。

## 加深回顾

其实还没写全，我只把几个比较有特点的工具写了。剩下的，不过是基于类似的实现，稍微改动来一下。我列出来，大家可以尝试一下，自己实现。
1. Required - 把所有属性变为必选
```typescript
// tip: 跟 Partial 有关
type MyRequired<T> =  // 请自行脑补

type My = MyRequired<{ name?: string, age: number }>
// type My = { name: string }
```
2. Extract - 取出两个类型集合的交集
```typescript
// tip: 跟 Exclude 有关
type MyExtract<T, K> =  // 自行脑补

type My = MyExtract<1 | 2, 2>
// type My = 2
````

3. NonNullable - 去除null和undefined类型
```typescript
// tip: 利用条件类型的联合类型自动分发特点
type MyNonNullable =  // 自行脑补

type My = MyNonNullable<string | null | undefined>
// type My = string
```

4. Parameters - 获取函数的行参结构类型
```typescript
// tip: 跟 ReturnType 有关
type MyParameters<T> = {

}

type My = MyParameters<(arg: { name: string }) => string>
// type My = [arg: {
//     name: string;
// }]
```

还有一些不怎么常用的工具，可以在官网查看学习[官网传送门](https://www.typescriptlang.org/docs/handbook/utility-types.html)

## 总结
网上有很多相关的文章，大多都是源码结合解析，我也是如此。不过，按照我的学习路径，先总结了基本前置知识，比如`infer`、`条件判断`等，通过递进式的方式学习掌握。我没写全，因为我希望，通过我文章，总有点收获吧，可以自己把剩下的拿来练习。可能大家一开始会比较抵触，尤其是应用到实际中时，会比较考验解构的能力。

## 附加
[官方源码](https://github.com/microsoft/TypeScript/blob/9ff24b6fc87dd4a376a434d1019d356dfe743c53/lib/lib.es5.d.ts)