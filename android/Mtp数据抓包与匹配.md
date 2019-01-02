# 说明

该文档总共分为两大部分，分别对应于MTP文档数据抓取和数据匹配。以下分别介绍这两大部分。

# 第一部分：MTP数据抓取

该项目主要是通过mtp后台接口直接获取MTP文档所对应的数据。所以这个项目主要做的工作是封装好抓取mtp文档数据的接口，将抓取到的数据整理成文档保存。

## 开发环境与配置

- IntelliJ IDEA 2018.2.3 (Ultimate Edition)
- JAVA 8

## 用到的第三方库

- HttpClient
- fastJson

## 项目结构

![MTP数据抓取项目结构](WEBRESOURCEa548cec6390fc8c7595e13924a827cfe)

- mtp 存放文档
- async 线程池
- beans 为了将mtp抓取到的数据反序列而准备的对象类
- crawler 该项目的核心之一，存放着抓取mtp文档各个接口的类，抓取数据以及生成文档的类
- net 网络库
- utils 一些工具类，比距文件操作等


## 重点介绍

![主要的类](WEBRESOURCEbdfee17ae822e1f7c3211a65668fb2a7)

上面的类图展示的类是主要的类，下面一一介绍。

### 网络库

#### BaseRequest

公共的网络请求类，有一个`mUrl`成员变量。主要方法如下：

~~~java

    // 设置请求头
    public abstract BaseRequest setHeader(Map<String, String> headers);

    // 设置请求头
    public abstract BaseRequest setHeader(String[] headers);

    // 设置请求参数
    public abstract BaseRequest setParams(Map<String, String> params);

    // 设置请求参数
    public abstract BaseRequest setParams(String[] params);
~~~

#### HttpClientRequest

构建一个HttpClient的请求，可以是get也可以是post，请求参数格式默认是json形式。

如果需要构建一个请求可以按照下面的形式：

~~~java
HttpPost = (HttpPost) new HttpClientRequest(url).post()
.setHeader(headers)
.setParams(params)
.create();
~~~

#### HttpURLRequest

构建一个`HttpURLConnection`的请求，其创建形式与`HttpClientRequest`差不多，这里不再赘述，调用`connection()`方法可以得到一个`HttpURLConnection`对象。


#### BaseResponse

网络请求相应处理的基类，若是成功的话会回调`onSuccess(T t)`,否则回调`onFailed(BaseException e)`。

~~~java

    public abstract class BaseResponse<T> {

    public abstract void onSuccess(T t);

    public abstract void onFailed(BaseException e);
}
~~~

### BaseException、NetException

针对发生网络请求失败或者响应码不是1000的情况而定义的类。失败处理时回调中会传入这些异常的实例，通过异常可以查看网络请求失败的错误信息和响应码。


#### HttpClientUtil、HttpURLConnectionUtil

这两个类都是发起网络请求的核心类，负责处理网络请求的全部流程。这里介绍几个比较重要的方法。根据同步与异步、get和post可以分为这么几类请求：

- 同步`get`、同步`post`
- 异步`get`、异步`post`

同步和异步请求都是同名的，区别在于参数。具有`BaseResponse`参数的方法看做是异步请求（会回调），而不带这个参数的是同步请求。

下面以`post`请求作为一个示例

~~~java

// 异步请求
HttpClientUtil.getInstance()
.post(Const.URL_CATS, sHeaders, params, new SuccessResponse<CatsBeans>() {
            @Override
            public void onSuccess(CatsBeans catsBeans) {
                
            }
        });
 
// 同步请求       
CatsBeans cats = HttpClientUtil.getInstance()
.post(Const.URL_CATS, sHeaders, params, CatsBeans.class);
~~~

使用`HttpURLConnectionUtil`也是一样的形式。

类中方法说明：

- buildHttpClientRequest、buildHttpURLRequest 生成一个请求
- doAsyncProcess 处理异步请求
- doSyncProcess 处理同步请求
- doSuccess 请求成功并响应码为1000时调用，进行反序列化操作
- doFailed 请求失败是调用

