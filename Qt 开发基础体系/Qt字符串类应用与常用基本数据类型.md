## Qt 字符串类应用与常用基本数据类型

在 Qt 开发中，字符串处理和基本数据类型的使用是基础且核心的内容。

## 一、Qt 字符串类（QString）应用

QString 是 Qt 中用于处理字符串的核心类，提供了丰富的成员函数，涵盖字符串组合、查询、转换等多种常用操作，且支持 Unicode 编码，类型安全，使用灵活。

### （一）字符串组合

字符串组合是将多个字符串或不同数据类型拼接为一个完整字符串的操作，QString 提供了多种便捷方式：

1.**二元 “+” 操作符与 “+=” 操作符**

- 功能：直接拼接两个字符串，“+=” 是 “str = str + 字符串” 的简化写法，效率更高。

- 实例代码

  ```cpp
  #include <QCoreApplication> // Qt提供的一个事件循环
  #include <QDebug>
  
  int main(int argc, char *argv[]) {
      QCoreApplication a(argc, argv);
  
      // “+”操作符应用
      QString str1 = "Ling";
      qDebug() << str1; // 输出："Ling"
      str1 = str1 + "Sheng EDU!";
      qDebug() << qPrintable(str1); // qPrintable()去掉字符串双引号，输出：LingSheng EDU!
  
      // “+=”操作符应用
      QString str2 = "12345";
      str2 += "ABCDE";
      qDebug() << qPrintable(str2); // 输出：12345ABCDE
  
      return a.exec();
  }
  ```

 **2.`append () `函数**

- 功能：与 “+=” 操作符功能完全一致，在字符串末尾直接添加另一个字符串，语法更直观。

- 示例代码：

  ```cpp
  QString str1 = "Good";
  QString str2 = "bye";
  str1.append(str2); // str1 变为 "Goodbye"
  qDebug() << qPrintable(str1); // 输出：Goodbye
  str1.append(" Hello world!"); // str1 变为 "Goodbye Hello world!"
  qDebug() << qPrintable(str1); // 输出：Goodbye Hello world!
  ```

**3.`sprintf () `函数**

- 功能：与 C++ 标准库中的 sprintf () 用法一致，通过格式化字符串实现拼接，支持占位符。

- 示例代码：

  ```cpp
  QString strTemp;
  strTemp.sprintf("%s", "Hello ");
  qDebug() << qPrintable(strTemp); // 输出：Hello
  strTemp.sprintf("%s", "Hello world.");
  qDebug() << qPrintable(strTemp); // 输出：Hello world.
  strTemp.sprintf("%s %s", "Welcome", "to you.");
  qDebug() << qPrintable(strTemp); // 输出：Welcome to you.
  ```

**4.`arg ()` 函数（推荐）**

- 核心优势：类型安全、支持 Unicode 编码，可灵活调整占位符顺序（% n 表示第 n 个参数），支持多种数据类型（字符串、数值等）。

- 示例代码：

  ```cpp
  QString strTemp;
  // %1 对应第一个 arg() 参数，%2 对应第二个 arg() 参数
  strTemp = QString("%1 was born in %2.").arg("Sunny").arg(2000);
  qDebug() << strTemp; // 输出："Sunny was born in 2000."
  ```

**5.其他组合函数**

```cpp
- insert (int position, const QString &str)：	//在指定位置插入字符串。
- prepend (const QString &str)：	//在字符串开头添加字符串。
- replace (const QString &before, const QString &after)：	//替换字符串中的指定子串。
```

- 可自行测试，根据实际场景选择使用。

### （二）字符串查询与判断

用于判断字符串的开头、结尾、包含关系，以及字符串比较、类型转换等。

**1.`startsWith ()` 与 `endsWith ()`**

- `startsWith ()`：判断字符串是否以指定子串开头。

- `endsWith ()`：判断字符串是否以指定子串结尾。

- 关键参数：

  - **`Qt::CaseSensitive`：大小写敏感（默认）**。
  - **`Qt::CaseInsensitive`：大小写不敏感**。

- 示例代码

  ```cpp
  QString strTemp = "How are you";
  // 大小写敏感，判断是否以 "How" 开头 → true
  qDebug() << strTemp.startsWith("How", Qt::CaseSensitive);
  // 大小写敏感，判断是否以 "are" 开头 → false
  qDebug() << strTemp.startsWith("are", Qt::CaseSensitive);
  // 大小写不敏感，判断是否以 "HOW" 开头 → true
  qDebug() << strTemp.startsWith("HOW", Qt::CaseInsensitive);
  ```

**2.`contains () `函数**

- 功能：判断字符串中是否包含指定子串，支持大小写敏感 / 不敏感配置。

- 示例代码：

  ```cpp
  QString strTemp = "How are you";
  // 大小写敏感，判断是否包含 "How" → true
  qDebug() << strTemp.contains("How", Qt::CaseSensitive);
  // 大小写不敏感，判断是否包含 "how" → true
  qDebug() << strTemp.contains("how", Qt::CaseInsensitive);
  ```

**3.字符串转数值类型**

- 核心函数：`toInt ()`（转整型）、`toDouble ()`（转双精度浮点型）、`toFloat ()`（转浮点型）、`toLong ()`（转长整型）等。

- 示例代码（以 `toInt () `为例）：

  ```cpp
  QString str = "25";
  bool isSuccess; // 用于判断转换是否成功
  int num = str.toInt(&isSuccess, 10); // 10 表示十进制
  if (isSuccess) {
      qDebug() << "转换成功，数值：" << num; // 输出：25
  }
  
  // 十六进制转换示例
  QString hexStr = "25";
  int hexNum = hexStr.toInt(&isSuccess, 16); // 16 表示十六进制
  if (isSuccess) {
      qDebug() << "十六进制转换结果：" << hexNum; // 输出：37（0x25 = 37）
  }
  ```

