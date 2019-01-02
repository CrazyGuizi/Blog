
## Activty的生命周期


<img alt="Activity生命周期" src="https://developer.android.com/images/activity_lifecycle.png"/>

- onCreate():当Activity第一次创建的时候调用。在这个方法中可以创建视图、绑定数据等。后面都是接着onStart()

- onRestart():当Activity停止的再重新开始时调用。


- onStart():当用户可见时调用。如果Activity来到前台则调用onResum()，如果被隐藏则调用onStop()。


- onResume():当可以和用户交互时调用，此时Activity处于栈顶。


- onPause():当系统开始恢复之前的Activity时调用。适合保存一些不安全状态数据，停止动画。所做的事需要快速，因为新的Activity在这个方法返回之前不会处于resume。如果Activity重回前台时调用onResume（）,如果变为不可见则调用onStop()。


- onStop():对用户不再可见时调用，因为另一个Activity已经处于前台并且覆盖了它。如果重回可交互状态则调用onRestart()，销毁的话则调用onDestroy()。


- onDestroy():当Activity被销毁前的最后一个回调。当调用finish()或者系统回收这个Activity实例内存 时调用该方法。

## Fragment的生命周期

- onAttach()：与activity绑定的时候
- onCreate():做一些初始化工作
- onCreateView()：创建于Fragment相关联的视图
- onActivityCreate():让Fragment知道Activity已经完成onCreate()
- onViewStateRestored():所有的视图结构已经恢复
- onStart():可见
- onResume():可以互动
- onPause():当activity调用pause()或者Fragment的状态被修改时不再与用户互动
- onStop():不可见
- onDestroyView():可以让Fragment释放资源
- onDestroy():fragment的最后清理
- onDetach():与activity没有关联


<img alt="Fragment生命周期" src="http://img.my.csdn.net/uploads/201211/29/1354170699_6619.png"/>