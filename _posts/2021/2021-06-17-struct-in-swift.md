---
layout: article
title: struct in swift
tags: ['iOS','swift']
---

## immutable struct

struct 和 class 类似，但是由于 class 支持继承等特性，所以在 OOP 编程中，更经常使用 class，往往忽略了 struct。但在 iOS 引入 swiftUI 之后，可以看到很多 struct 的影子，有必要来关注下 struct 的特性，避免在掉入一些陷阱中。

```swift
struct Student {
    var age: Int
}

var s = Student(age: 12)
s.age = 13
print(s.age)  //13
```
上面是一段平平无奇的 struct 代码，你甚至不会想在 Xcode 中运行它。现在我们为 `s` 添加一个 `didSet` 的属性观察器。

```swift
struct Student {
    var age: Int
}

var s = Student(age: 12) {
    didSet {
        print("s changed : \(t)")
    }
}
s.age = 13
print(s.age)  //13
```
再次运行会发现 `didSet` 中的代码被执行了，这意味着 `s` 被改变了！ `s` 被改变的原因在于：**struct 是值类型，值类型意味着不可改变**。上面代码中，我们对 `age` 进行了修改，会导致新创建一个 `age = 13` 的结构体，然后将新结构体重新赋值给 `s`，所以表面上看起来 `s` 还是那个 `s`，但实际上已经物是人非了。

👆示例代码总结出来：**struct 是值类型，是不可改变的(immutable)，改变属性的值会新创建一整个结构体重新赋值**。

由于 struct 是值类型，所以在 struct 的方法中是无法修改属性值的。
```swift
struct Student {
    var age: Int

    func addAge() {
        self.age += 1  // Error!!🥵
    }
}
```
在 `addAge` 中试图对 `age` 进行修改会导致编译器报错，**如果一定要在方法中修改 struct 的属性，那么方法需要使用  `mutating` 修饰**

```swift

struct Student {
    var age: Int
    mutating func addAge() {
        self.age += 1  // OK  🥳
    }
}

var s = Student(age: 12) {
    didSet {
        print("s changed : \(t)")  // 仍然被执行
    }
}
s.addAge()
```
使用 `mutating` 修饰之后可以在方法中修改属性值，但是本质上，还是新创建了一个结构体实例替换了原本的 `s`。

那么这个特性在 swiftUI 中有何体现？

## `@State with struct`

下面的 swiftUI 代码肯定很熟悉

```swift
struct User {
    var name:String = ""
}

struct ContentView: View {
    
    @State var user = User()

    var body: some View {
        VStack{
            Text("hello \(user.name)")
            TextField("input your name", text: $user.name)
        }
    }
}
```
使用 `@State` 修饰之后，每次 `user` 变化，都会及时的通知 UI 更新数据。`@State` 是如何做到的呢？

注意到，`User` 是使用 struct 定义的，而不是 class，那么如同上面我们提到的，每次修改 `user.name` 的值，其实都会创建一个新的 `User` 实例重新赋值给 `user` 变量，`@State` 观察到 `user` 变化，便通知 UI 进行更新

如果把 `User` 从 struct 改成 class，会发现 UI 不会更新，这是因为 class 是「引用类型」，属性值变化不会创建一个新的 `User class` 赋值给 `user` 变量，`user` 变量没有变化，UI 也无法收到通知。
