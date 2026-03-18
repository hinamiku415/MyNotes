## Qt 核心容器类QMap、QHash 与 QVector

在 Qt 开发中，容器类是存储和管理数据的核心工具。本文详细整理了 Qt 中常用的三大容器类 ——QMap、QHash、QVector 的核心特性、常用操作及实际应用场景

## 一、QMap 类：有序键值对映射容器

QMap<Key, T> 是一种**键值对（key-value）** 容器，核心特点是**按照键（Key）的自然顺序存储数据**，支持 “一键一值” 和 “一键多值” 两种模式，适用于需要有序遍历或基于键快速查找的场景。

### （一）核心特性

1. 键值对映射：每个键（Key）默认对应一个值（T），键唯一且有序。
2. 一键多值支持：通过 `insertMulti()` 函数或 `QMultiMap` 类实现一个键对应多个值。
3. 有序存储：数据按键的自然顺序（如字符串字典序、数值大小序）排列。
4. 键要求：**键类型必须提供 `<` 运算符（用于排序）**

### （二）常用操作及示例代码

#### **1.头文件与实例创建**

```cpp
#include <QCoreApplication>
#include <QDebug>
#include <QMap>
#include <QMultiMap> // 一键多值时需包含

int main(int argc, char *argv[]) {
    QCoreApplication a(argc, argv);

    // 创建 QMap 实例：键为 QString 类型，值为 int 类型
    QMap<QString, int> qmap;
```

#### **2. 插入数据**

支持两种插入方式：**数组下标法（`[]`）和 `insert()` 函数，重复插入同一键会覆盖原有值**。

```cpp
// 方式 1：数组下标法
qmap["Chinese"] = 119;
qmap["English"] = 120;

// 方式 2：insert() 函数
qmap.insert("Math", 115);
qmap.insert("Physics", 99);
qmap.insert("Chemistry", 100);

qDebug() << "插入后的数据：" << qmap; 
// 输出（有序）：QMap(("Chinese",119), ("Chemistry",100), ("English",120), ("Math",115), ("Physics",99))
```

#### **3. 删除数据**

通过键（Key）删除对应键值对，使用 `remove(const Key &key)` 函数。

```cpp
qmap.remove("Chemistry"); // 删除键为 "Chemistry" 的数据
qDebug() << "删除后的数据：" << qmap << endl;
// 输出：QMap(("Chinese",119), ("English",120), ("Math",115), ("Physics",99))
```

#### **4. 遍历数据**

支持两种迭代方式：Java 风格迭代器（`QMapIterator`）和 STL 风格迭代器。

```cpp
// 方式 1：Java 风格迭代器（推荐，简洁直观）
QMapIterator<QString, int> itr(qmap);
qDebug() << "Java 风格遍历：";
while (itr.hasNext()) {
    itr.next(); // 移动到下一个元素
    qDebug() << itr.key() << ":" << itr.value();
}

// 方式 2：STL 风格迭代器（兼容 STL 语法）
qDebug() << "\nSTL 风格遍历：";
QMap<QString, int>::const_iterator stritr = qmap.constBegin();
while (stritr != qmap.constEnd()) {
    qDebug() << stritr.key() << ":" << stritr.value();
    stritr++; // 迭代器自增
}
```

#### **5. 查找数据**

支持 “键查值” 和 “值查键”，分别使用 `value(const Key &key)` 和 `key(const T &value)` 函数。

```cpp
qDebug() << "\n查找结果：";
qDebug() << "键 Math 对应的值：" << qmap.value("Math"); // 键查值 → 115
qDebug() << "值 99 对应的键：" << qmap.key(99); // 值查键 → "Physics"
```

#### **6. 修改数据**

重复插入同一键即可覆盖原有值，或直接通过下标赋值修改。

```cpp
qmap.insert("Math", 118); // 覆盖原有值 115
qDebug() << "\n修改后 Math 的值：" << qmap.value("Math"); // 输出 → 118
```

#### **7. 判断是否包含某个键**

使用 `contains(const Key &key)` 函数，返回布尔值。

```cpp
qDebug() << "\n包含性判断：";
qDebug() << "是否包含 Chinese：" << qmap.contains("Chinese"); // true
qDebug() << "是否包含 Chemistry：" << qmap.contains("Chemistry"); // false
```

