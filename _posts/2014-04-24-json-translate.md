---
title: JSON技术规格翻译
date: 2014-04-24T20:56:35+08:00
categories: [撸代码, JavaScript]
tags: [翻译]
---

最近做毕业设计，学校需要每人附一篇英文文档翻译，平时因为做数据通讯的时候用JSON做交换数据结构很多，所以干脆就翻译JSON RFC文档好了。下面是译文：

## 关于本文

本文讲述关于互联网通讯的信息，并不是准确形式上的标准。你可以随意传播这份文档。

## 版权提示

Copyright (C) The Internet Society (2006)

## 摘要

JavaScript Object Notation (JSON)是一个轻量级的，基于文本的，跨语言的数据交换格式。它衍生自ECMAScript编程语言标准（ECMAScript Programming Language Standard）。JSON定义了一组用于表示结构化数据的可移植的格式化规则。

## 1. 简介

JavaScript Object Notation (JSON)是一种序列化结构化数据的文本格式。它派生自ECMAScript Programming Language Standard, Third Edition [ECMA]定义的JavaScript对象字面量。

JSON包含4种基础数据类型（字符串，数字，布尔和null）和两种结构类型（对象和数组）。

JSON字符串是一个由零或者多个Unicode字符组成的序列。

JSON对象是一个由零或者多个键值对组成的无序集合，其中键值对键名是字符串类型，值则可以是字符串，数字，布尔，null，对象或数组类型。

JSON数组是一个由零或者多个值组成的有序序列。

术语“对象”和“数组”的叫法来源于JavaScript的习惯叫法。

JSON的设计目标是它应当是尽可能小的，可移植的，文本化的，并且可以作为JavaScript的一个子集。

### 1.1 语法约定

本文中的”MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”,”SHOULD NOT”, “RECOMMENDED”, “MAY”, 和 “OPTIONAL” 关键字意思遵循[RFC2119]中的定义。

本文中的语法规则遵循[RFC4234]中的定义。

## 2.  JSON语法规则

一个JSON文本是一串标记序列。JSON序列包括含6种结构字符，字符串，数字和3个字面量。

一个JSON文本可以是对象或者数字的一个序列化字符串。

```
    JSON-text = object / array
```

下面是6种结构字符：

```
    begin-array     = ws %x5B ws  ; [ 左中括号

    begin-object    = ws %x7B ws  ; { 左大括号

    end-array       = ws %x5D ws  ; ] 右中括号

    end-object      = ws %x7D ws  ; } 右大扩号

    name-separator  = ws %x3A ws  ; : 冒号

    value-separator = ws %x2C ws  ; , 逗号
```

所有的六种数据结构字符都允许在前面添加空白字符。

```
    ws = *(
        %x20 /              ; 空格
        %x09 /              ; 水平制表符
        %x0A /              ; 换行符
        %x0D                ; 回车符
        )
```

### 2.1 值

JSON值MUST是一个对象，数组，数字，字符串或下列三个字面量之一：

```
    false null true
```

### 2.2 对象

一个对象的数据结构表示一个被大括号包裹的0个以上的键值对（或称为成员）。键值对中的键名必须是一个字符串，后面是一个冒号，用来分隔键和值。值后面是一个逗号用来分隔值和下一个名键值对的名称。一个对象内的名称SHOULD是唯一的。

```
    object = begin-object [ member *( value-separator member ) ] end-object
    member = string name-separator value
```

### 2.3 数组

一个数组的数据结构表示一对被中括号包裹着0个以上的值（或者叫元素）。值之间用逗号分隔。

```
    array = begin-array [ value *( value-separator value ) ] end-array
```

### 2.4 数字

数字的表示方法类似其他大部分语言。数字包含一个以可选的减号为前缀的整数部分，整数部分
后面可以添加小数部分和/或指数部分。

JSON数字不允许八进制或者十六进制的形式，同时以0开头也是不允许的。

JSON数字的小数部分可以在一个小数点后跟随一位或多位数字。

JSON数字的指数部分以不限大小写的字母E开头，之后可跟一个加号或减号。E和可选的符号后可
跟随一位或多位数字。

JSON数字不允许不能被表示为数字的序列（例如，无穷大和NaN的数字值）。

```
    number        = [ minus ] int [ frac ] [ exp ]
    decimal-point = %x2E                             ; .
    digit1-9      = %x31-39                          ; 1-9
    e             = %x65 / %x45                      ; e E
    exp           = e [ minus / plus ] 1*DIGIT
    frac          = decimal-point 1*DIGIT
    int           = zero / ( digit1-9 *DIGIT )
    minus         = %x2D                             ; -
    plus          = %x2B                             ; +
    zero          = %x30                             ; 0
```

### 2.5 字符串

JSON字符串的表示形式与源自C语言的编程语言的语法类似。一个字符串的开头和结尾都有引号，除了以下一些必须被转义的字符以外所有的Unicode字符都可以直接被放在字符串中：引号（”或’），反斜杠(\)和控制字符（U+0000 到 U+001F）。

任何字符都可以被转义。如果是在定义Basic Multilingual Plane (U+0000 到 U+FFFF)内，则应该表示为6字符序列：反斜杠后面跟一个小写字母u，再跟4位表示字符所在位置的16进制数字。16进制数字中的字母A-F可以是大写的，也可以是小写的。例如：一个只有一个反斜杠组成的字符串可以表示为”\u005C”。

另外，有一些流行的字符可以用两字符序列来转义，例如：一个只有一个反斜杠组成的字符串可以表示为”\\”。

