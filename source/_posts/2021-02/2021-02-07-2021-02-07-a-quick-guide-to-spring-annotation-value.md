---
title: Spring 注解 @Value 快速使用指南
comments: false
date: 2021-02-07 09:02:50
tags:
  - Spring Basics
categories:
  - 后端开发
  - Spring
cover: https://cdn.jsdelivr.net/gh/bianyun1981/CDN@latest/img/posts/2021-02/2021-02-07-095839-100.jpg
description: 快速了解 Spring 注解 @Value 的使用。@Value 注解用于将字面值或配置文件中的值注入到 Spring 管理的 bean 的字段中，支持三种注入方式：实例属性、构造方法、setter 方法。
---

# 基本用法

`@Value` 注解支持从 `Properties` 配置文件或者 `YAML` 配置文件中读取配置并注入到 `bean` 中，鉴于 `YAML` 配置的比 `Properties` 配置更有层次，功能也更强大，本文中均以 `YAML` 配置文件为例进行说明。

```java
// 注入字面常量值
@Value("This is a literal string value")
private String literalStringValue;

// 注入配置文件中的值
@Value("${demo-cfg.basic.string}")
private String stringValue;

// 指定默认值，如果配置文件中找不到相应的 key，则使用默认值
@Value("${demo-cfg.basic.string:defaultValue}")
private String stringValueWithDefaultValue;

// 指定默认值为空字符串（即长度为 0 的字符串）
@Value("${demo-cfg.basic.string:}")
private String stringValueWithDefaultEmptyString;
```

注意：使用 `@Value` 注解的类必须是 Spring 管理的 bean，即：该类上面配置有下面任意一种注解：`@Component`, `@Service`, `@Controller`, `@RestController`, `@Configuration`.

# 三种注入方式

@Value 注解支持三种注入方式：实例属性、构造方法、setter 方法。

## 实例属性

```java
@Value("${demo-cfg.basic.string}")
private String stringValue;
```

## 构造方法

```java
@Component
public class DemoComponent {
    private String stringValue; 

    @Autowired
    public DemoComponent(@Value("${demo-cfg.basic.string}") String stringValue) {
        this.stringValue = stringValue;
    }
}
```

## setter 方法

```java
@Component
public class DemoComponent {
    private String stringValue;
    
    @Autowired
    public void setStringValue(@Value("${demo-cfg.basic.string}") String stringValue) {
        this.stringValue = stringValue;
    }
}
```

一般推荐使用实例属性注入方式，比较简洁。

# 初级用法

由于字符串类型已经在基本用法中介绍过，此处不再赘述。下面对基本类型、包装类型、数组类型、List类型分别举例如何使用 `@Value` 注解。为了便于比对，采用三个 Tab页分别放置 Java代码、YAML配置、变量结果打印。Java变量结果打印的方法如下：

```java
private static void print(String name, Object value) {
    // 此处将 byte[] 数组中的 byte 转换成 int 类型后再打印，避免打印出的是乱码字符
    if (value instanceof byte[]) {
        byte[] byteArray = (byte[]) value;
        int[] tempIntArray = new int[byteArray.length];
        for (int i = 0; i < byteArray.length; i++) {
            tempIntArray[i] = byteArray[i];
        }
        value = tempIntArray;
    }
    System.out.println(StrUtil.format("{} = {}", 
          StrUtil.padAfter(name, 12, StrUtil.C_SPACE), value));
}
```

## 基本类型

{% tabs primitive types %}

<!-- tab Java -->

```java
    @Value("${demo-cfg.basic.integer}")
    private int integerValue;

    @Value("${demo-cfg.basic.integerHex}")
    private int integerHex;

    @Value("${demo-cfg.basic.integerOct}")
    private int integerOct;

    @Value("${demo-cfg.basic.integerBin}")
    private int integerBin;

    @Value("${demo-cfg.basic.long}")
    private long longValue;

    @Value("${demo-cfg.basic.float}")
    private float floatValue;

    @Value("${demo-cfg.basic.double}")
    private double doubleValue;

    @Value("${demo-cfg.basic.char}")
    private char charValue;

    @Value("${demo-cfg.basic.byte}")
    private byte byteValue;

    @Value("${demo-cfg.basic.short}")
    private short shortValue;

    @Value("${demo-cfg.basic.boolean}")
    private boolean booleanValue;
```

<!-- endtab -->

<!-- tab YAML -->

```yaml
demo-cfg:
  basic:
    integer: 12345
    integerHex: 0xABCDE       # 十六进制整数
    integerOct: 012345        # 八进制整数
    integerBin: 0b11001100    # 二进制整数
    long: 0xCAFEBABE          # 十六进制整数
    float: 123.456
    double: 12.3456789
    char: a
    byte: 68
    short: 12345
    boolean: true
```

<!-- endtab -->

<!-- tab Output -->

