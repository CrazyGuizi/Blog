# 布局
在Flutter中，布局的核心是Widget，可以说在Flutter中绝大部分东西都是Widget，比如图片、图标和文字等。有些看不见元素的如列（column）、行（row）、表格（grids）等也是Widget。

**Container**部件可以让你定制子部件，当你需要使用padding、margins、borders或者背景颜色的时候可以采用Container。

创建简单布局的步骤：

1. 选择合适的布局部件 （<a href="https://flutter.io/widgets/">Flutter布局</a>）
2. 创建一个部件用来放置可见的对象
3. 在布局部件中添加可见部件
所有的布局部件都有一个child属性或者children属性。前者可用于放置一个孩子，后者可用于放置一个列表形式的孩子。

4. 将布局部件添加到页面中
对于Flutter自己和大多数的部件来说都有一个build()方法。将声明的部件放置在app的build()方法中，设备将会显示这些部件。

可以使用mainAxisAlignment和crossAxisAlignment属性控制行或列的子部件的对齐方式。

android中的View可对应为Flutter中的Widget。在android中可以直接对view进行改变从而更新视图，但Flutter中的widget是不可变的，所以需要使用状态。Flutter中有StatelessWidget和StatefulWidget两种。它们每一帧都会重新构建，不同之处在于Stateful有一个State对象，可跨帧存储状态数据并恢复。

android中可以利用xml文件编写布局，而Flutter是利用widget树进行编写的。android中可以通过addChild或removeChild进行子View的添加或删除，Flutter中可以通过一个方法返回相应的widget对象控制view的添加或删除。

android中自定义View需要继承View重写方法，在Flutter中通过组合相应的Widget实现自定义View。

# 管理状态
常见方法：

1. widget管理自己的State
2. 父widget管理widget状态
3. 混搭管理（父widget和widget自身都管理状态）

**原则**：

- 如果状态是用户数据，如复选框的选中状态、滑块的位置，则该状态最好由父管理。
- 如果所讨论的状态是有关界面外观效果的，例如动画，那么状态最好由widget本身来管理
- 如果有疑问，首选是在父widget中管理状态

# 切换屏幕

android中利用Intent进行activity的跳转和调用外部组件，Flutter切换屏幕是通过访问路由以绘制新的Widget。管理多个屏幕需要两个核心类：**Route**和**Navigator**。Route是应用程序的“页面”（可认为是android中的activity），Navigitor是管理Route的widget，它可通过push和pop route实现页面的切换。

# 线程和同步

Dart是单线程执行模型，支持solates（在另一个线程上运行Dart代码的方式）、事件循环和异步编程。Flutter中在UI线程中运行网络请求并不会导致UI线程挂起。

要运行异步代码可以将其声明为一异步函数：

~~~dart
	funName() async {
    	... = await ...
    }
~~~

```await```等待耗时操作的结果。
