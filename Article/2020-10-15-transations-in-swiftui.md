# Transactions in SwiftUI

来源:  [Swift with Majid](https://swiftwithmajid.com/2020/10/07/transactions-in-swiftui/)   翻译: 阳阳

动画在SwiftUI中扮演了直观中演的角色。我们看到有很多在SwiftUI中可以实现复杂动画的例子。创建流畅动画的引导在SwiftUI中只有一步：修改你的State，然后SwiftUI会自动动画展示改变在你的Views里。今天我打算聊聊SwiftUI中一个隐藏的亮点-过度动画。

### 基础

过渡是上下文xian

```swift
import SwiftUI

struct AnimatedView: View {
    let scale: CGFloat

    var body: some View {
        Circle()
            .fill(Color.accentColor)
            .scaleEffect(scale)
            .animation(.spring())
    }
}
```

如我们所知，动画 modifier 应用动画对所有子 views 应用该动画效果。苹果认为我们使用这个modifier在叶子view而不是container view。这种方式允许我们指定动画为我们需要的views。