---
description: 本节的内容是选取主要功能进行需求建模和流程分析
---

# 二. 主要功能分析与建模

        Fastjson的主要功能是Java Object和JSON字符串的互相转换，但以下只会选择将JSON对象转换为Java Object这一主要功能进行分析。如果读者感兴趣的是序列化过程，可以参阅雷正宇同学的报告第二章：[fastjson-learning-report](https://842376130.gitbook.io/fastjson-learning-report/di-er-zhang-serializer-de-jie-gou)。

## 0. JSON对象简介

        为此先在此处简要介绍JSON对象，一个JSON对象形如：

{% code-tabs %}
{% code-tabs-item title="JSON对象" %}
```java
{name: "SBZeng", age: 20}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

        它由一对大括号和多个键值对组成，一个键值对由一个域名和对应的值构成，表现为key: value，其中value可以是字符串、数字等。上述的JSON对象实际上表达了一个具有成员变量String name和int age的对象，这两个变量的值分别是"SBZeng"和20。这个简要的介绍已经足够继续我们接下来的旅程。

## 1. 从一个简单示例说起

        以下给出了一个稍显不同的示例：

{% code-tabs %}
{% code-tabs-item title="简单示例" %}
```java
import com.alibaba.fastjson.*;

public class Test
{
    public static void main(String[] args)
    {
        String str = "{" + "name:\"SBZeng\"," + "age:20" + "}";
        System.out.println(str);
        // 将JSON字符串转换为Java Object
        JSONObject obj = JSON.parseObject(str);
        System.out.println("Object = name: " + obj.getString("name") + 
                           ", age: " + obj.getIntValue("age"));
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

        输出结果为：

{% code-tabs %}
{% code-tabs-item title="输出结果" %}
```java
{name:"SBZeng",age:20}
Object = name: SBZeng, age: 20
```
{% endcode-tabs-item %}
{% endcode-tabs %}

        它与之前的示例的区别是：之前我们声明了UserInfo这个类，做反序列化时使用了支持泛型的parse Object，得到UserInfo类的对象。而本次示例中我们使用了不支持泛型的parseObject函数，得到JSON Object类的对象，然后通过JSONObject类的getType\(“field\_name”\)这些方法获取其成员变量的值。这个示例更容易分析，因为它不使用泛型。同时它也提供了另一种使用方法的示范。

        正如我们一开始就讲到，文章的重点是面向对象思想，而非项目本身。所以我们试图通过选择简单的分析样例来避开项目自身的实现复杂性，避开诸多细节，只接触核心流程。

## 2. 需求建模

        下面我们对上述示例做一个分析，首先来看一下如果我们需要完成这一功能，应该怎么做，换言之，进行需求建模：

{% code-tabs %}
{% code-tabs-item title="需求建模" %}
```text
[用例名称]
    反序列化JSON对象
[场景]
    Who：调用者、解析器、返回的对象
    Where：内存
    When：运行时
```
{% endcode-tabs-item %}
{% endcode-tabs %}

        这样的描述太过笼统，我们加入一些细节，调用者在调用接口函数后，由解析器进行词法分析并将结果存储在返回对象中，但词法分析太过复杂，因此我们将解析器拆分为词法分析器和解析器两部分。另一方面，我们以存储器代替返回对象的名称，这是因为返回对象是由解析器创建的用于存储结果的存储器，存储器显然更合本意。从而我们得到以下的用例描述：

{% code-tabs %}
{% code-tabs-item title="需求建模" %}
```text
[用例描述]
    1. 调用者调用接口函数，输入JSON对象
    2. 解析器调用词法分析器进行解析
        2.1 输入串异常，抛出异常信息
    3. 词法分析器进行词法分析、寻找键值对，传递给解析器
    4. 解析器调用存储器存储键值对
    5. 接口函数返回存储器
[用例价值]
    完成JSON对象的反序列化
[约束和限制]
    输入串应该合理
```
{% endcode-tabs-item %}
{% endcode-tabs %}

        通过寻找其中的名词和动词：

{% code-tabs %}
{% code-tabs-item title="需求建模" %}
```text
[名词]：JSON对象、解析器、词法分析器、存储器
[动词]：解析、词法分析、寻找键值对、存储键值对
```
{% endcode-tabs-item %}
{% endcode-tabs %}

        因为JSON对象实质上是一个字符串 ，所以没必要抽象成类。从而我们得到应该抽象出来的类及其方法和属性：

{% code-tabs %}
{% code-tabs-item title="需求建模" %}
```text
[类]：解析器(Parser)
[方法]：解析
[属性]：配置信息、输入串

[类]：词法分析器(Lexer)
[方法]：分析、寻找键值对
[属性]：配置信息、输入串位置

[类]：存储器(Storage)
[方法]：存储
[属性]：储存的信息
```
{% endcode-tabs-item %}
{% endcode-tabs %}

##  3. 实际执行流程

       接下来我们看一下Fastjson如何完成这一任务，即其执行流程，类与类间关系的设计会在下一节谈到。

        首先我们先认识一下JSONObject类，它用于存储反序列化的JSON对象，即上面提到的存储器，其关键在于该类中的成员变量map：

{% code-tabs %}
{% code-tabs-item title="JSONObject.java" %}
```java
private final Map<String, Object> map;
```
{% endcode-tabs-item %}
{% endcode-tabs %}

        map把域名映射到对象上，这已经足够用于存储JSON对象反序列化的结果，也很容易get和set。

        下面我们将进入这个关键的JSON.parseObject函数的执行流程，实际上，接下来的部分是解析器的内容，即调用词法分析器进行词法分析以及调用存储器存储结果，该函数的关键代码段是：

{% code-tabs %}
{% code-tabs-item title="JSON.java" %}
```java
public static JSONObject parseObject(String text) {
    Object obj = parse(text);
    if (obj instanceof JSONObject) {
        return (JSONObject) obj;
    }
    ... // 无关部分，下同
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

        接下来进入parse函数，跳过其中调用重载函数的部分，直接看真正有效的parse函数：

{% code-tabs %}
{% code-tabs-item title="JSON.java" %}
```java
public static Object parse(String text, ParserConfig config, int features) {
    if (text == null) {
        return null;
    }
    DefaultJSONParser parser = new DefaultJSONParser(text, config, features);
    Object value = parser.parse();
    ...
    return value;
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

        我们再进入DefaultJSONParser.parse函数一探究竟，我们仍然跳过调用重载函数部分：

{% code-tabs %}
{% code-tabs-item title="DefaultJSONParser.java" %}
```java
public Object parse(Object fieldName) {
    final JSONLexer lexer = this.lexer;
    switch (lexer.token()) {
            case ...
            case LBRACE:
                JSONObject object = new (lexer.isEnabled(Feature.OrderedField));
                return parseObject(object, fieldNaJSONObjectme);
                                           // 此处filedName = null, 表示解析所有域
            case ERROR:
            default:
                throw new JSONException("syntax error, " + lexer.info());
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

       其中的ERROR和default部分即用例分析中2.1的异常处理部分， 看样子接下来我们应该探索的是parseObject函数。函数parseObject处理字符串的各种case，通过调用JSONLexer接口来解析词法，找到key: value键值对，然后调用以下函数来完成JSONObject对象的创建：

{% code-tabs %}
{% code-tabs-item title="DefaultJSONParser.java" %}
```java
map.put(key, obj);
```
{% endcode-tabs-item %}
{% endcode-tabs %}

        最后它在遇到右大括号时结束解析并返回得到的JSONObject。此处不详谈其实现，是因为函数有446行代码，但大抵只是各种case的处理，实在太过冗长。另一方面，该函数调用词法分析器JSONLexer进行词法分析，进一步的递归阅读将会深入到词法分析的流程，但词法分析的具体实现比较复杂和乏味，这不是我们的目标。看样子，本次执行流程分析是时候结束了。

        实际上，我觉得此处的设计并不合理，一个如此长的函数给代码的维护和阅读带来了及其不良的体验，也会带来大量代码的高度耦合，这并不是良好的程序设计风格（不局限于面向对象程序设计）。就面向对象程序设计而言，这也不符合封装和模块化的思想以及高内聚、低耦合的要求。

        到此我们已经完成了对示例代码执行流程的分析，美中不足的是我们并没有进入其词法分析部分，而是止步于parseObject函数。但正如文章开头所说的，文章重点在于面向对象思想，而非项目具体实现，所以这无关紧。计算机科学的一大魅力就是抽象化和信息隐藏，这可以帮助我们在无需了解太多底层细节的情况下就可以对一个项目进行维护和扩展，可以帮助我们无需递归式地阅读代码，而在必要时终止下来。面向对象思想相对于面向过程提供了更进一步的抽象化和信息隐藏能力，我们应该感谢它。