### crawler包

#### CrawlerUtil

针对平安付文档中心所做的一个抓取数据接口中心，管理着抓取各个接口的方法

封装了抓取数据的网络请求，采用这样的模式可以抓取的不单单是mtp，其他文档也可以抓取
如果抓取数据的接口参数改变也可以在此类中修改，需要添加接口的也可以在此类添加

该方法的网络请求全是基于同步的，如果需要异步操作可以使用 `main.async.ThreadManager`这个类。

方法说明：

限定符和类型 |	方法和说明
--|--
static Teams	 | getTeams()</br>获取平安付文档中心的全部工程
static Versions	 | getVersions(java.lang.String prjName)</br>返回某个工程的全部版本
static CatsBeans |	getCatsBeans(java.lang.String prjName, java.lang.String ver)</br>获取左边列表全部数据
static ViewBeans | getViewBeans(java.lang.String prjId, java.lang.String cat)</br>获取右边列表全部数据
static DocBeans	 | getDocBeans(java.lang.String prjId, java.lang.String docId)</br>获取接口详情


- getVersions

public static Versions getVersions(java.lang.String prjName)

参数:
prjName  这个工程的名字，比如mtp,avms


- getCatsBeans
public static CatsBeans getCatsBeans(java.lang.String prjName,
                                     java.lang.String ver)

参数:prjName - 工程名字，比如mtp

ver - 版本，比如20181018_req128823


- getViewBeans
public static ViewBeans getViewBeans(java.lang.String prjId,
                                     java.lang.String cat)

参数:

prjId - 工程Id，比如mtp#20181018_req128823

cat - BT信贷


- getDocBeans
public static DocBeans getDocBeans(java.lang.String prjId,
                                   java.lang.String docId)

参数:

prjId - 工程Id，比如mtp#20181018_req128823

docId - com.pinganfu.mtp.web.controller.p2.CESRepayController#getLoanDetailInfo


#### GetMtp

用于处理MTP数据
<p>
本类提供了获取全部版本mtp数据，根据版本获取mtp数据，获取AVMS数据的接口。
支持修改保存的位置以及合成文件的名字
<p>
注意：不能同时调用多个方法，因为可能在写入文件时因为多线程原因导致文件写入失败等问题

重要方法说明：
这些方法都是利用了`CrawlerUtil`所提供的接口实现的

- GetMtp()  构造函数，可以修改数据文档保存的位置和文档名字


- getMtp() 抓取mtp的全部版本数据，每个版本写一个文件夹


- getAVMS 获取AVMS的全部版本数据，全部版本数据整合在一起的一个HashMap


- getMtpAllToMerge() 将mtp、avms所有版本合为一个版本。其实现的思路是利用一个`HashMap`存放每个版本的合成数据，先从低版本放起，最后所得的即为最新且最全的数据。


- getMtpByVersion 根据版本生成一个合成文件，并且每一个版本的合成文件名字都是一样的（merge.json，支持修改



---

# 第二部分：数据匹配

该项目主要是利用mtp后台接口获取的数据与客户端所传入的数据做匹配。

## 开发环境与配置

- Android Studio 3.1.4
- 该版本自带的SDK

## 用到的第三方库

- fastJson

## 重点介绍

该项目主要的类是`MatchUtil`，对外暴露的接口为`isMatch()`，这个方法可以匹配入参也可以匹配出参，由该方法的参数`matchReq`决定（`matchReq`是个布尔值，真则匹配出参，假则匹配出参）

匹配之前要先生成一个数据结构用以表示后台mtp数据。这个类如下：

