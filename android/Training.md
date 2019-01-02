# 谷歌android官方教程

## 添加actionBar按钮

1. 创建相应的menu资源，在res下创建menu目录，然后添加一个`xml`文件
2. 在activity中实现onCreateOptionMenu回调方法，获取Menu对象
3. 在onOptionItemSelected方法中添加响应事件

menu的`xml`文件格式大致上如下：
~~~xml
<item
	android:id="按钮的id"
    android:icon="按钮的图标"
    android:title="按钮的标题"
    app:showAsAction="显示的方式"/>
~~~
显示的方式有5种：

- always 总是显示在actionBar上
- collapseActionView 将操作的视窗折叠到一个按钮中
- ifRoom 如果空间足够就显示
- never 从不显示
- withText 图标和标题文字一起显示

~~~java
	public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.nav, menu);
        return super.onCreateOptionsMenu(menu);
    }
~~~

~~~java
	    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.nav_title:
                Toast.makeText(this, "测试", Toast.LENGTH_SHORT).show();
        }
        return super.onOptionsItemSelected(item);
    }
~~~

如果需要返回上一级按钮则可以：
`getSupportActionBar.setDisplayHomeAsUpEnabled(true)`

## 自定义主题

1. 在res/values目录下创建一个themes.xml文件
2. 在这个xml文件中实现自定义主题
3. 在manifest.xml文件中设置theme

xml文件的主要结构为：

~~~xml
<resources>
    <style name="你的样式名称"
           parent="需要继承的样式">
        <item name="需要改写的属性">自定义的属性值</item>
    </style>
</resources>
~~~

如果需要actionBar覆盖层叠的话，则可以同样重写样式，其属性名为`windowActionBarOverlay`，其值为`true`

## 兼容不同设备

- 适配语言

1. 在res下创建多个values文件目录，例如`values/strings.xml,values-es/strings.xml,values-fr/strings/xml`。values目录以连接符“-”加上ISO的国家代码结尾。
2. 使用字符资源可以通过`getResource.getString(R.strings.定义的字符名称)`
3. 字符在定义的时候可以使用占位符，比如my %1$s is %2$d，使用的时候可以使用getResource.getString()获取这个字符，然后String.format(字符串对象,具体内容...)

- 适配屏幕
- 适配系统版本