## `QVariant` 类深度解析与实战应用笔记

在 Qt 开发中，`QVariant` 类是一个非常实用且强大的工具，它本质上是 C++ 联合（Union）数据类型，能够灵活存储多种 Qt 类型的值，包括常见的基础类型、Qt 容器类型，甚至支持用户自定义类型，极大地简化了数据传递和处理的逻辑。本文将详细拆解 `QVariant` 类的核心特性、常用类型映射，并通过实战案例快速掌握其应用场景。

## 一、`QVariant` 类核心特性

`QVariant`的核心优势在于**类型灵活性**，它就像一个 “万能容器”，可以：

1. 存储 Qt 内置的多种基础类型（如 `bool`、`int`、`double`、`QString` 等）；
2. 存储 Qt 容器类型（如 `QList`、`QMap`、`QStringList `等）；
3. 存储 Qt 图形相关类型（如 `QColor`、`QBrush`、`QPixmap` 等）；
4. 支持用户自定义类型（如结构体、自定义类）；
5. 提供便捷的类型转换接口，确保数据安全提取。

## 二、`QVariant`常用枚举类型映射表

`QVariant` 通过枚举类型 `QVariant::type` 标识存储的数据类型，以下是常用的类型映射关系，方便开发中快速查询：

|       枚举变量       | 对应的实际类型 |        枚举变量        |      对应的实际类型       |
| :------------------: | :------------: | :--------------------: | :-----------------------: |
| `QVariant::Invalid`  |    无效类型    |    `QVariant::Time`    |          `QTime`          |
|  `QVariant::Region`  |   `QRegion`    |    `QVariant::Line`    |          `QLine`          |
|  `QVariant::Bitmap`  |   `QBitmap`    |  `QVariant::Palette`   |        `QPalette`         |
|   `QVariant::Bool`   |     `bool`     |    `QVariant::List`    |     `QList<QVariant>`     |
|  `QVariant::Brush`   |    `QBrush`    | `QVariant::SizePolicy` |       `QSizePolicy`       |
|   `QVariant::Size`   |    `QSize`     |   `QVariant::String`   |         `QString`         |
|   `QVariant::Char`   |    `QChar`     |    `QVariant::Map`     | `QMap<QString, QVariant>` |
|  `QVariant::Color`   |    `QColor`    | `QVariant::StringList` |       `QStringList`       |
|  `QVariant::Cursor`  |   `QCursor`    |   `QVariant::Point`    |         `QPoint`          |
|   `QVariant::Date`   |    `QDate`     |    `QVariant::Pen`     |          `QPen`           |
| `QVariant::DateTime` |  `QDateTime`   |   `QVariant::Pixmap`   |         `QPixmap`         |
|  `QVariant::Double`  |    `double`    |    `QVariant::Rect`    |          `QRect`          |
|   `QVariant::Font`   |    `QFont`     |   `QVariant::Image`    |         `QImage`          |
|   `QVariant::Icon`   |    `QIcon`     |  `QVariant::UserType`  |             -             |

## 三、实战案例：QVariant 各类用法详解

下面通过完整的代码案例，演示 QVariant 存储基础类型、容器类型、自定义结构体的具体用法，所有案例均基于 Qt 环境编写。

### 1. 环境准备与头文件依赖

使用 `QVariant` 需包含核心头文件，若涉及自定义类型，需通过 `Q_DECLARE_METATYPE` 声明，确保 Qt 元对象系统识别。

**头文件（`mainwindow.h`）**：

```cpp
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>
#include <QVariant>
#include <QDebug>
#include <QColor>
#include <QMap>
#include <QStringList>

// 定义自定义学生结构体
struct student {
    int iNo;         // 学号
    QString strName; // 姓名
    int score;       // 分数
};
// 声明元类型，使 QVariant 支持该结构体
Q_DECLARE_METATYPE(student) //

class MainWindow : public QMainWindow
{
    Q_OBJECT
public:
    MainWindow(QWidget *parent = nullptr);
    ~MainWindow();
};

#endif // MAINWINDOW_H
```

### 2. 基础类型存储与转换

`QVariant` 可直接存储 `int`、`double`、`QString` 等基础类型，通过 `toXxx()` 系列函数（如 `toInt()`、`toString()`）安全转换提取。

**核心代码**：

```cpp
// 存储 int 类型
QVariant qv1(298);
qDebug() << "qv1（int类型）:" << qv1.toInt(); // 输出：298

// 存储 QString 类型
QVariant qv2("LingShengEDU");
qDebug() << "qv2（QString类型）:" << qv2.toString(); // 输出："LingShengEDU"
```

