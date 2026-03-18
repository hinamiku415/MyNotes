## Qt 链表容器`QList`与 `QLinkedList` 

在 Qt 容器家族中，`QList` 和` QLinkedList` 是针对线性数据存储的核心类，前者兼顾随机访问与灵活操作，后者专注于高效插入删除。本文详细拆解两类容器的存储特性、常用操作、迭代器使用及适用场景，附完整可运行代码。

## 一、`QList` 类：Qt 首选线性容器

`QList<T>`是 Qt 中最常用的线性容器，采用**动态数组 + 灵活存储策略**设计，兼顾随机访问速度与插入删除效率，几乎适用于所有线性数据存储场景，是 Qt 开发的 “万能容器” 首选。

### （一）核心特性

1. **自适应存储策略：**
   - 若存储**指针类型**或**指针大小的基本类型**（如 int、char*），直接存储在内部数组中，内存紧凑。
   - 若存储普通对象，实际存储对象的指针，指针指向堆内存中的实际对象。
2. **支持随机访问**：**可通过下标 `[]` 或 `at()` 函数快速访问元素**（O (1) 复杂度）。
3. 操作高效：尾部插入 / 删除效率高（O (1)），中间插入 / 删除需移动部分元素（效率优于普通数组）。
4. 兼容 STL 风格迭代器：支持读写迭代器与只读迭代器，遍历灵活。

### （二）常用操作及示例代码

#### **1. 头文件与实例创建**

```cpp
#include <QCoreApplication>
#include <QDebug>
#include <QList>

int main(int argc, char *argv[]) {
    QCoreApplication a(argc, argv);

    // 创建 QList 实例：存储 int 类型数据
    QList<int> qlist;
```

#### **2. 插入元素**

支持在指定位置插入（`insert()`）或尾部追加（`append()`），此处演示在列表末尾插入连续数值。

```cpp
// 循环插入 10 个元素：10、11、...、19（插入到列表末尾）
for (int i = 0; i < 10; i++) {
    qlist.insert(qlist.end(), i + 10);
}
qDebug() << "初始插入后的数据：" << qlist;
// 输出：(10, 11, 12, 13, 14, 15, 16, 17, 18, 19)
```

#### **3. 迭代器操作（读写 + 只读）**

- **读写迭代器（`QList<T>::iterator`）**：可遍历并修改元素。
- **只读迭代器（`QList<T>::const_iterator`）**：仅用于遍历，不可修改元素（更安全）。

```cpp
// 1. 读写迭代器：遍历并修改元素（每个元素 ×10 +6）
qDebug() << endl << "读写迭代器修改前：";
QList<int>::iterator writeItr;
for (writeItr = qlist.begin(); writeItr != qlist.end(); writeItr++) {
    qDebug() << *writeItr; // 输出原始值
    *writeItr = (*writeItr) * 10 + 6; // 修改元素
}

// 2. 只读迭代器：遍历修改后的元素
qDebug() << endl << "只读迭代器遍历（修改后）：";
QList<int>::const_iterator readItr;
for (readItr = qlist.constBegin(); readItr != qlist.constEnd(); readItr++) {
    qDebug() << *readItr; // 输出：106, 116, 126, ..., 196
}
```

#### **4. 追加元素**

使用 `append()` 函数在列表尾部添加元素（效率高）。

```cpp
qlist.append(666); // 尾部追加 666
qDebug() << endl << "追加 666 后的数据：";
for (auto it = qlist.begin(); it != qlist.end(); it++) {
    qDebug() << *it; // 最后一个元素为 666
}
```

#### 5. 元素查询

支持按索引查询（`at()`）和包含性判断（`contains()`）。

```cpp
qDebug() << endl << "查询结果：";
qDebug() << "索引 3 对应的元素：" << qlist.at(3); // 按索引查询（第 4 个元素）
qDebug() << "是否包含 77：" << qlist.contains(77); // false（无该元素）
qDebug() << "是否包含 166：" << qlist.contains(166); // true（原 16 → 16×10+6=166）
```

#### 6. 修改元素

使用 `replace(int index, const T &value)` 函数，替换指定索引的元素。

```cpp
qlist.replace(5, 888); // 将索引 5 的元素替换为 888
qDebug() << endl << "替换索引 5 后的数组：" << qlist;
```