```java
    integerValue = 12345
    integerHex   = 703710
    integerOct   = 5349
    integerBin   = 204
    longValue    = 3405691582
    floatValue   = 123.456
    doubleValue  = 12.3456789
    charValue    = a
    byteValue    = 68
    shortValue   = 12345
    booleanValue = true
```

<!-- endtab -->

{% endtabs %}

**注意：**对于基本类型，如果配置中相应的 `key` 不存在，那么 `@Value` 注解中必须要指定默认值，否则会报错；或者即使相应的 `key` 存在，但是对应的 `value` 没有设置或格式错误，也会报错。

```java
    @Value("${demo-cfg.basic.integer:16888}")
    private int integerValue;

    @Value("${demo-cfg.basic.float:1.23}")
    private float floatValue;

    @Value("${demo-cfg.basic.boolean:true}")
    private boolean booleanValue;
```

## 包装类型

{% tabs wrapper classes %}

<!-- tab Java -->

```java
    @Value("${demo-cfg.basic.integer}")
    private Integer integerWrapper;

    @Value("${demo-cfg.basic.long}")
    private Long longWrapper;

    @Value("${demo-cfg.basic.float}")
    private Float floatWrapper;

    @Value("${demo-cfg.basic.double}")
    private Double doubleWrapper;

    @Value("${demo-cfg.basic.char}")
    private Character charWrapper;

    @Value("${demo-cfg.basic.byte}")
    private Byte byteWrapper;

    @Value("${demo-cfg.basic.short}")
    private Short shortWrapper;

    @Value("${demo-cfg.basic.boolean}")
    private Boolean booleanWrapper;
```

<!-- endtab -->

<!-- tab YAML -->

```yaml
demo-cfg:
  basic:
    integer: 12345
    long: 0xCAFEBABE          # 十六进制整数
    float: 123.456
    double: 12.3456789
    char: a
    byte: 68
    short: 12345
    boolean: true
```

<!-- endtab -->

<!-- tab Output -->

```java
    integerWrapper = 12345
    longWrapper    = 3405691582
    floatWrapper   = 123.456
    doubleWrapper  = 12.3456789
    charWrapper    = a
    byteWrapper    = 68
    shortWrapper   = 12345
    booleanWrapper = true
```

<!-- endtab -->

{% endtabs %}

**注意：**与基本类型一样，如果配置中相应的 `key` 不存在，那么 `@Value` 注解中必须要指定默认值，否则会报错；但当 `key` 对应的 `value` 值设置为空（空字符串或者空白字符）时，包装类型的值为 `null`.

## 数组类型

{% tabs array types %}

<!-- tab Java -->

```java
    @Value("${demo-cfg.advanced.array.string}")
    private String[] stringArray;

    @Value("${demo-cfg.advanced.array.integer}")
    private int[] integerArray;

    @Value("${demo-cfg.advanced.array.long}")
    private long[] longArray;

    @Value("${demo-cfg.advanced.array.float}")
    private float[] floatArray;

    @Value("${demo-cfg.advanced.array.double}")
    private double[] doubleArray;

    @Value("${demo-cfg.advanced.array.char}")
    private char[] charArray;

    @Value("${demo-cfg.advanced.array.byte}")
    private byte[] byteArray;

    @Value("${demo-cfg.advanced.array.short}")
    private short[] shortArray;

    @Value("${demo-cfg.advanced.array.boolean}")
    private boolean[] booleanArray;
```

<!-- endtab -->

<!-- tab YAML -->

```yaml
demo-cfg:
  advanced:
    array:
      string: Big Bang Theory, Sheldon Cooper
      integer: 1234, 56789
      long: 0xCAFEBABE, 0xDEADBEEF
      float: 1.2, 3.45, 6.789
      double: 1.23456789, 2.3456789, 3.45678912345
      char: abc
      byte: -12, 67, 89
      short: 12345, 23456, 32767
      boolean: true, false, true
```

<!-- endtab -->

<!-- tab Output -->

```java
    stringArray  = [Big Bang Theory, Sheldon Cooper]
    integerArray = [1234, 56789]
    longArray    = [3405691582, 3735928559]
    floatArray   = [1.2, 3.45, 6.789]
    doubleArray  = [1.23456789, 2.3456789, 3.45678912345]
    charArray    = [a, b, c]
    byteArray    = [-12, 67, 89]
    shortArray   = [12345, 23456, 32767]
    booleanArray = [true, false, true]
```

<!-- endtab -->

{% endtabs %}