**4.`compare () `函数**

- 功能：比较两个字符串的大小，返回整型结果。

- 返回值规则：

  - 小于 0：第一个字符串小于第二个字符串。
  - 等于 0：两个字符串相等。
  - 大于 0：第一个字符串大于第二个字符串。

- 示例代码：

  ```cpp
  // 大小写敏感比较："about" vs "Cat" → 'a' ASCII 小于 'C'，返回负数
  int result1 = QString::compare("about", "Cat", Qt::CaseSensitive);
  // 大小写不敏感比较："abcd" vs "Cat" → 不区分大小写时 "abcd" 小于 "Cat"，返回负数
  int result2 = QString::compare("abcd", "Cat", Qt::CaseInsensitive);
  qDebug() << "result1:" << result1 << ", result2:" << result2;
  ```

**5.`QString` 转 ASCII 码**

- 方法：通过 `toUtf8 ()` 转换为 `QByteArray `，再遍历获取每个字符的 ASCII 码。

- 示例代码：

  ```cpp
  QString str = "ABC abc";
  QByteArray bytes = str.toUtf8();
  for (int i = 0; i < str.size(); i++) {
      // 输出每个字符的 ASCII 码：65(A)、66(B)、67(C)、32(空格)、97(a)、98(b)、99(c)
      qDebug() << int(bytes.at(i));
  }
  ```

## 二、Qt 常见基本数据类型

Qt 定义了一套跨平台的基本数据类型，确保在不同操作系统（Windows、Linux、macOS）和架构（32 位、64 位）下的一致性，需包含头文件 `<QtGlobal>`。

|   类型名称   | 对应 C++ 类型                             | 注释             | 备注                                      |
| :----------: | ----------------------------------------- | ---------------- | ----------------------------------------- |
|   `qint8`    | signed char                               | 有符号 8 位数据  | -                                         |
|   `qint16`   | signed short                              | 有符号 16 位数据 | -                                         |
|   `qint32`   | signed int                                | 有符号 32 位数据 | -                                         |
|   `qint64`   | long long int / __int64                   | 有符号 64 位数据 | Windows 中定义为 __int64                  |
|  `qintptr`   | qint32 或 qint64                          | 指针类型         | 32 位系统为 qint32，64 位系统为 qint64    |
| `qlonglong`  | long long int / __int64                   | 长整型           | Windows 中定义为 __int64                  |
|  `qptrdiff`  | qint32 或 qint64                          | 指针差值类型     | 随系统架构变化                            |
|   `qreal`    | double 或 float                           | 浮点型           | 默认为 double，可通过 `-qreal float `配置 |
|   `quint8`   | unsigned char                             | 无符号 8 位数据  | -                                         |
|  `quint16`   | unsigned short                            | 无符号 16 位数据 | -                                         |
|  `quint32`   | unsigned int                              | 无符号 32 位数据 | -                                         |
|  `quint64`   | unsigned long long int / unsigned __int64 | 无符号 64 位数据 | Windows 中定义为 unsigned __int64         |
|  `quintptr`  | quint32 或 quint64                        | 无符号指针类型   | 随系统架构变化                            |
| `qulonglong` | unsigned long long int / unsigned __int64 | 无符号长整型     | Windows 中定义为 unsigned __int64         |
|   `uchar`    | unsigned char                             | 无符号字符类型   | -                                         |
|    `uint`    | unsigned int                              | 无符号整型       | -                                         |
|   `ulong`    | unsigned long                             | 无符号长整型     | -                                         |
|   `ushort`   | unsigned short                            | 无符号短整型     | -                                         |

### 补充说明

1. 跨平台兼容性：Qt 基本数据类型明确了位数和符号属性，避免了直接使用 C++ 原生类型（如 int、long）在不同平台下位数不一致的问题（例如 32 位系统中 long 为 32 位，64 位系统中为 64 位）。
2. 指针相关类型（`qintptr`、`quintptr`）：用于存储指针或地址差值，确保在 32 位和 64 位系统中都能正确适配，常用于底层内存操作。
3. 后续拓展类型：Qt 还提供了 `QByteArray`（字节数组）、`QVector`（动态数组）、`QVariant`（通用数据类型）等常用容器和类型.

## 三、拓展示例（`QDateTime `与 `QByteArray`）

### （一）`QDateTime`：日期时间处理

```cpp
#include <QCoreApplication>
#include <QDebug>
#include <QDateTime>

int main(int argc, char *argv[]) {
    QCoreApplication a(argc, argv);

    // 获取当前日期时间并格式化输出
    QDateTime dt = QDateTime::currentDateTime();
    // 格式：年-月-日 时:分:秒
    QString strDT = dt.toString("yyyy-MM-dd hh:mm:ss");
    qDebug() << strDT; // 输出示例："2024-05-20 14:30:45"

    return a.exec();
}
```

### （二）QByteArray：字节数组操作

```cpp
#include <QCoreApplication>
#include <QDebug>
#include <QByteArray>

int main(int argc, char *argv[]) {
    QCoreApplication a(argc, argv);

    QByteArray al = "Qt Creator Hello World.";
    QByteArray b1 = al.toLower(); // 转小写
    qDebug() << b1; // 输出："qt creator hello world."
    QByteArray c1 = al.toUpper(); // 转大写
    qDebug() << c1; // 输出："QT CREATOR HELLO WORLD."

    return a.exec();
}
```