要转义不在定义Basic Multilingual Plane内的字符，则使用表示为UTF-16编码代理对的12字符序列。例如：一个只包含G谱字符（U+1D11E）的字符串可以被表示为”\uD834\uDD1E”。

```
    string = quotation-mark *char quotation-mark
    char = unescaped / escape （
        0x22 /                                             ; " 引号 U+0022 
        0x5C /                                             ; \ 反斜杠 U+005c
        0x2F /                                             ; / 斜杠 U+002F
        0x62 /                                             ; b 退格符 U+0062
        0x66 /                                             ; f 分页符 U+0066
        0x6E /                                             ; n 换行符 U+006E
        0x72 /                                             ; r 回车符 U+0072
        0x74 /                                             ; t 水平制表符 U+0074
        0x75 4HEXDIG                                       ; uXXXX U+XXXX 
    )
    escape = %x5C                                          ; \
    quotation-mark = %x22                                  ; "
    unescaped = %x20-21 / %x23-5B / %x5D-10FFFF
```

## 3. 编码

JSON文本必须以Unicode编码编码，默认的编码格式为UTF-8。

由于JSON文本的头两个字符一定是ASCII字符[RFC0020]，因此可以通过观察第一组4个8位字节来判断字节流是UTF-8，UTF-16（BE或LE）还是UTF-32（BE或LE）编码的。

```
    00 00 00 xx UTF-32BE
    00 xx 00 xx UTF-16BE
    xx 00 00 00 UTF-32LE
    xx 00 xx 00 UTF-16BE
    xx xx xx xx UTF-8
```

## 4. 解析

一个JSON解析器可以转换JSON文本到其它形式。一个JSON解释器MUST能接受符合JSON语法的所有文本。一个JSON解析器MAY能接受非JSON形式的文本或扩展。

JSON解析器的实现必须告知用户它接受文本大小的限制，同时必须告知用户递归对象层数的深度，和相应数字字符串的限制。

## 5. 生成器

JSON生成器能够生成JSON文本，生成结果MUST严格符合JSON的语法。

## 6. IANA方面的考虑

JSON文本的MIME媒体类型是application/json。

类型名称： application

子类型名称： json

必选参数： n/a

可选参数： n/a

编码方面的考虑： 如果是UTF-8则是8位字节，如果是UTF-16和UTF-32则是二进制

JSON可以用UTF-8,UTF-16和UTF-32编码表示。如果使用UTF-8,则JSON是8位字节兼容的。如果是UTF-16或UTF-32，则必须使用二进制内容传输编码。

安全方面的考虑：

- 通常，脚本语言都有安全问题，JSON作为JavaScript的一个子集，但由于它排除了分配和调用，所以它是安全的。
- 如果JSON文本中除去字符串部分的字符都是JSON标记（token）字符，则它可以安全的传递给JavaScript的eval()方法（用来编译和执行一个字符串的方法）。JavaScript中通过分别调用两个正则表达式的test和replace方法可以快速的确定是否满足该条件。

```
    var my_JSON_object = !(/[^,:{}\[\]0-9.\-+Eaeflnr-u \n\r\t]/.test(text.replace(/"(\\.|[^"\\])*"/g, ''))) && eval('(' + text + ')');
```

互操作性方面的考虑：n/a

发布规范：RFC 4627

使用这个媒体类型的应用程序：

JSON曾被用于用以下所有编程语言编写的应用程序间传递数据：ActionScript, C, C#, ColdFusion, Common Lisp, E, Erlang, Java, JavaScript, Lua, Objective CAML, Perl, PHP, Python, Rebol, Ruby, and Scheme.

额外的信息：

- 魔术数字： n/a
- 文件扩展名： .json
- Macintosh文件类型的代码： TEXT

进一步的信息请联系：

- Douglas Crockford

- douglas@crockford.com

预期的用法： COMMON

受限制的用法： 无

作者：

- Douglas Crockford
- douglas@crockford.com

主导修改人：

- Douglas Crockford
- douglas@crockford.com

## 7. 安全方面的考虑

参照第六节的“安全方面的考虑”。

## 8. 例子

下面是一个JSON对象：

```
    {
        "Image": {
            "Width": 800,
            "Height": 600,
            "Title": "View from 15th Floor",
            "Thumbnail": {
            "Url": "http://www.example.com/image/481989943",
            "Height": 125,
            "Width": "100"
        },
        "IDs": [116, 943, 234, 38793]
        }
    }
```    
    
这个对象里的Imgage成员是一个完整的成员对象，而它的IDS成员则是一个以数字组成的数组。

下面是一个包括两个对象的JSON数组：

```
    [
          {
             "precision": "zip",
             "Latitude":  37.7668,
             "Longitude": -122.3959,
             "Address":   "",
             "City":      "SAN FRANCISCO",
             "State":     "CA",
             "Zip":       "94107",
             "Country":   "US"
          },
          {
             "precision": "zip",
             "Latitude":  37.371991,
             "Longitude": -122.026020,
             "Address":   "",
             "City":      "SUNNYVALE",
             "State":     "CA",
             "Zip":       "94085",
             "Country":   "US"
          }
    ]
```

## 9. 参考

### 9.1 参考的规范

1. [ECMA]    European Computer Manufacturers Association, "ECMAScript Language Specification 3rd Edition", December 1999,<http://www.ecma-international.org/publications/files/ecma-st/ECMA-262.pdf>.
2. [RFC0020] Cerf, V., "ASCII format for network interchange", RFC 20,October 1969.
3. [RFC2119] Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, March 1997.
4. [RFC4234] Crocker, D. and P.  Overell, "Augmented BNF for Syntax Specifications: ABNF", RFC 4234, October 2005.