---
title: Markdown 语法
date: 2019-10-24 16:06:54
categories: Markdown
tags: [Markdown]
toc: true
thumbnail: /images/markdown/markdown-thumbnail.jpg
---
用 Markdown 写作效率杠杠滴，手把手带你学习 Markdown 语法。
<!-- more -->

## 1 通用语法

### 1.1 标题

**示例：**

```markdown
# 这是一级标题

## 这是二级标题

### 这是三级标题

#### 这是四级标题

##### 这是五级标题

###### 这是六级标题
```

### 1.2 文本样式

**示例/效果：**

```markdown
*这是倾斜的文字*

**这是加粗的文字**

***这是斜体加粗的文字***

~~这是加删除线的文字~~

==这是高亮的文字（Markdown标准中不存在）==

段落: 段落之间空一行

换行符: 一行结束时输入两个空格  
这里换行了
```

### 1.3 引用

**示例：**

```markdown
> 这是引用的内容

>> 这是引用的内容

>>> 这是引用的内容
```

**效果：**

> 这是引用的内容

>> 这是引用的内容

>>> 这是引用的内容

### 1.4 分割线

**示例：**

```bash
# 三个或者三个以上的 - 或者 * 都可以：
---
***
```

**效果：**

---

***

### 1.5 图片

**示例：**

```bash
![这是图片的描述](https://gss2.bdstatic.com/0.jpg "这是图片的 title")
```

**效果：**

![这是图片的描述](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1580794670938&di=b385fa4defa36ef5f1255775a95f8490&imgtype=jpg&src=http%3A%2F%2Fimg2.imgtn.bdimg.com%2Fit%2Fu%3D2372297862%2C56418493%26fm%3D214%26gp%3D0.jpg "这是图片的 title")

### 1.6 超链接

**示例：**

```bash
[简书](https://jianshu.com)
[百度](https://baidu.com "这是超链接的 title")
```

**效果：**