#### 7. 删除元素

支持按索引删除（`removeAt()`）、删除第一个元素（`removeFirst()`）等。

```cpp
qlist.removeAt(0); // 删除索引 0 的元素
qlist.removeFirst(); // 删除第一个元素（原索引 1 变为新索引 0）
qlist.removeAt(6); // 删除当前索引 6 的元素
qDebug() << endl << "删除后的数组：" << qlist;
```

```cpp
    return a.exec();
}
```

### （三）关键补充

- **`at(index)` 与 `[]` 的区别**：`at()` 会做越界检查（越界抛出异常），`[]` 不做检查（效率略高），建议优先使用 `at()` 保证安全性。
- 存储对象时：`QList` 存储对象指针，需手动管理对象内存（避免内存泄漏），或使用智能指针（如 `QSharedPointer`）。

## 二、`QLinkedList` 类：高效链表容器

`QLinkedList<T>` 是基于**双向链表**实现的线性容器，数据存储在非连续内存块中，核心优势是大规模数据的插入 / 删除效率，适用于需频繁在中间操作元素的场景。

### （一）核心特性

1. 非连续内存存储：元素通过指针链接，不占用连续内存，无需扩容时移动大量数据。
2. 插入 / 删除高效：任意位置插入 / 删除元素仅需修改指针指向（O (1) 复杂度），适合大规模数据。
3. 不支持随机访问：**不能使用下标 `[]` 或 `at()` 函数**，只能通过迭代器访问元素。
4. 迭代器稳定：插入 / 删除元素时，除被删除元素的迭代器外，其他迭代器仍有效（`QList`迭代器可能失效）。

### （二）常用操作及示例代码

#### **1. 头文件与实例创建**

```cpp
#include <QCoreApplication>
#include <QDebug>
#include <QLinkedList> // 注意头文件：QLinkedList（非 qlinkedlist.h）

int main(int argc, char *argv[]) {
    QCoreApplication a(argc, argv);

    // 创建 QLinkedList 实例：存储 QString 类型数据
    QLinkedList<QString> qAllMonth;
```

#### 2. 插入元素

支持流式运算符（`<<`）尾部追加，也可通过 `insert()` 在指定位置插入。

```cpp
// 循环插入 12 个元素：Month:1 ~ Month:12
for (int i = 1; i <= 12; i++) {
    // arg() 函数拼接字符串：%1 替换为 "Month:", %2 替换为 i
    qAllMonth << QString("%1%2").arg("Month:").arg(i);
}
```

#### 3. 迭代器操作（读写 + 只读）

QLinkedList 仅支持迭代器访问，用法与 QList 迭代器类似。

```cpp
// 1. 读写迭代器：遍历元素（可修改）
qDebug() << "读写迭代器遍历：";
QLinkedList<QString>::iterator writeItr = qAllMonth.begin();
for (; writeItr != qAllMonth.end(); writeItr++) {
    qDebug() << *writeItr; // 输出：Month:1, Month:2, ..., Month:12
    // 若需修改：*writeItr = "修改后的值";
}

// 2. 只读迭代器：遍历元素（不可修改）
qDebug() << endl << "只读迭代器遍历：";
QLinkedList<QString>::const_iterator readItr = qAllMonth.constBegin();
for (; readItr != qAllMonth.constEnd(); readItr++) {
    qDebug() << *readItr; // 输出结果与上面一致
}
```

```cpp
    return a.exec();
}
```

### (三）关键注意事项

- 头文件说明：Qt 中标准头文件为 `<QLinkedList>`，而非文档中提及的 `<qlinkedlist.h>`，实际开发需使用标准头文件。
- 不支持随机访问：若尝试用 `qAllMonth[0]` 或 `qAllMonth.at(0)` 访问元素，会编译报错。
- 适用场景：仅当需要频繁在列表中间插入 / 删除大量数据时使用，否则优先选择 QList（随机访问更便捷）

## 三、STL 风格迭代器对照表

`QList` 和 `QLinkedList` 均支持 STL 风格迭代器，迭代器类型与容器的对应关系如下：

