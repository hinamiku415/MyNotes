## Qt 文件读写：文本 / 二进制文件核心知识点

在 Qt 开发中，文件读写是必备基础技能，无论是存储配置、保存数据还是读取资源，都离不开`QFile`、`QTextStream`、`QDataStream`这三个核心类。本文聚焦实战，带你快速掌握文本文件与二进制文件的读写逻辑，吃透关键知识点。

## 一、核心类概览：文件操作工具

Qt 的文件读写体系围绕三个核心类展开，分工明确，覆盖绝大多数场景：

|     类名      |    核心用途    |        适用文件类型        |                           关键优势                           |
| :-----------: | :------------: | :------------------------: | :----------------------------------------------------------: |
|    `QFile`    |  底层文件操作  | 所有文件（文本 / 二进制）  | 负责文件打开、关闭、删除、重命名等基础操作，是文件读写的 “载体” |
| `QTextStream` |  文本数据读写  |  文本文件（.txt/.csv 等）  |  支持字符串、数字等文本格式，可设置编码、对齐方式，操作直观  |
| `QDataStream` | 二进制数据读写 | 二进制文件（.bin/.dat 等） | 支持 Qt 数据类型（如`QString`、`QList`）序列化，数据存储紧凑、读取高效 |

核心逻辑：`QFile`负责 “连接文件”，`QTextStream`/`QDataStream`负责 “处理数据”，三者配合完成完整的文件操作。

## 二、文本文件读写（QFile + QTextStream）

文本文件读写是最常用场景（如日志记录、配置存储），核心是 “按行 / 按字符读写”，支持编码设置和格式控制。

### 1. 关键知识点

- **文件打开模式**：必须指定正确的打开模式（如只读`QIODevice::ReadOnly`、只写`QIODevice::WriteOnly`、追加`QIODevice::Append`），否则操作失败。
- **编码设置**：默认编码为系统编码，建议显式设置`UTF-8`（`stream.setCodec("UTF-8")`），避免中文乱码。
- **读写方式**：支持按行读写（`readLine()`/`writeLine()`）、整体读写（`readAll()`/`operator<<`），按需选择。
- **资源释放**：文件使用后需关闭（`file.close()`），或通过`QFile`的析构函数自动关闭。

### 2. 实战代码：文本文件读写示例

```cpp
#include <QFile>
#include <QTextStream>
#include <QDebug>

// 写入文本文件
void writeTextFile(const QString& filePath) {
    // 1. 创建QFile对象，指定文件路径
    QFile file(filePath);
    
    // 2. 打开文件：只写模式，若文件不存在则创建
    if (!file.open(QIODevice::WriteOnly | QIODevice::Text)) {
        qDebug() << "文件打开失败：" << file.errorString();
        return;
    }

    // 3. 创建QTextStream对象，关联QFile
    QTextStream stream(&file);
    // 设置编码为UTF-8，避免中文乱码
    stream.setCodec("UTF-8");

    // 4. 写入数据：支持字符串、数字等，可链式调用
    stream << "Qt文本文件读写示例" << "\n";
    stream << "员工编号：2022001" << "\n";
    stream << "员工姓名：张三" << "\n";
    stream << "年龄：25" << "\n";

    // 5. 关闭文件（可选，析构函数会自动关闭）
    file.close();
    qDebug() << "文本文件写入成功！";
}

// 读取文本文件
void readTextFile(const QString& filePath) {
    QFile file(filePath);
    
    // 打开文件：只读模式
    if (!file.open(QIODevice::ReadOnly | QIODevice::Text)) {
        qDebug() << "文件打开失败：" << file.errorString();
        return;
    }

    QTextStream stream(&file);
    stream.setCodec("UTF-8");

    // 方式1：按行读取（适合大文件，避免内存占用过高）
    qDebug() << "按行读取结果：";
    while (!stream.atEnd()) {
        QString line = stream.readLine(); // 读取一行，不包含换行符
        qDebug() << line;
    }

    // 方式2：整体读取（适合小文件）
    // QString allData = stream.readAll();
    // qDebug() << "整体读取结果：" << allData;

    file.close();
}

// 调用示例
int main() {
    QString filePath = "employee.txt";
    writeTextFile(filePath);
    readTextFile(filePath);
    return 0;
}
```

### 3. 进阶技巧

- 格式控制：`stream.setFieldAlignment(QTextStream::AlignRight)`设置文本右对齐，`stream.setRealNumberNotation(QTextStream::FixedNotation)`设置浮点数固定格式。
- 追加写入：打开模式使用`QIODevice::Append | QIODevice::WriteOnly`，在文件末尾添加内容，不覆盖原有数据。

## 三、二进制文件读写（QFile + QDataStream）

二进制文件读写适合存储结构化数据（如自定义对象、批量数值），数据存储紧凑、读写速度快，且不易被篡改。

### 1. 关键知识点

- **序列化 / 反序列化**：`QDataStream`支持 Qt 内置类型（`int`、`QString`、`QVector`等）和自定义类型的序列化，需保证读写顺序一致。
- **版本兼容性**：建议设置数据流版本（`stream.setVersion(QDataStream::Qt_5_15)`），避免不同 Qt 版本读写数据兼容问题。
- **打开模式**：二进制文件打开无需加`QIODevice::Text`，否则会导致换行符转换，破坏二进制数据结构。
- **状态判断**：通过`stream.status()`判断读写状态（`QDataStream::Ok`为正常），避免读取损坏数据。