**注意：**数组元素类型也可以是包装类型。另外，数组类型配置的值默认是由逗号分隔符（英文半角）进行分隔，逗号分隔符前后可以有若干空白字符，不会影响解析。如果希望分隔符是其它符号，则需要用到高级用法中的 `SpEL`，参见后面高级用法之 [使用 Spring 表达式 SpEL](#使用-Spring-表达式-SpEL) 。

**例外：**`char[]` 类型的数组，配置值不能是逗号分隔的如 "a, b, c" 这样的值，否则注入进来得到的 `char[]` 是一个长度为 7 的数组，其值为 `['a', ',', ' ', 'b', ',', ' ', 'c']`，即中间的逗号和空格都被算作一个 `char` 注入 `char[]` 数组中了。对于包装类型数组 `Character[]` 和下面的 `List` 类型 `List<Character>`, 其值仍然是用逗号分隔。

## List类型

与数组类型类似，只是 `List` 类型的元素类型只支持包装类型，不支持基本类型。

{% tabs array types %}

<!-- tab Java -->

```java
    @Value("${demo-cfg.advanced.array.string}")
    private List<String> stringList;

    @Value("${demo-cfg.advanced.array.integer}")
    private List<Integer> integerList;

    @Value("${demo-cfg.advanced.array.long}")
    private List<Long> longList;

    @Value("${demo-cfg.advanced.array.float}")
    private List<Float> floatList;

    @Value("${demo-cfg.advanced.array.double}")
    private List<Double> doubleList;

    @Value("${demo-cfg.advanced.array.char}")
    private List<Character> charList;

    @Value("${demo-cfg.advanced.array.byte}")
    private List<Byte> byteList;

    @Value("${demo-cfg.advanced.array.short}")
    private List<Short> shortList;

    @Value("${demo-cfg.advanced.array.boolean}")
    private List<Boolean> booleanList;
```

<!-- endtab -->

<!-- tab YAML -->

```yaml
demo-cfg:
  advanced:
    array:
      string: Big Bang Theory, Sheldon Cooper
      integer: 1234, 56789
      long: 0xCAFEBABE, 0xDEADBEEF
      float: 1.2, 3.45, 6.789
      double: 1.23456789, 2.3456789, 3.45678912345
      char: a, b, c
      byte: -12, 67, 89
      short: 12345, 23456, 32767
      boolean: true, false, true
```

<!-- endtab -->

<!-- tab Output -->

```java
    stringList   = [Big Bang Theory, Sheldon Cooper]
    integerList  = [1234, 56789]
    longList     = [3405691582, 3735928559]
    floatList    = [1.2, 3.45, 6.789]
    doubleList   = [1.23456789, 2.3456789, 3.45678912345]
    charList     = [a, b, c]
    byteList     = [-12, 67, 89]
    shortList    = [12345, 23456, 32767]
    booleanList  = [true, false, true]
```

<!-- endtab -->

{% endtabs %}

# 高级用法

## 使用 Spring 表达式 SpEL

{% tabs array types %}

<!-- tab Java -->

```java
    // 非默认逗号分隔符
    // 通过 SpEL 表达式对配置字符串值进行 split 操作
    // split 方法的参数为正则表达式 \s*__\s*
    @Value("#{'${demo-cfg.advanced.spel.short}'.split('\\s*__\\s*')}")
    private List<Short> shortListSpEL;

    // 还可以直接注入某个系统属性的值
    // 如果该系统属性没有被定义，则最终 priority 的值为 null
    @Value("#{systemProperties['priority']}")
    private Integer priority;

    // SpEL 表达式也可以指定系统属性未定义时的默认值
    @Value("#{systemProperties['priority'] ?: '123'}")
    private Integer priorityWithDefault;
```

<!-- endtab -->

<!-- tab YAML -->

```yaml
demo-cfg:
  advanced:
    spel:
      short: 12345  __ 23456   __   32767
```

<!-- endtab -->

<!-- tab Output -->

```java
    shortListSpEL       = [12345, 23456, 32767]
    priority            = null
    priorityWithDefault = 123
```

<!-- endtab -->

{% endtabs %}

## 利用 SpEL 注入 Map

{% tabs array types %}

<!-- tab Java -->

```java
    @Value("#{${demo-cfg.advanced.spel.map}}")
    private Map<String, String> configMap;
```

<!-- endtab -->

<!-- tab YAML -->

```yaml
demo-cfg:
  advanced:
    spel:
      map: '{
        host: "192.168.88.168",
        port: "3306",
        username: "Sheldon",
        password: "Cooper"
      }'
```

<!-- endtab -->

<!-- tab Output -->

```java
    configMap = {host=192.168.88.168, port=3306, username=Sheldon, password=Cooper}
```

<!-- endtab -->

{% endtabs %}

**注意：**YAML 配置中 `map` 类型的值必须是 `JSON` 格式的字符串（最外层的引号为单引号或双引号都可以，外层为单引号，则 `JSON` 内部用双引号，反之亦然）。

# 总结

本文对 `Spring` 的 `@Value` 注解的使用进行了比较完整的介绍，初级用法中分别对基本类型、包装类型、数组类型、List 类型的注入给出了示例，高级用法中对 `@Value` 注解中如何使用 `SpEL` 表达式进行了简单介绍。