|         容器类          |           只读迭代器类           |        读写迭代器类        |
| :---------------------: | :------------------------------: | :------------------------: |
| `QList<T>`、`QQueue<T>` |    `QList<T>::const_iterator`    |    `QList<T>::iterator`    |
|    `QLinkedList<T>`     | `QLinkedList<T>::const_iterator` | `QLinkedList<T>::iterator` |

### 迭代器使用要点

1. 读写迭代器（iterator）：可通过 `*it` 修改元素，仅在需要修改时使用。
2. 只读迭代器（const_iterator）：不可修改元素，遍历速度更快、更安全，优先使用。
3. **迭代器失效：**
   - **`QList`：插入 / 删除中间元素时，该元素后的迭代器可能失效，需重新获取。**
   - **`QLinkedList`：仅被删除元素的迭代器失效，其他迭代器保持有效。**



## 四、`QList` 与 `QLinkedList`核心区别与选型建议

|      特性       |              `QList`               |          `QLinkedList`           |
| :-------------: | :--------------------------------: | :------------------------------: |
|    存储方式     |     连续内存（数组）或指针数组     |      非连续内存（双向链表）      |
|    随机访问     |     支持（`[]`/`at()`，O(1)）      |    不支持（仅迭代器，O (n)）     |
| 插入 / 删除效率 | 尾部 O (1)，中间 O (n)（移动元素） |    任意位置 O (1)（修改指针）    |
|  迭代器稳定性   |     中间插入 / 删除后可能失效      |      仅删除元素的迭代器失效      |
|    内存占用     |    较低（连续存储，无指针开销）    |  较高（每个元素需额外存储指针）  |
|    遍历速度     |          较快（缓存友好）          | 较慢（内存不连续，缓存命中率低） |

### 选型建议

1. 优先使用` QList` 的场景：
   - 需要随机访问元素（按索引读写）。
   - 插入 / 删除操作主要在列表尾部。
   - 数据量不大，或对内存占用敏感。
2. 选择` QLinkedList `的场景：
   - 需频繁在列表中间插入 / 删除大量数据。
   - 数据量极大，避免 `QList`扩容时的内存移动开销。
   - 依赖稳定的迭代器（插入 / 删除后无需重新获取迭代器）。

## 总结

`QList` 和 `QLinkedList` 是 Qt 线性容器的核心代表，二者各有侧重：

- `QList` 兼顾灵活性与效率，支持随机访问，是大多数场景的首选。
- `QLinkedList` 专注于高效插入删除，适合大规模数据的中间操作，但不支持随机访问。

掌握两类容器的存储特性与操作方法，结合实际场景（是否需要随机访问、插入删除位置）选择合适的容器，能显著提升 Qt 程序的性能与可读性。后续可进一步学习 `QQueue`（队列）、`QStack`（栈）等基于 `QList` 实现的容器，拓展线性数据处理能力。





## 补充：迭代器失效

### 一、迭代器失效的核心含义

迭代器可以理解为**指向容器中某个元素的 “指针”**，迭代器失效就是指：这个 “指针” 原本指向的内存位置已经无效，继续使用这个迭代器会导致程序崩溃、数据错乱或未定义行为（比如遍历到错误数据、程序闪退）。

简单类比：你手里有一张写着朋友住址的纸条（迭代器），但朋友搬家了（容器底层结构变化），你还按纸条上的地址找他，自然找不到（迭代器失效）。

### 二、迭代器失效的核心原因

迭代器失效的本质是：**容器的底层内存布局发生了改变**，导致迭代器指向的内存地址不再对应原来的元素，具体分两类场景：

### 1. 内存重新分配（以 `QList` 为例）

`QList `基于动态数组实现，当插入元素导致容器容量不足时，Qt 会重新申请一块更大的内存，把原有元素全部拷贝过去，然后释放旧内存。

- 原迭代器指向的是**旧内存地址**，旧内存已被释放，此时迭代器就失效了。

- 示例场景：

  ```cpp
  QList<int> list{1,2,3};
  QList<int>::iterator it = list.begin(); // 指向第一个元素 1
  // 插入大量元素，触发 QList 扩容（内存重新分配）
  for(int i=0; i<1000; i++) {
      list.insert(list.end(), i);
  }
  qDebug() << *it; // 迭代器失效！访问已释放的内存，程序可能崩溃
  ```

