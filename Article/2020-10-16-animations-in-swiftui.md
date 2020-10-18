# Animations in SwiftUI

原文：[Animations in SwiftUI](https://swiftwithmajid.com/2019/06/26/animations-in-swiftui/)    作者：[[Swift with Majid](https://swiftwithmajid.com/)]

翻译：阳阳

Swift 为我们带来了声明式这样直接的方式来创建用户界面。我们有 List 和 Form 等组件还有各种绑定值(Bindings)。这些都让SwiftUI如此易用同时十分强大。不过今天我们准备聊一聊SwiftUI中的另一特性，动画。

### 动画

在SwiftUI中你可以通过把值的改变包装进  `withAnimation`  块中来用流畅的动画展现。默认的，SwiftUI会用淡入 (fade in) 和淡出 (fade out) 的方式来展现变化。让我们来看一个小例子。

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

在这个例子中，我们把状态变化包装进 `withAnimation`  块中就实现了不错的淡入效果。你可以通时间和spring参数控制动画效果。也可以在需要做动画的 `view` 后面使用 `.animation` 修饰器。

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

这上面的示例代码中，我们用过简单的添加 `.animation` 修饰器实现了和之前同样的效果。这里使用淡入淡出动画，你也可以通过参数自定义的动画的属性。

有时候多个 `views` 需要依赖一些状态，并且我们想要同时用动画表现所有相关  `view` 的状态变化。在这种场景中，我们可以用可动画的 (animatable) 绑定值 (bindings)。

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

如你所见，我们可以容易的转换我们 的 binding 动画的binding 通过调用动画方法在它上面。这个方法包装了所有变化绑定至一个 animation 块中。你可以传递动画的设置以方法参数方式。更多关于 bindings 你可以看我[之前的文章](https://swiftwithmajid.com/2019/06/12/understanding-property-wrappers-in-swiftui)。

#### 过渡动画

As I said before, SwiftUI uses fade in and fade out transition by default, but we can apply any other transition we want. Let’s replace fading with moving.

像我上面说的，SwiftUI 默认使用的淡入淡出效果，不过我们也可以使用我们想要的其他动画效果。下面让我们把淡入淡出效果换成移动效果。

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

在上面的例子中，我们在 view 的后面添加了 `.transition` 修饰器。SwiftUI 有一组可以的过渡效果比如 move, slide, scale, offset, opacity 等等。我们可以把它们结合成一个新的过渡效果。让我们来看一个例子。

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

我们创建了一个 *moveAndScale* 过渡动画，是 move 过渡和 scale 过渡的简单结合。SwiftUI 应用这个过渡动画对称根据时间和spring 值 你传到这个动画方法里的。

SwiftUI provides a way of building asymmetric transitions also. Assume that you need a move transition on insertion and a fade transition on removal. For those cases, we have an *asymmetric* method on *AnyTransition* struct, which we can use to build asymmetric transitions.

SwiftUI 提供了一种方式创建不对称的过渡。假设你需要一个move过渡动画在插入和一个淡出动画在移除的时候。我们有 `.asymmetric`方法 `AnyTransition` 结构体，我们可以创建不对称过渡效果。

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

在这个例子里我们使用AnyTransition的  `.asymmetric` 方法并传递了一个 move 过渡动画和一个 opacity 过渡动画作为参数，move 用来显示插入效果，opacity 用来显示移除效果。

#### 结论

今天我们讨论了SwiftUI 中的多种动画方式。如何使用完全取决于你的喜好和使用场景。通过在 SwiftUI上花费越来越多的时间，我发现 SwiftUI 现在已经是一个引人入胜的框架了。如果对这篇文章有疑问可以到 [Twitter](https://twitter.com/mecid) 上关注我并向我提问。感谢阅读，下周见！