### 2. 实战代码：二进制文件读写示例

```cpp
#include <QFile>
#include <QDataStream>
#include <QDebug>
#include <QString>

// 自定义结构体（需实现序列化/反序列化）
struct Employee {
    QString empNo;   // 员工编号
    QString name;    // 姓名
    int age;         // 年龄
    double salary;   // 薪资

    // 序列化：将结构体写入数据流
    friend QDataStream& operator<<(QDataStream& stream, const Employee& emp) {
        stream << emp.empNo << emp.name << emp.age << emp.salary;
        return stream;
    }

    // 反序列化：从数据流读取结构体
    friend QDataStream& operator>>(QDataStream& stream, Employee& emp) {
        stream >> emp.empNo >> emp.name >> emp.age >> emp.salary;
        return stream;
    }
};

// 写入二进制文件
void writeBinaryFile(const QString& filePath) {
    QFile file(filePath);
    
    // 打开二进制只写模式（无QIODevice::Text）
    if (!file.open(QIODevice::WriteOnly)) {
        qDebug() << "文件打开失败：" << file.errorString();
        return;
    }

    QDataStream stream(&file);
    // 设置数据流版本（与Qt版本匹配）
    stream.setVersion(QDataStream::Qt_5_15);

    // 写入单个结构体
    Employee emp1 = {"2022001", "张三", 25, 8000.0};
    stream << emp1;

    // 写入多个结构体（QVector）
    QVector<Employee> empList;
    empList << Employee{"2022002", "李四", 28, 9500.0}
            << Employee{"2022003", "王五", 30, 12000.0};
    stream << empList;

    file.close();
    qDebug() << "二进制文件写入成功！";
}

// 读取二进制文件
void readBinaryFile(const QString& filePath) {
    QFile file(filePath);
    
    if (!file.open(QIODevice::ReadOnly)) {
        qDebug() << "文件打开失败：" << file.errorString();
        return;
    }

    QDataStream stream(&file);
    stream.setVersion(QDataStream::Qt_5_15);

    // 读取单个结构体
    Employee emp1;
    stream >> emp1;
    qDebug() << "单个员工数据：" << emp1.empNo << emp1.name << emp1.age << emp1.salary;

    // 读取多个结构体
    QVector<Employee> empList;
    stream >> empList;
    qDebug() << "员工列表数据：";
    for (const auto& emp : empList) {
        qDebug() << emp.empNo << emp.name << emp.age << emp.salary;
    }

    // 判断读写状态
    if (stream.status() == QDataStream::Ok) {
        qDebug() << "二进制文件读取成功！";
    } else {
        qDebug() << "二进制文件读取失败：数据损坏或格式错误";
    }

    file.close();
}

// 调用示例
int main() {
    QString filePath = "employee.bin";
    writeBinaryFile(filePath);
    readBinaryFile(filePath);
    return 0;
}
```

### 3. 进阶技巧

- 自定义类型序列化：需重载`operator<<`和`operator>>`运算符，确保所有成员变量都被读写。
- 部分读取：通过`file.seek(pos)`定位文件位置，实现从指定偏移量读取数据（适合大文件分片处理）。

## 四、核心知识点总结：避坑 + 选型指南

### 1. 必记避坑点

- 编码问题：文本文件务必设置`UTF-8`编码，二进制文件无需编码设置。
- 打开模式：文本文件加`QIODevice::Text`，二进制文件不加，避免数据损坏。
- 读写顺序：二进制文件读写顺序必须完全一致（如先写`name`再写`age`，读取时也需先读`name`再读`age`）。
- 版本控制：`QDataStream`必须设置版本，否则跨 Qt 版本读写可能失败。

### 2. 文件类型选型指南

|               场景               |      推荐文件类型      |      核心类组合       |
| :------------------------------: | :--------------------: | :-------------------: |
|   日志记录、配置文件、CSV 数据   |        文本文件        | `QFile + QTextStream` |
| 结构化数据、自定义对象、批量数值 |       二进制文件       | `QFile + QDataStream` |
|    中文内容、需人工编辑的文件    | 文本文件（UTF-8 编码） | `QFile + QTextStream` |
|     数据量大、读写效率要求高     |       二进制文件       | `QFile + QDataStream` |

### 3. QFile 常用辅助操作

除了读写，`QFile`还支持文件管理核心操作：

```cpp
QString filePath = "employee.txt";

// 1. 判断文件是否存在
if (QFile::exists(filePath)) {
    qDebug() << "文件已存在";
}

// 2. 复制文件
QFile::copy(filePath, "employee_backup.txt");

// 3. 重命名文件
QFile::rename(filePath, "employee_new.txt");

// 4. 删除文件
QFile::remove("employee_backup.txt");

// 5. 获取文件大小
QFile file(filePath);
qDebug() << "文件大小：" << file.size() << "字节";
```



## 六、总结

Qt 文件读写的核心是 “`QFile`搭载体，`QTextStream`/`QDataStream`处理数据”，掌握以下三点即可应对绝大多数场景：

1. 文本文件：关注编码和打开模式，用`QTextStream`按行 / 整体读写；
2. 二进制文件：关注序列化顺序和版本，用`QDataStream`存储结构化数据；
3. 选型原则：需人工编辑用文本文件，追求效率和结构化用二进制文件。

只要理清 “载体 - 数据 - 操作” 的逻辑，再通过实战练习巩固，文件读写就能成为你的基础技能～