### 2. 元素位置移动（以` QList `中间插入 / 删除为例）

`QList `是连续内存存储，在中间插入 / 删除元素时，后续元素会整体前移 / 后移，导致指向这些元素的迭代器指向错误位置。

- 示例场景：

  ```cpp
  QList<int> list{10,20,30,40};
  QList<int>::iterator it = list.begin() + 2; // 指向 30
  list.removeAt(1); // 删除索引 1 的元素 20，后续元素 30、40 前移
  qDebug() << *it; // 原本指向 30，现在指向 40（迭代器失效，指向错误元素）
  ```

### 3. 元素被直接删除

如果迭代器指向的元素被直接删除，这个迭代器就会变成 “野指针”，直接使用必然崩溃。

- 示例场景：

  ```cpp
  QList<int> list{1,2,3};
  QList<int>::iterator it = list.begin() + 1; // 指向 2
  list.erase(it); // 删除迭代器指向的元素 2
  qDebug() << *it; // 迭代器失效！访问已删除的元素，程序崩溃
  ```

### 三、Qt 同容器的迭代器失效场景对比

不同容器的底层结构不同，迭代器失效的规则也不一样，这也是之前区分 `QList` 和 `QLinkedList` 的核心点之一：

|   容器类型    |                        迭代器失效场景                        |         迭代器稳定性         |
| :-----------: | :----------------------------------------------------------: | :--------------------------: |
|    `QList`    | 1. 插入元素触发扩容；2. 中间插入 / 删除元素，该位置后的迭代器失效；3. 删除迭代器指向的元素 |           稳定性差           |
| `QLinkedList` | 仅删除迭代器指向的元素时，该迭代器失效；其他插入 / 删除操作不会导致任何迭代器失效 | 稳定性好（推荐频繁修改场景） |
| `QMap/QHash`  | 1. 删除迭代器指向的元素时，该迭代器失效；2. 插入元素不会导致迭代器失效（有序 / 哈希结构不移动其他元素） |          稳定性较好          |

### 四、如何避免迭代器失效？

针对不同场景，有 3 种核心解决方案，从易到难推荐：

### 方案 1：重新获取迭代器（最简单）

在容器结构变化后，**放弃旧迭代器，重新获取新的迭代器**，适用于所有容器。

```cpp
QList<int> list{10,20,30,40};
QList<int>::iterator it = list.begin() + 2; // 指向 30
list.removeAt(1); // 删除 20，迭代器失效
it = list.begin() + 2; // 重新获取迭代器（现在指向 40）
qDebug() << *it; // 安全，输出 40
```

### 方案 2：使用 `erase ()`的返回值（删除元素时专用）

Qt 容器的 `erase()` 函数会返回**指向被删除元素下一个位置的有效迭代器**，用这个返回值更新迭代器，避免失效。

```cpp
QList<int> list{1,2,3,4,5};
QList<int>::iterator it = list.begin();
while(it != list.end()) {
    if(*it % 2 == 0) { // 删除偶数元素
        // erase 返回下一个有效迭代器，直接赋值给 it
        it = list.erase(it);
    } else {
        it++; // 非偶数，迭代器正常后移
    }
}
// 最终 list：{1,3,5}，全程无迭代器失效
qDebug() << list;
```

### 方案 3：选择迭代器稳定的容器（从根源避免）

如果业务中需要频繁插入 / 删除元素，**优先选择 `QLinkedList` 而非 `QList`**——`QLinkedList`基于双向链表实现，插入 / 删除仅修改指针，除了被删除元素的迭代器，其他迭代器永远有效。

```cpp
QLinkedList<int> list{10,20,30,40};
QLinkedList<int>::iterator it = list.begin() + 2; // 指向 30
list.removeAt(1); // 删除 20，QLinkedList 迭代器不失效
qDebug() << *it; // 安全，仍指向 30
```

### 总结

1. 迭代器失效的本质是**容器底层内存布局变化**，导致迭代器指向的内存无效；
2. `QList` 迭代器稳定性差（扩容 / 中间修改易失效），`QLinkedList` 稳定性好（仅删除元素的迭代器失效）；
3. 避免失效的核心方法：重新获取迭代器、使用 `erase ()` 返回值、选择稳定的容器（如 `QLinkedList`）。