~~~java

    /**
     * 包装mtp数据的实体类
     */
    class Match {

    public String url; // 请求链接

    // 入参
    public List<Params> reqParams;

    // 出参
    public List<Params> resParams;

    public Match(String url, List<Params> reqParams, List<Params> resParams) {
        this.url = url;
        this.reqParams = reqParams;
        this.resParams = resParams;
    }

    public static class Params {
        public String name; // 参数名字

        public String type; // 参数类型

        public boolean require; // 是否必需

        // 用后台的数据表示则为：该字段如果是一个List或者自定义的类
        // 并且他有子参数的话，那么这个mChild就是这些子参数
        public List<Params> mChild;

        public Params(String name, String type, boolean require) {
            this.name = name;
            this.type = type;
            this.require = require;
        }
    }
}
~~~

`MatchUtil`类的`init()`方法负责加载mtp后台数据，并且将数据转化为`Match`类，这个方法返回一个`HashMap`对象，键为一个`url`，值为这个`url`所对应的入参和出参（即一个`Match`对象，他封装了这两者）

匹配过程：

调用`isMatch(String url, Object params, boolean matchReq)`方法，`params`表示客户端传出的带匹配的对象，如果是入参则是一个`Map`对象，出参则是一个自定义对象实例。`matchReq`用来表示此刻匹配的是入参还是出参。

接着会调用`checkParams()`方法，该方法的详细说明如下：

~~~java
/**
     * 检验所给的参数和后台的数据在类型、是否必须上是否一致
     * <p>
     * 遍历后台抓取的全部参数，与传入的参数一一匹配
     * <p>
     * 1. 第一个条件处理说明
     * 待检验数据是一个map对象，比如客户端传入的是一个map，
     * 它有<“name”,"LDG">这样的值，我们需要做的是用后台数据和这个name
     * 相匹配，首先查看名字是否相同，如果相同则看类型是否匹配（这里是String类型）
     * <p>
     * 2. 第二个条件处理说明
     * 假如传入的是一个List对象，一般来说这个list中存放的是一个封装好的类，
     * 我们需要根据这个类拿出它的属性字段，这些属性字段理论上来说是和后台
     * 接口所规定的参数是一样的，即名称一样，类型也一样
     * <p>
     * 3. 第三个条件处理说明
     * 假如传入的直接是一个对象，比如用户自定义的一个类对象，和2的处理基本一样
     *
     * @param req    包含待匹配参数集合的一个对象
     * @param params 后台参数集合
     */
    private static void checkParams(Object req, List<Match.Params> params) {
    // ……
        
    }
~~~

`checkMapTypeParams（）`方法的处理过程如下：

1. 取出map中该参数的类型，如果为空进一步判断该字段是否为必须，如果必须则设置匹配结果为false，否则继续
2. 将参数类型与后台参数类型进行匹配，类型一样则继续，否则设置匹配结果为false
3. 遍历完所有后台参数，都没有问题则返回true
4. 如果map中取出的值是List对象或者是其他含有子参数的对象，则递归调用checkParams进行处理


`checkClassTypeParams()`方法的处理过程如下：

1. 利用class和根据后台参数的name取出这个类的属性字段
2. 这个字段如果有值，则说明这个类确实有这个字段，否则抛出异常
3. 在异常处理中判断这个字段是否为必须字段，如果不必须，可认为无影响，否则返回匹配结果为false
4. 如果存在这个字段，进一步判断这个字段是否有子参数，如果有则继续调用checkParams进行递归处理

以上为匹配过程的大体流程，其细节部分可以查看源代码，里面应该有些详细的注释o(╯□╰)o。

如果需要将生成的日志文件进行整合则可以使用`MergeLogUtil`类，其默认构造函数`mergeLog`指定了日志存放的默认位置，如果日志文件有改变，可以使用另一个构造函数`mergeLog(String path, String mergeLogName)`进行修正。这个类主要是利用文件的MD5值和一个`HashMap`进行整理，MD5值一样的`map`只做一个出现次数加1的操作，如果不一样则会将MD5值作为键，文件内容作为值存放到`map`中，并且设置这个错误日志出现的次数为1。