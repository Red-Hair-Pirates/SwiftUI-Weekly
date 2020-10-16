# Animations in SwiftUI

原文：[Animations in SwiftUI](https://swiftwithmajid.com/2019/06/26/animations-in-swiftui/) 作者：[[Swift with Majid](https://swiftwithmajid.com/)]

翻译：阳阳    校对：小辉辉

Swift 为我们带来了声明式这样直接的方式来创建用户界面。我们有 List 和 Form 等组件还有各种绑定值(Bindings)。这些都让SwiftUI如此易用同时十分强大。不过今天我们准备聊一聊SwiftUI中的另一特性，它就是动画。

### Animation

你可以流畅的动画在SwiftUI通过包装变化进 withAnimation 块中。默认的，SwiftUI会用fade in和fade out 的方式来显示变化。让我们来看一个小例子、

```swift
struct ContentView : View {
    @State private var isButtonVisible = true

    var body: some View {
        VStack {
            Button(action: {
                withAnimation {
                    self.isButtonVisible.toggle()
                }
            }) {
                Text("Press me")
            }

            if isButtonVisible {
                Button(action: {}) {
                    Text("Hidden Button")
                }
            }
        }
    }
}
```

在这个例子中，我们把状态变化包装进 withAnimation block中，并且它产生了不错的fade in效果。你可以修改动画通过传递时间和spring值。另一个选择是在需要做动画的 view 后面使用 animation modifier。

```swift
struct ContentView : View {
    @State private var isButtonVisible = true

    var body: some View {
        VStack {
            Button(action: {
                self.isButtonVisible.toggle()
            }) {
                Text("Press me")
            }

            if isButtonVisible {
                Button(action: {}) {
                    Text("Hidden Button")
                }.animation(.easeInOut)
            }
        }
    }
}
```

这上面的示例代码中，我们得到相同的动画使用简单的添加 animation modifier。我们使用 enseInOut 动画，但是你可以设置自定义的动画的属性。

有时候我们有一个解决方案 很多 views 依赖某些状态，并且我们想要animate所有相关的  view 一起改变。这种场景中，我们有 animatable bindings.

```swift
struct ContentView : View {
    @State private var isButtonVisible = true

    var body: some View {
        VStack {
            Toggle(isOn: $isButtonVisible.animation()) {
                Text("Show/Hide button")
            }

            if isButtonVisible {
                Button(action: {}) {
                    Text("Hidden Button")
                }
            }
        }
    }
}
```

As you can see, we can easily convert our binding into animatable binding by calling animation method on it. This method wraps every change of binding value into an animation block. You can pass animation settings as parameters to this method. More about bindings you can read in my [previous post](https://swiftwithmajid.com/2019/06/12/understanding-property-wrappers-in-swiftui).

#### Transitions

As I said before, SwiftUI uses fade in and fade out transition by default, but we can apply any other transition we want. Let’s replace fading with moving.

```swift
struct ContentView : View {
    @State private var isButtonVisible = true

    var body: some View {
        VStack {
            Toggle(isOn: $isButtonVisible.animation()) {
                Text("Show/Hide button")
            }

            if isButtonVisible {
                Button(action: {}) {
                    Text("Hidden Button")
                }.transition(.move(edge: .trailing))
            }
        }
    }
}
```

In the example above, we attach transition modifier to the view. SwiftUI has a bunch of ready to use transitions like *move*, *slide*, *scale*, *offset*, *opacity*, etc. We can combine them into a single transition. Let’s take a look at the example.

```swift
extension AnyTransition {
    static func moveAndScale(edge: Edge) -> AnyTransition {
        AnyTransition.move(edge: edge).combined(with: .scale())
    }
}

struct ContentView : View {
    @State private var isButtonVisible = true

    var body: some View {
        VStack {
            Toggle(isOn: $isButtonVisible.animation()) {
                Text("Show/Hide button")
            }

            if isButtonVisible {
                Button(action: {}) {
                    Text("Hidden Button")
                }.transition(.moveAndScale(edge: .trailing))
            }
        }
    }
}
```

We created a *moveAndScale* transition, which is basically a combination of move and scale transitions. SwiftUI applies the current transition symmetrically according to timing or spring values which you pass into the animation method.

SwiftUI provides a way of building asymmetric transitions also. Assume that you need a move transition on insertion and a fade transition on removal. For those cases, we have an *asymmetric* method on *AnyTransition* struct, which we can use to build asymmetric transitions.

```swift
extension AnyTransition {
    static func moveOrFade(edge: Edge) -> AnyTransition {
        AnyTransition.asymmetric(
            insertion: .move(edge: edge),
            removal: .opacity
        )
    }
}

struct ContentView : View {
    @State private var isButtonVisible = true

    var body: some View {
        VStack {
            Toggle(isOn: $isButtonVisible.animation()) {
                Text("Show/Hide button")
            }

            if isButtonVisible {
                Button(action: {}) {
                    Text("Hidden Button")
                }.transition(.moveOrFade(edge: .trailing))
            }
        }
    }
}
```

As you can see, we use *asymmetric* method to pass two transitions, the first one for insertion and another one for removal. We can pass here combined transition which we created.

#### Conclusion

Today we discussed multiple ways of animating changes in SwiftUI. It totally depends on you and on use-case which way you have to choose. By spending more and more time with SwiftUI, I understand that it is already a compelling framework. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!