#### **8. 获取所有键和值**

通过 `keys()` 和 `values()` 函数分别**获取键列表和值列表**（顺序与存储一致）。

```cpp
QList<QString> keyList = qmap.keys();
QList<int> valueList = qmap.values();
qDebug() << "\n所有键：" << keyList; // 输出：("Chinese", "English", "Math", "Physics")
qDebug() << "所有值：" << valueList; // 输出：(119, 120, 118, 99)
```

#### **9. 一键多值（QMultiMap**）

`QMultiMap` 是 `QMap` 的子类，专门用于 **“一键多值” 场景**，插入相同键不会覆盖，而是追加值。

```cpp
qDebug() << "\n一键多值（QMultiMap）：";
QMultiMap<QString, QString> mulmap;
mulmap.insert("student", "no");
mulmap.insert("student", "name");
mulmap.insert("student", "sex");
mulmap.insert("student", "age");
mulmap.insert("student", "high");
mulmap.insert("student", "weight");

qDebug() << mulmap; 
// 输出：QMap(("student","age"), ("student","high"), ("student","name"), ("student","no"), ("student","sex"), ("student","weight"))
```

```cpp
    return a.exec();
}
```

## 二、QHash 类：高效无序哈希容器

QHash<Key, T> 与 QMap 功能高度相似，核心区别是**基于哈希表实现**，查找速度更快，但数据存储无序，适用于对顺序无要求、追求高效查找的场景。

### （一）核心特性

1. 键值对映射：与 QMap 一致，默认一键一值。
2. 无序存储：数据按哈希表结构存储，顺序不固定。
3. 高效查找：哈希表的查找复杂度为 O (1)，比 QMap 更快。
4. 键要求：**键类型必须提供 `==` 运算符（用于判断相等）和全局 `qHash()` 函数（用于计算哈希值）**。

### （二）常用操作及示例代码

#### **1. 头文件与实例创建**

```cpp
#include <QCoreApplication>
#include <QDebug>
#include <QHash>

int main(int argc, char *argv[]) {
    QCoreApplication a(argc, argv);

    // 创建 QHash 实例：键为 QString 类型，值为 int 类型
    QHash<QString, int> qhash;
```

#### **2. 插入与修改数据**

**支持数组下标法（`[]`）和 `insert()` 函数，重复插入同一键会覆盖原有值**。

```cpp
qhash["key 1"] = 3;
qhash["key 1"] = 8; // 覆盖原有值 3
qhash["key 4"] = 4;
qhash["key 2"] = 2;
qhash.insert("key 3", 30); // insert() 函数插入
```

#### **3. 遍历数据**

支持键列表遍历和 STL 风格迭代器遍历，**顺序不固定**。

```cpp
// 方式 1：通过 keys() 列表遍历
qDebug() << "QHash 数据（键列表遍历）：";
QList<QString> keyList = qhash.keys();
for (int i = 0; i < keyList.length(); i++) {
    qDebug() << keyList[i] << "," << qhash.value(keyList[i]);
}

// 方式 2：STL 风格迭代器遍历
qDebug() << "QHash 数据（STL 迭代器遍历）：";
QHash<QString, int> hash;
hash["key 1"] = 33;
hash["key 2"] = 44;
hash["key 3"] = 55;
hash["key 4"] = 66;
hash.insert("key 3", 100); // 覆盖原有值 55

QHash<QString, int>::const_iterator it;
for (it = hash.begin(); it != hash.end(); it++) {
    qDebug() << it.key() << "-->" << it.value();
}
```

```cpp
    return a.exec();
}
```

### （三）QMap 与 QHash 核心区别

|   特性   |               QMap                |                   QHash                   |
| :------: | :-------------------------------: | :---------------------------------------: |
| 存储顺序 |        按键的自然顺序排列         |            无序（哈希表结构）             |
| 查找速度 |         较慢（O (log n)）         |               较快（O (1)）               |
| 键的要求 | **必须提供 `<` 运算符（排序用）** | **必须提供 `==` 运算符和 `qHash()` 函数** |
| 内存占用 |               较低                |          较高（哈希表冗余空间）           |
| 适用场景 |     需有序遍历、键有序的场景      |     对顺序无要求、追求高效查找的场景      |

