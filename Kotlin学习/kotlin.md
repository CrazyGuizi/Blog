常量：val

变量：var

定义变量：

var 变量名: 变量类型(变量类型首字母大写)

默认整数类型是Int，小数类型是Double

变量占位符：${}

类型转换函数：toInt()等

位运算：

一左移二 = 1 shl 2

一右移二 = 1 shr 2

一无符号右移二 = 1 ushr 2

与：and 或：or 异或：xor 取反：.inv()

元组：Pair(x,y) Triple(x,y,z)

可空类型：在类型后面加上一个？，如Int？

可用下划线分隔千分位或万分位等，如100_0000

重命名导入：name as anotherName

Kotlin中所有的异常都是非必检的，处理异常的方法和java一样

检查引用是否相等用"==="或"!=="操作符，判断结构相等则用"=="和"!="操作符

表达式可以不止一行，可以是块，但是最后一行必须是表达式，这个表达式就是整个表达式的值。

区间：..，如1..9。它是一个全闭区间

when是switch的加强版：

when(value) {

1 -> 语句

2-> 语句

else -> 语句

}

如果when没有参数，则类似多分支的if-else语句

函数返回：return会从最近的嵌套函数或匿名函数返回。

判断字符串内容是否相等：可用"=="或者".contentEquals()"

变长参数的定义借助vararg修饰符，在调用函数时，传入的参数前面加上"*"号，"*"加在变量名前表示展开。

Kotlin中的函数按照作用范围可分为成员函数、本地函数和顶层函数。成员函数即java中的方法。Kotlin允许把函数声明在其他函数内部，它们被称为本地函数或者嵌套函数。顶层函数不属于任何源码文件的小集团(class，对象，interface)，它直接定义在源文件中。

操作符重载：利用operator关键字，加在函数前面。圆括号可以通过重载invoke函数作为操作符。比较操作符可以通过重载compareTo函数实现。

数组：Array<类型> 或 arrayof()。大小了调用.count()或.size

可变列表：MutableList<类型>或mutableListOf()。添加.add()，移除remove()，移除指定位置元素removeAt()。

字典：mapOf<Key,Value>(Pair(key,value),...)。获取某个键所对应的值可以用get，getOrDefault或者下标法。所有的键.keys，所有的值.values，所有的条目.entries。

可变字典：mutableMapOf<Key,Value>(Pair(key,value))。添加或更新put或下标法

父类前面加上open，表示可以被继承，Kotlin不允许多继承。如果需要继承一个类，则在它后面加上冒号和需要继承的类名。如果父类有主构造函数，那么子类需要负责初始化其参数。如果父类没有主构造函数而只有次构造函数，那么子类需要借助super来初始化。

Kotlin的方法如果不加open，则默认是final类型的，所以子类需要继承或者覆盖则需要在相应的方法前面加上open。注意，val属性只有getter，var属性既有getter也有setter，所以子类覆盖其属性的时候只能扩展不能减少，例如用val覆盖var就是不允许的。

声明抽象类和抽象方法则在前面加上abstract。声明一个接口用 interface，实现也是用冒号。如果需要实现多个接口与，接口之间用逗号隔开

子类实现的接口中有相同名字的方法，则可利用super<接口名>.方法

open和final控制是否可被继承或覆盖

![修饰符](.\修饰符.png)