### 3. `QMap `容器与 QVariant 结合

`QVariant` 常与 `QMap` 搭配，实现 “键值对存储多种类型数据” 的场景（类似字典），尤其适合配置项、参数传递等需求。

**核心代码**：

```cpp
// 定义 QMap（键为 QString，值为 QVariant）
QMap<QString, QVariant> qmap;

// 存储不同类型的数据
qmap["int"] = 20000;                // 整型
qmap["double"] = 99.88;             // 浮点型
qmap["string"] = "Good";            // 字符串型
qmap["color"] = QColor(255, 255, 0); // QColor 类型（黄色）

// 提取并输出数据
qDebug() << "\nQMap 中存储的各类数据：";
qDebug() << "int 类型：" << qmap["int"] << "，转换后：" << qmap["int"].toInt();
qDebug() << "double 类型：" << qmap["double"] << "，转换后：" << qmap["double"].toDouble();
qDebug() << "string 类型：" << qmap["string"] << "，转换后：" << qmap["string"].toString();
qDebug() << "QColor 类型：" << qmap["color"] << "，转换后：" << qmap["color"].value<QColor>();
```

**输出结果**：

```json
int 类型： 20000 ，转换后： 20000
double 类型： 99.88 ，转换后： 99.88
string 类型： "Good" ，转换后： "Good"
QColor 类型： QColor(255, 255, 0) ，转换后： QColor(255, 255, 0)
```

### 4. `QStringList` 容器与 QVariant 结合

`QVariant` 支持存储 `QStringList` 类型，通过 `type()` 函数判断类型后，用 `toStringList()` 提取列表数据。

**核心代码**：

```cpp
// 创建 QStringList 列表
QStringList qsl;
qsl << "A" << "B" << "C" << "D" << "E" << "F";

// 存储到 QVariant 中
QVariant qvsl(qsl);

// 判断类型并提取数据
qDebug() << "\nQStringList 存储与提取：";
if (qvsl.type() == QVariant::StringList) {
    QStringList qlist = qvsl.toStringList();
    for (int i = 0; i < qlist.size(); i++) {
        qDebug() << "索引" << i << "：" << qlist.at(i);
    }
}
```

**输出结果**：

```json
QStringList 存储与提取：
索引 0 ： "A"
索引 1 ： "B"
索引 2 ： "C"
索引 3 ： "D"
索引 4 ： "E"
索引 5 ： "F"
```

### 5. 自定义结构体与 QVariant 结合

要让 QVariant 支持自定义结构体，需满足两个条件：

1. **用 `Q_DECLARE_METATYPE` 声明结构体（头文件中）；**
2. **通过 `QVariant::fromValue()` 存储，`value<T>()` 或 `qvariant_cast<T>()` 提取。**

**核心代码**：

```cpp
// 初始化自定义结构体
student stu;
stu.iNo = 202201;
stu.strName = "sunny";
stu.score = 715;

// 存储结构体到 QVariant
QVariant qstu = QVariant::fromValue(stu);

// 判断是否可转换为 student 类型，再提取数据
qDebug() << "\n自定义结构体存储与提取：";
if (qstu.canConvert<student>()) {
    // 方法1：value<T>() 提取
    student temp = qstu.value<student>();
    qDebug() << "方法1 - 学号：" << temp.iNo << "，姓名：" << temp.strName << "，分数：" << temp.score;

    // 方法2：qvariant_cast<T>() 提取
    student qtemp = qvariant_cast<student>(qstu);
    qDebug() << "方法2 - 学号：" << qtemp.iNo << "，姓名：" << qtemp.strName << "，分数：" << qtemp.score;
}
```

**输出结果**：

```json
自定义结构体存储与提取：
方法1 - 学号： 202201 ，姓名： "sunny" ，分数： 715
方法2 - 学号： 202201 ，姓名： "sunny" ，分数： 715
```

## 四、关键注意事项

1. **类型匹配**：提取数据时，需**确保转换的目标类型与存储类型一致**，否则可能返回默认值（如 `toInt()` 转换失败返回 0）；
2. **自定义类型声明**：必须通过 **`Q_DECLARE_METATYPE` 声明自定义类型**，否则 `QVariant` 无法识别；
3. **转换安全性**：使用 `canConvert<T>()` 先判断是否支持转换，再调用 `value<T>()` 或 `qvariant_cast<T>()`，避免崩溃；
4. **容器类型处理**：存储 Qt 容器（如 `QList`、`QMap`）时，容器的元素类型需是 `QVariant`支持的类型。