[简书](https://jianshu.com)

[百度](https://baidu.com "这是超链接的 title")

### 1.7 列表

**示例：**

```bash
# 无序列表用 - + * 任何一种都可以：
- 无序列表1
- 无序列表1

+ 无序列表2
+ 无序列表2

* 无序列表3
* 无序列表3

# 有序列表用数字加点：
1. 有序列表
2. 有序列表
  2.1. 列表嵌套
  2.2. 列表嵌套
```

**效果：**

- 无序列表1
- 无序列表1

+ 无序列表2
+ 无序列表2

* 无序列表3
* 无序列表3

1. 有序列表
2. 有序列表  
  2.1. 列表嵌套  
  2.2. 列表嵌套

### 1.8 表格

**示例：**

```bash
|姓名|技能|排行|
|:-|:-:|-:|
|文字默认居左|-两边加:表示文字居中|-右边加:表示文字居右|
|刘备|哭哭哭|大哥|
|关羽|打打打|二哥|
|张飞|骂骂骂|三弟|

第二行分割表头和内容。
- 有一个就行，为了对齐，可多加几个
文字默认居左
-两边加：表示文字居中
-右边加：表示文字居右
```

**效果：**

|姓名|技能|排行|
|:-|:-:|-:|
|文字默认居左|-两边加:表示文字居中|-右边加:表示文字居右|
|刘备|哭哭哭|大哥|
|关羽|打打打|二哥|
|张飞|骂骂骂|三弟|

### 1.9 代码

**示例：**

```bash
# 单行代码：代码之间分别用一个反引号包起来：
`这是单行代码`

# 代码块：代码之间分别用三个反引号包起来，且两边的反引号单独占一行：
# 注：为了防止冲突前后加了a，实际上没有。
a```javaa
// 下面是多行 java 代码
// FileName: HelloWorld.java
public class HelloWorld {
  // Java 入口程序，程序从此入口
  public static void main(String[] args) {
    System.out.println("Hello,World!"); // 向控制台打印一条语句
  }
}
a```a

# 支持以下语言种类：
bash
clojure，cpp，cs，css
dart，dockerfile, diff
erlang
go，gradle，groovy
haskell
java，javascript，json，julia
kotlin
lisp，lua
makefile，markdown，matlab
objectivec
perl，php，python
r，ruby，rust
scala，shell，sql，swift
tex，typescript
verilog，vhdl
xml
yaml
```

**效果：**

`这是单行代码`

```java
// 下面是多行 java 代码
// FileName: HelloWorld.java
public class HelloWorld {
  // Java 入口程序，程序从此入口
  public static void main(String[] args) {
    System.out.println("Hello,World!"); // 向控制台打印一条语句
  }
}
```

## 2 特殊语法

**来源 [https://mdnice.com/](https://mdnice.com/)**

### 2.1 脚注

> 支持平台：微信公众号、知乎。

**示例：**

```bash
# 脚注与链接的区别如下所示：
链接：[文字](链接)
脚注：[正文中显示的脚注文字](脚注中显示的脚注解释 "脚注中显示的脚注名称")

# 脚注的效果拉到文章最下面观看。
```

**效果：**

[正文中显示的脚注文字](https://www.baidu.com "脚注中显示的脚注名称")

### 2.2 代码块

> 支持平台：微信代码主题仅支持微信公众号！其他主题无限制。

如果在一个行内需要引用代码，只要用反引号引起来就好，如下：

Use the `printf()` function.

在需要高亮的代码块的前一行及后一行使用三个反引号，同时**第一行反引号后面表示代码块所使用的语言**，如下：

```java
// FileName: HelloWorld.java
public class HelloWorld {
  // Java 入口程序，程序从此入口
  public static void main(String[] args) {
    System.out.println("Hello,World!"); // 向控制台打印一条语句
  }
}
```

支持以下语言种类：

```
bash
clojure，cpp，cs，css
dart，dockerfile, diff
erlang
go，gradle，groovy
haskell
java，javascript，json，julia
kotlin
lisp，lua
makefile，markdown，matlab
objectivec
perl，php，python
r，ruby，rust
scala，shell，sql，swift
tex，typescript
verilog，vhdl
xml
yaml
```

如果想要更换代码主题，可在上方挑选，不支持代码主题自定义。

其中**微信代码主题与微信官方一致**，有以下注意事项：

- **带行号且不换行，代码大小与官方一致**
- **需要在代码块处标志语言，否则无法高亮**
- **粘贴到公众号后，用鼠标点代码块内外一次，完成高亮**

diff 不能同时和其他语言的高亮同时显示，且需要调整代码主题为微信代码主题以外的代码主题才能看到 diff 效果，使用效果如下:

```diff
+ 新增项
- 删除项
```

**其他主题不带行号，可自定义是否换行，代码大小与当前编辑器一致**

### 2.3 数学公式

> 支持平台：微信公众号、知乎。

行内公式使用方法，比如这个化学公式：$\ce{Hg^2+ ->[I-] HgI2 ->[I-] [Hg^{II}I4]^2-}$

块公式使用方法如下：

$$H(D_2) = -\left(\frac{2}{4}\log_2 \frac{2}{4} + \frac{2}{4}\log_2 \frac{2}{4}\right) = 1$$

矩阵：

$$
  \begin{pmatrix}
  1 & a_1 & a_1^2 & \cdots & a_1^n \\
  1 & a_2 & a_2^2 & \cdots & a_2^n \\
  \vdots & \vdots & \vdots & \ddots & \vdots \\
  1 & a_m & a_m^2 & \cdots & a_m^n \\
  \end{pmatrix}
$$

公式由于微信不支持，目前的解决方案是转成 svg 放到微信中，无需调整，矢量不失真。

目前测试如果公式量过大，在 Chrome 下会存在粘贴后无响应，但是在 Firefox 中始终能够成功。

### 2.4 TOC

> 支持平台：微信公众号、知乎。

**示例：**

```bash
# TOC 全称为 Table of Content，列出全部标题。由于示例标题过多，需要使用将下方代码段去除即可。
# 由于微信只支持到二级列表，本工具仅支持二级标题和三级标题的显示。
[[TOC]]
```

**效果：**

[[TOC]]

### 2.5 注音符号

> 支持平台：微信公众号。

**示例：**

```bash
Markdown Nice 这么好用，简直是{喜大普奔|hē hē hē hē}呀！
```

**效果：**

Markdown Nice 这么好用，简直是{喜大普奔|hē hē hē hē}呀！

### 2.6 横屏滑动幻灯片

> 支持平台：微信公众号。

**示例：**

```bash
# 通过`<![](url),![](url)>`这种语法设置横屏滑动滑动片，具体用法如下：
<![蓝1](https://my-wechat.mdnice.com/mdnice/%E8%93%9D1_20191109174052.jpg),![绿2](https://my-wechat.mdnice.com/mdnice/%E7%BB%BF2_20191109174052.jpg),![红3](https://my-wechat.mdnice.com/mdnice/%E7%BA%A23_20191109174052.jpg)>
```

**效果：**

<![蓝1](https://my-wechat.mdnice.com/mdnice/%E8%93%9D1_20191109174052.jpg),![绿2](https://my-wechat.mdnice.com/mdnice/%E7%BB%BF2_20191109174052.jpg),![红3](https://my-wechat.mdnice.com/mdnice/%E7%BA%A23_20191109174052.jpg)>

## 3 其他语法

**来源 [https://mdnice.com/](https://mdnice.com/)**

### 3.1 HTML

**示例：**

```bash
#支持原生 HTML 语法，请写内联样式，如下：
<span style="display:block;text-align:right;color:orangered;">橙色居右</span>
<span style="display:block;text-align:center;color:orangered;">橙色居中</span>
```

**效果：**

<span style="display:block;text-align:right;color:orangered;">橙色居右</span>
<span style="display:block;text-align:center;color:orangered;">橙色居中</span>

### 3.2 UML

不支持，推荐使用开源工具`https://draw.io/`制作后再导入图片

### 3.3 组件图床

组件目前共支持 3 种图床和 1 种自定义图床，主要特点如下：

| 图床   | 费用     | 有效期 | 失败率 |
| ------ | -------- | ------ | ------ |
| SM.MS  | 免费     | 长期   | 高     |
| 阿里云 | 付费     | 自定义 | 低     |
| 七牛云 | 10G 免费 | 自定义 | 低     |
| 自定义 | 高昂 | 自定义 | 自定义 |

4 个图床的缺点：

| 图床   | 缺点                     |
| ------ | ------------------------ |
| SM.MS  | 失败率高可用性很差       |
| 阿里云 | 配置繁琐，费用昂贵       |
| 七牛云 | 配置繁琐，需购买长期域名 |
| 自定义 | 搭建后台繁琐 |

### 3.4 更多文档

更多文档请参考 [markdown-nice-docs](https://docs.mdnice.com "Markdown Nice 更多文档")


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)