## 三、QVector 类：动态数组容器

QVector<T> 是 Qt 中的动态数组容器，**数据存储在连续的内存空间中**，支持快速随机访问，适用于需要频繁按索引读写、数据量动态变化的场景。

### （一）核心特性

1. 连续内存存储：元素在内存中连续排列，随机访问（按索引）速度快（O (1)）。
2. 动态扩容：支持自动扩容，无需手动管理内存大小。
3. 插入 / 删除效率：尾部插入 / 删除效率高（O (1)）；头部或中间插入 / 删除效率低（O (n)，需移动大量元素）。
4. 支持随机访问：可通过下标 `[]` 或 `at(int index)` 快速访问元素。

### （二）常用操作及示例代码

#### **1. 头文件与实例创建**

```cpp
#include <QCoreApplication>
#include <QDebug>
#include <QVector>

int main(int argc, char *argv[]) {
    QCoreApplication a(argc, argv);

    // 创建 QVector 实例：存储 int 类型数据
    QVector<int> qvr;
```

#### **2. 插入数据**

支持两种方式：**流式运算符（`<<`）和 `append()` 函数**（均为**尾部插入**）。

```cpp
// 方式 1：流式运算符 <<
qvr << 10 << 20 << 40 << 30;

// 方式 2：append() 函数（尾部追加）
qvr.append(50);
qvr.append(70);
qvr.append(90);
qvr.append(100);
qvr.append(60);
qvr.append(80);

qDebug() << "插入后的数据：" << qvr << endl;
// 输出：(10, 20, 40, 30, 50, 70, 90, 100, 60, 80)
```

#### 3. 获取元素个数

使用 `count()` 或 `size()` 函数（两者功能一致）。

```cpp
qDebug() << "元素个数：" << qvr.count() << endl; // 输出：10
```

#### **4. 遍历数据**

支持按索引遍历（最常用）或迭代器遍历。

```cpp
qDebug() << "按索引遍历所有元素：";
for (int i = 0; i < qvr.count(); i++) {
    qDebug() << qvr[i]; // 或 qvr.at(i)，at() 更安全（越界会报错）
}
```

#### **5. 删除数据**

使用 `remove()` 函数，支持两种删除方式：

- `remove(int index)`：删除指定索引的单个元素。
- `remove(int index, int count)`：从指定索引开始，删除 count 个元素。

```cpp
// 方式 1：删除索引为 0 的元素（第一个元素）
qDebug() << "\n删除索引 0 后的元素：";
qvr.remove(0);
for (int i = 0; i < qvr.count(); i++) {
    qDebug() << qvr[i]; // 输出：20, 40, 30, 50, 70, 90, 100, 60, 80
}

// 方式 2：从索引 2 开始，删除 3 个元素
qDebug() << "\n从索引 2 开始删除 3 个元素后：";
qvr.remove(2, 3); // 删除索引 2、3、4 的元素（30、50、70）
for (int i = 0; i < qvr.count(); i++) {
    qDebug() << qvr[i]; // 输出：20, 40, 90, 100, 60, 80
}
```

#### 6. 判断是否包含某个元素

使用 `contains(const T &value)` 函数，返回布尔值。

```cpp
qDebug() << "\n包含性判断：";
qDebug() << "是否包含 90：" << qvr.contains(90); // true
qDebug() << "是否包含 901：" << qvr.contains(901); // false
```

```cpp
    return a.exec();
}
```

## 总结

本文详细介绍了 Qt 中三大核心容器类的使用：

1. QMap：有序键值对容器，适用于需按键排序的场景，支持一键多值。
2. QHash：无序哈希容器，查找速度更快，适用于对顺序无要求、追求效率的场景。
3. QVector：动态数组容器，连续内存存储，随机访问高效，适用于按索引操作的场景。

选择容器时，可根据 “是否需要有序”“查找效率要求”“操作类型（插入 / 删除 / 查询）” 来决策：

- 需有序 + 键值对 → QMap
- 无序 + 高效查找 + 键值对 → QHash
- 按索引读写 + 动态扩容 → QVector