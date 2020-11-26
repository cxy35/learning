---
title: 不要再自己写 Java 工具类了，这些开源的不香吗？
date: 2020-05-31 12:46:35
categories: 工具
tags: [工具]
toc: true
---
本文收集各种 Java 常用工具类，包括字符串、日期、集合/数组、IO、计时等。
<!-- more -->

![](https://oscimg.oschina.net/oscnet/up-dd979dd54e1088eead9eceec461db85dac4.png)

## 1 字符串

### 1.1 StringUtils

Maven 依赖信息：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.10</version>
</dependency>
```

- 判断字符串是否为空

```java
if (StringUtils.isEmpty(str)) {

}

// 如果字符串都是空格的话
StringUtils.isBlank(" ")       = true;
StringUtils.isEmpty(" ")       = false； 
```

判断字符串是否为空，使用频率非常高，这里大家可以使用 IDEA Prefix 的功能，输入直接生成判空语句。

![](https://oscimg.oschina.net/oscnet/up-45a6dceb5fb04a1c54f4007bf25a6c9e7f3.png)

- 字符串固定长度

```java
// 字符串固定长度 8 位，若不足，往左补 0
StringUtils.leftPad("test", 8, "0");
// 字符串固定长度 8 位，若不足，往右补 0
StringUtils.rightPad("test", 8, "0");
```

- 字符串关键字替换

```java
// 默认替换所有关键字
StringUtils.replace("aba", "a", "z")   = "zbz";
// 替换关键字，仅替换一次
StringUtils.replaceOnce("aba", "a", "z")   = "zba";
// 使用正则表达式替换
StringUtils.replacePattern("ABCabc123", "[^A-Z0-9]+", "")   = "ABC123";

// ......
```

- 字符串拼接

```java
StringUtils.join(["a", "b", "c"], ",")    = "a,b,c"
```

StringUtils 只能传入数组拼接字符串，不支持集合拼接，推荐使用 Guava 中的 Joiner ，具体使用见下文。

- 字符串拆分

```java
// 返回数组
StringUtils.split("a..b.c", '.')   = ["a", "b", "c"]
StringUtils.splitByWholeSeparatorPreserveAllTokens("a..b.c", ".") = ["a","", "b", "c"]
```

### 1.2 Joiner/Splitter

Maven 依赖信息：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>29.0-jre</version>
</dependency>
```

- 字符串拼接

```java
String[] array = new String[]{"test", "1234", "5678"};
List<String> list = new ArrayList<>();
list.add("test");
list.add("1234");
list.add("5678");
StringUtils.join(array, ",");

// 逗号分隔符，跳过 null
Joiner joiner = Joiner.on(",").skipNulls();
joiner.join(array);
joiner.join(list);
```

- 字符串拆分

```java
// 返回 List 集合
Splitter splitter = Splitter.on(",");
// 结果：[ab, , b, c]
splitter.splitToList("ab,,b,c");
// 忽略空字符串，输出结果 [ab, b, c]
splitter.omitEmptyStrings().splitToList("ab,,b,c")
```

## 2 日期

### 2.1 DateUtils/DateFormatUtils

Maven 依赖信息：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.10</version>
</dependency>
```

JDK8 之前，Java 只提供一个 `Date` 类，平常我们需要将 `Date` 按照一定格式转化成字符串，我们需要使用 `SimpleDateFormat` 。

```java
SimpleDateFormat simpleDateFormat=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
// Date 转 字符串
simpleDateFormat.format(new Date());
// 字符串 转 Date
simpleDateFormat.parse("2020-05-07 22:00:00");
```

代码虽然简单，但是这里需要注意 `SimpleDateFormat` ，不是线程安全的，多线程环境一定要注意使用安全。推荐使用  `commons-lang3` 下的时间工具类 `DateUtils/DateFormatUtils` 来解决 Date 与字符串转化问题。。

```java
// Date 转化为字符串
DateFormatUtils.format(new Date(),"yyyy-MM-dd HH:mm:ss");
// 字符串 转 Date
DateUtils.parseDate("2020-05-07 22:00:00","yyyy-MM-dd HH:mm:ss");
```

除了格式转化之外，DateUtils 还提供时间计算的相关功能。

```java
Date now = new Date();
// Date 加 1 天
Date addDays = DateUtils.addDays(now, 1);
// Date 加 33 分钟
Date addMinutes = DateUtils.addMinutes(now, 33);
// Date 减去 233 秒
Date addSeconds = DateUtils.addSeconds(now, -233);
// 判断是否 Wie 同一天
boolean sameDay = DateUtils.isSameDay(addDays, addMinutes);
// 过滤时分秒,若 now 为 2020-05-07 22:13:00 调用 truncate 方法以后
// 返回时间为 2020-05-07 00:00:00
Date truncate = DateUtils.truncate(now, Calendar.DATE);
```

### 2.2 JDK8 时间类

JDK8 之后，Java 将日期与时间分为 `LocalDate` 和 `LocalTime` ，功能定义更加清晰，当然其也提供一个 `LocalDateTime` ，包含日期与时间。这些类相对于 Date 类优点在于，这些类与 `String` 类一样都是不变类型，不但线程安全，而且不能修改。

> ps：仔细对比 mysql 时间日期类型 `DATE/TIME/DATETIME` ，有没有感觉差不多

现在 mybatis 等 ORM 框架已经支持 `LocalDate` 与 JDBC 时间类型转化，所以大家可以直接将时间字段实际类型定义为 JDK8 时间类型，然后再进行相关转化。

如果依然使用的是 Date 类型，如果需要使用新的时间类型，我们需要进行相关转化。两者之间进行转化， 稍微复杂一点，我们需要显示指定当前时区。

```java
Date now = new Date();
// Date-----> LocalDateTime 这里指定使用当前系统默认时区
LocalDateTime localDateTime = now.toInstant().atZone(ZoneId.systemDefault()).toLocalDateTime();
// LocalDateTime------> Date 这里指定使用当前系统默认时区
Date date = Date.from(localDateTime.atZone(ZoneId.systemDefault()).toInstant());
```

接下来我们使用 `LocalDateTime` 进行字符串格式化。

```java
// 按照 yyyy-MM-dd HH:mm:ss 转化时间
LocalDateTime dateTime = LocalDateTime.parse("2020-05-07 22:34:00", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
// 将 LocalDateTime 格式化字符串
String format = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss").format(dateTime);
```

另外我们使用 `LocalDateTime` 获取当前时间年份，月份特别简单：

```java
LocalDateTime now = LocalDateTime.now();
// 年
int year = now.getYear();
// 月
int month = now.getMonthValue();
// 日
int day = now.getDayOfMonth();
```

最后我们还可以使用 `LocalDateTime` 进行日期加减，获取下一天的时间：

```java
LocalDateTime now = LocalDateTime.now();
// 当前时间加一天
LocalDateTime plusDays = now.plusDays(1l);
// 当前时间减一个小时
LocalDateTime minusHours = now.minusHours(1l);

// ......
```

## 3 集合/数组

### 3.1 CollectionUtils/MapUtils

Maven 依赖信息：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.4</vesion>
</dependency>
```

我们可以使用 `CollectionUtils/MapUtils` 进行判空判断。

```java
// List/Set 集合判空
if(CollectionUtils.isEmpty(list)){

}
// Map 等集合进行判空
if (MapUtils.isEmpty(map)) {
    
}
```

至于数组判空判断需要使用 `commons-lang` 下的 `ArrayUtils` 进行判断:

```java
// 数组判空
if (ArrayUtils.isEmpty(array)) {
    
}
```

除此之外还有一些别的对于集合增强方法，比如快速将数组加入到现有集合中：

```java
List<String> listA = new ArrayList<>();
listA.add("1");
listA.add("2");
listA.add("3");
String[] arrays = new String[]{"a", "b", "c"};
CollectionUtils.addAll(listA, arrays);
```

### 3.2 Lists/Maps/Sets

Maven 依赖信息：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>29.0-jre</version>
</dependency>
```

使用工具类，我们可以快速创建集合，如：

```java
List<String> list = Lists.newArrayList();
Map<String,String> map = Maps.newHashMap();
Set<String> set = Sets.newHashSet();
```

另外还可以指定集合类的初始化大小。

```java
List<String> list = Lists.newArrayList("a", "b");
List<String> list2 = Lists.newArrayListWithExpectedSize(100);
List<String> list3 = Lists.newArrayListWithCapacity(100);
```

## 4 IO

Maven 依赖信息：

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.6</version>
</dependency>
```

### 4.1 FileUtils（文件操作工具类）

快速实现文件/文件夹拷贝操作，如：`FileUtils.copyDirectory/FileUtils.copyFile`

```java
// 拷贝文件
File fileA = new File("E:\\test\\test.txt");
File fileB = new File("E:\\test1\\test.txt");
FileUtils.copyFile(fileA,fileB);
```

使用 `FileUtils.listFiles` 获取指定文件夹上所有文件

```java
// 按照指定文件后缀如java,txt等去查找指定文件夹的文件
File directory = new File("E:\\test");
FileUtils.listFiles(directory, new String[]{"txt"}, false);
```

使用 `FileUtils.readLines` 读取该文件所有行。

```java
// 读取指定文件所有行 不需要使用 while 循环读取流了
List<String> lines = FileUtils.readLines(fileA)
```

有读就存在写，可以使用 `FileUtils.writeLines`，直接将集合中数据，一行行写入文本。

```java
// 可以一行行写入文本
List<String> lines = new ArrayList<>();
.....
FileUtils.writeLines(lines)
```

### 4.2 IOUtils（I/O 操作相关工具类）

`FileUtils` 主要针对相关文件操作，`IOUtils` 更加针对底层 I/O，可以快速读取 `InputStream`。实际上 `FileUtils` 底层操作依赖就是 `IOUtils` 。

`IOUtils` 可以适用于一个比较实用的场景，比如支付场景下，HTTP 异步通知场景（从 Servlet 获取异步通知内容）。如果我们使用 JDK 原生方法写:

```java
byte[] b = null;
ByteArrayOutputStream baos = null;
String respMsg = null;
try {
    byte[] buffer = new byte[1024];
    baos = new ByteArrayOutputStream();
   // 获取输入流
    InputStream in = request.getInputStream();
    for (int len = 0; (len = in.read(buffer)) > 0; ) {
        baos.write(buffer, 0, len);
    }
    b = baos.toByteArray();
    baos.close();
   // 字节数组转化成字符串
    String reqMessage = new String(b, "utf-8");
} catch (IOException e) {
  
} finally {
    if (baos != null) {
        try {
            baos.close();
        } catch (IOException e) {
           
        }
    }
}
```

上面代码说起来还是挺复杂的。不过我们使用 `IOUtils` ，一个方法就可以简单搞定：

```java
// 将输入流信息全部输出到字节数组中
byte[] b = IOUtils.toByteArray(request.getInputStream());
// 或将输入流信息转化为字符串
// String resMsg = IOUtils.toString(request.getInputStream());
```

## 5 计时

### 5.1 Stopwatch

Maven 依赖信息：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>29.0-jre</version>
</dependency>
```

编程中有时需要统计代码的的执行耗时，当然执行代码非常简单，结束时间与开始时间相减即可。

```java
long start = System.currentTimeMillis();   //获取开始时间

//其他代码
//...
long end = System.currentTimeMillis(); //获取结束时间

System.out.println("程序运行时间： " + (end - start) + "ms");
```

虽然代码很简单，但是非常不灵活，默认情况我们只能获取 ms 单位，如果需要转换为秒，分钟，就需要另外再计算。

这里我们介绍 `Guava` 中的 `Stopwatch` 计时工具类，借助他统计程序执行时间，使用方式非常灵活。

> `commons-lang3` 与 `spring-core` 也有这个工具类，使用方式大同小异，大家根据情况选择。

```java
// 创建之后立刻计时，若想主动开始计时
Stopwatch stopwatch = Stopwatch.createStarted();
// 创建计时器，但是需要主动调用 start 方法开始计时
// Stopwatch stopwatch = Stopwatch.createUnstarted();
// stopWatch.start();
// 模拟其他代码耗时
TimeUnit.SECONDS.sleep(2L);

// 当前已经消耗的时间
System.out.println(stopwatch.elapsed(TimeUnit.SECONDS));

TimeUnit.SECONDS.sleep(2L);

// 停止计时 未开始的计时器调用 stop 将会抛错 IllegalStateException
stopwatch.stop();
// 再次统计总耗时
System.out.println(stopwatch.elapsed(TimeUnit.SECONDS));
// 重新开始，将会在原来时间基础计算，若想重新从 0开始计算，需要调用 stopwatch.reset()
stopwatch.start();
TimeUnit.SECONDS.sleep(2L);
System.out.println(stopwatch.elapsed(TimeUnit.SECONDS));
```

输出结果为：

```
2
4
6
```


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)