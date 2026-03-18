## Qt Model-View-Delegate 框架：数据与界面的解耦

在 Qt 开发中，处理列表、表格、树形结构等数据展示与交互时，Model-View-Delegate（模型 - 视图 - 代理）框架是当之无愧的核心工具。它通过**数据与界面分离**的设计理念，让代码结构更清晰、维护更高效，今天就带大家全面拆解这个框架的核心逻辑与实战要点。

## 一、框架核心思想：为什么需要 Model-View-Delegate？

传统数据展示方式中，数据存储与界面渲染往往耦合在一起（比如直接在 `QWidget` 中绘制数据），导致：

- 数据修改后，界面无法自动更新；
- 同一组数据想换不同样式展示（比如列表→表格），需要重写大量代码；
- 编辑数据时，界面逻辑与数据处理混为一谈，维护困难。

而 Model-View-Delegate 框架通过**三组件分工协作**，完美解决这些问题：

- 数据与界面分离：模型管数据，视图管展示，代理管编辑；
- 灵活扩展：同一模型可搭配多个视图，同一视图可适配不同模型；
- 响应式更新：数据变化自动通知视图，视图操作通过代理同步给模型。

## 二、核心三组件：各司其职的 "铁三角"

框架的核心是 Model（模型）、View（视图）、Delegate（代理）三个组件，它们分工明确、通过信号槽联动。

![image-20260204184927213](C:\Users\张超然\AppData\Roaming\Typora\typora-user-images\image-20260204184927213.png)

### 1. Model（模型）：数据的 "管理者"

模型是数据的抽象表示，负责**存储数据、与数据源通信、提供统一的数据访问接口**，不关心数据如何展示。

- 基类体系：所有模型都继承自`QAbstractItemModel`（抽象基类），Qt 提供了三个常用子类：

  - **`QAbstractListModel`：适用于一维列表数据（如通讯录列表）；**
  - **`QAbstractTableModel`：适用于二维表格数据（如员工信息表）；**
  - **`QAbstractProxyModel`：代理模型，用于对已有模型的数据进行过滤、排序等二次处理。**

- 核心职责：

  1. 存储数据（或对接数据库、文件等数据源）；
  2. 提供接口让视图获取数据（如行 / 列数、单元格数据）；
  3. 数据变化时发出信号（如`dataChanged`），通知视图更新；
  4. 接收代理传来的编辑数据，同步到数据源。

  

### 2. View（视图）：数据的 "展示者"

视图负责**将模型中的数据以可视化形式呈现**，不关心数据的存储和来源，只通过模型提供的接口获取数据。

- 基类体系：所有视图都继承自`QAbstractItemView`（抽象基类），Qt 提供了常用现成视图：

  - `QListView`：列表视图（一维数据展示，如文件列表）；
  - `QTableView`：表格视图（二维数据展示，如 Excel 表格）；
  - `QTreeView`：树形视图（层级数据展示，如文件夹结构）；
  - `QColumnView`：列视图（多列层级展示）。

- 核心职责：

  1. 从模型中获取数据（通过`ModelIndex`模型索引定位数据）；
  2. 将数据渲染到界面（如表格单元格、列表项）；
  3. 捕捉用户操作（如点击、滚动），并传递给代理处理；
  4. 接收模型的更新信号，自动刷新界面。

  

### 3. Delegate（代理）：数据的 "编辑者"

代理是连接视图与模型的 "中间人"，负责**数据的编辑和自定义渲染**，是框架中灵活性最高的组件。

- 基类体系：所有代理都继承自`QAbstractItemDelegate`（抽象基类），Qt 提供两个常用子类：

  - `QItemDelegate`：基础代理，支持默认的数据编辑（如文本输入）；
  - `QStyledItemDelegate`：带样式的代理，与 Qt 样式表（QSS）兼容，推荐使用。

- 核心职责：

  1. 自定义数据渲染（如在表格单元格中显示图片、进度条）；
  2. 提供编辑控件（如在单元格中嵌入下拉框、复选框）；
  3. 处理编辑操作，将结果同步给模型；
  4. 通知视图编辑状态（如开始编辑、结束编辑）。

  

## 三、三组件联动机制：数据如何流转？

三个组件通过信号槽实现联动，核心流转逻辑如下：

1. 数据更新流程：

   **数据源变化 → 模型（Model）感知 → 发出`dataChanged`信号 → 视图（View）接收信号 → 刷新界面展示。**

2. 用户操作流程：

   用户点击视图（View）→ 视图通知代理（Delegate）→ 代理创建编辑控件 → 用户编辑数据 → 代理将新数据同步给模型（Model）→ 模型更新数据源。

## 四、实战

### 代码实例1

```cpp
#include "mainwindow.h"
#include <QApplication>

// 导入Model-View核心头文件
#include <QAbstractItemModel>   // 所有模型的抽象基类
#include <QAbstractItemView>    // 所有视图的抽象基类
#include <QItemSelectionModel>  // 选择模型（管理视图选中状态）

// 导入分割器、目录模型、三种视图的头文件
#include <QSplitter>    // 分割器（让多个控件可拖拽调整大小）
#include <QDirModel>    // 磁盘目录模型（读取文件/文件夹结构，Qt5中已被QFileSystemModel替代，但功能一致）
#include <QTreeView>    // 树形视图（适合展示层级结构，如文件夹树）
#include <QListView>    // 列表视图（适合单列展示文件/文件夹）
#include <QTableView>   // 表格视图（适合多列展示文件属性：名称、大小、类型等）

int main(int argc, char *argv[])
{
    // 1. 创建Qt应用程序实例（必须有，管理事件循环）
    QApplication a(argc, argv);
    
    // 注释掉主窗口相关代码，改用自定义的文件浏览器界面
    // MainWindow w;
    // w.show();

    // 2. 创建核心组件：模型 + 三种视图
    QDirModel model; // 核心：目录模型，自动读取电脑磁盘的文件/文件夹结构（无需手动存数据）
    QTreeView tree;  // 树形视图：展示文件夹的层级结构（如C盘→Users→桌面）
    QListView list;  // 列表视图：单列展示文件/文件夹名称
    QTableView table;// 表格视图：多列展示文件详情（名称、大小、修改时间等）

    // 3. 关键：三个视图绑定同一个模型（解耦的核心体现）
    // 所有视图都从model中获取文件数据，模型变则所有视图同步变
    tree.setModel(&model);
    list.setModel(&model);
    table.setModel(&model);

    // 4. 设置选择模式 + 同步选中状态
    // 4.1 树形视图设置为“多选模式”（可按住Ctrl选多个文件/文件夹）
    tree.setSelectionMode(QAbstractItemView::MultiSelection);
    
    // 4.2 列表/表格视图复用树形视图的选择模型（QItemSelectionModel）
    // 作用：树形视图选中某个文件，列表/表格视图会同步选中该文件，实现“一处选择、多处同步”
    list.setSelectionModel(tree.selectionModel());
    table.setSelectionModel(tree.selectionModel()); // 你代码里写了两次list，这里应该是table，我帮你修正

    // 5. 绑定双击事件：树形视图双击文件夹，列表/表格视图跳转到该文件夹
    // 5.1 树形视图双击某个目录索引 → 列表视图设置根索引为该目录（显示该目录下的内容）
    QObject::connect(&tree,SIGNAL(doubleClicked(QModelIndex)),&list,SLOT(setRootIndex(QModelIndex)));
    // 5.2 同理，表格视图也同步跳转到该目录
     	      QObject::connect(&tree,SIGNAL(doubleClicked(QModelIndex)),&table,SLOT(setRootIndex(QModelIndex)));

    // 6. 创建分割器，把三个视图放在同一个窗口，支持拖拽调整宽度
    QSplitter *qsp=new QSplitter; // 分割器：让多个控件横向/纵向排列，可拖拽调整大小
    qsp->addWidget(&tree);   // 第一个控件：树形视图
    qsp->addWidget(&list);   // 第二个控件：列表视图
    qsp->addWidget(&table);  // 第三个控件：表格视图

    // 7. 显示窗口 + 设置标题
    qsp->show();
    qsp->setWindowTitle("模型（Model）--测试操作");

    // 8. 进入应用程序事件循环（等待用户操作：点击、双击、关闭等）
    return a.exec();
}
```

<img src="C:\Users\张超然\AppData\Roaming\Typora\typora-user-images\image-20260204175402397.png" alt="image-20260204175402397" style="zoom:67%;" />

### 代码实战2

#### 头文件（`modelextended.h`）关键内容

```cpp
// 继承QAbstractTableModel，具备表格模型的基础能力
class ModelExtended : public QAbstractTableModel
{
    Q_OBJECT
public:
    explicit ModelExtended(QObject *parent = 0);

    // 重写Qt模型的核心虚函数
    virtual int rowCount(const QModelIndex &parent=QModelIndex()) const; // 行数
    virtual int columnCount(const QModelIndex &parent=QModelIndex()) const; // 列数
    QVariant data(const QModelIndex &index,int role) const; // 单元格数据
    QVariant headerData(int section,Qt::Orientation orientateion,int role) const; // 表头数据

private:
    // 存储数据的容器
    QVector<short> empindex;          // 员工编号索引（行对应关系）
    QVector<short> empnameindex;      // 员工姓名索引（行对应关系）
    QMap<short,QString> empno;       // 员工编号（键：索引，值：编号）
    QMap<short,QString> empname;     // 员工姓名（键：索引，值：姓名）
    QStringList viewlisttitle;       // 表头标题（员工编号/姓名/部门）
    QStringList empdepartment;       // 员工部门列表

    void ModelFunc(); // 初始化数据的辅助函数
};
```

#### 2. 实现文件（`modelextended.cpp`）核心逻辑

##### （1）构造函数 + 数据初始化

```cpp
ModelExtended::ModelExtended(QObject *parent) :
    QAbstractTableModel(parent)
{
    // 初始化员工编号（键值对：1→2022001，2→2022002...）
    empno[1]="2022001";
    empno[2]="2022002";
    empno[3]="2022003";
    empno[4]="2022004";
    empno[5]="2022005";

    // 初始化员工姓名（键值对：1→张三，2→李四...）
    empname[1]="张三";
    empname[2]="李四";
    empname[3]="王五";
    empname[4]="刘山";
    empname[5]="张平";

    ModelFunc(); // 调用辅助函数初始化其他数据
}

void ModelExtended::ModelFunc()
{
    // 设置表头标题
    viewlisttitle<<"员工编号"<<"员工姓名"<<"所在部门";
    // 初始化行索引（1-5，对应5行数据）
    empindex<<1<<2<<3<<4<<5;
    empnameindex<<1<<2<<3<<4<<5;
    // 初始化部门列表（按行对应：第0行→营销部，第1行→账务部...）
    empdepartment<<"营销部"<<"账务部"<<"研发部"<<"董事会"<<"后勤部";
}
```

##### （2）重写行 / 列数函数（告诉视图表格的行列规模）

```cpp
// 返回行数：empindex的大小（5行）
int ModelExtended::rowCount(const QModelIndex &parent) const
{
    return empindex.size();
}

// 返回列数：固定3列（编号、姓名、部门）
int ModelExtended::columnCount(const QModelIndex &parent) const
{
    return 3;
}
```

##### （3）重写 data 函数（核心：给视图提供单元格数据）

```cpp
QVariant ModelExtended::data(const QModelIndex &index,int role) const
{
    // 索引无效时返回空值
    if(!index.isValid())
        return QVariant();

    // 仅处理"显示角色"（Qt::DisplayRole，即单元格文本展示）
    if(role==Qt::DisplayRole)
    {
        switch (index.column()) {
        case 0: // 第0列：员工编号
            return empno[empindex[index.row()]]; // 按行索引取编号（如第0行→empindex[0]=1→empno[1]=2022001）
            break;
        case 1: // 第1列：员工姓名
            return empname[empnameindex[index.row()]]; // 同理，按行索引取姓名
            break;
        case 2: // 第2列：所在部门
            return empdepartment[index.row()]; // 直接按行索引取部门（如第0行→营销部）
            break;
        default:
            return QVariant();
        }
    }
    return QVariant();
}
```

##### （4）重写 headerData 函数（设置表头文本）

```cpp
QVariant ModelExtended::headerData(int section,Qt::Orientation orientation,int role) const
{
    // 仅处理"水平表头+显示角色"
    if(role==Qt::DisplayRole && orientation==Qt::Horizontal)
        return viewlisttitle[section]; // section是列索引：0→员工编号，1→员工姓名，2→所在部门

    // 其他情况调用父类默认实现
    return QAbstractTableModel::headerData(section,orientation,role);
}
```

#### 程序入口（main.cpp）：展示表格视图

```cpp
#include "mainwindow.h"
#include <QApplication>
#include "modelextended.h"
#include <QTableView>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);

    // 1. 创建自定义模型对象
    ModelExtended modelExts;

    // 2. 创建表格视图（QTableView是Qt自带的表格控件）
    QTableView view;
    // 3. 将模型绑定到视图（MVC模式：模型存数据，视图展示数据）
    view.setModel(&modelExts);
    // 4. 设置窗口标题、大小
    view.setWindowTitle("ModelExtended模型扩展--测试操作");
    view.resize(500,300);
    // 5. 显示视图
    view.show();

    return a.exec(); // 进入Qt事件循环
}
```

<img src="C:\Users\张超然\AppData\Roaming\Typora\typora-user-images\image-20260204183923640.png" alt="image-20260204183923640" style="zoom:67%;" />

**MVC 设计模式**：Qt 的 Model/View 架构中，`ModelExtended`（模型）负责存储和提供数据，`QTableView`（视图）负责展示数据，两者解耦（修改数据无需改视图，修改视图无需改数据）。

**核心虚函数重写**：

- `rowCount`/`columnCount`：告诉视图表格的行列数；
- `data`：视图渲染时会调用该函数，获取每个单元格的展示数据；
- `headerData`：设置表头的文本内容。

**Qt 容器的使用**：

- `QMap`：键值对存储（适合按索引快速查找员工编号 / 姓名）；
- `QVector`：有序存储行索引；
- `QStringList`：简单的字符串列表（适合存储表头、部门等有序文本）。



## 总结

Model-View-Delegate 框架是 Qt 中处理结构化数据的核心方案，其核心在于**数据、展示、编辑的分离设计**。通过模型管理数据、视图负责展示、代理处理编辑，让代码结构更清晰、扩展更灵活。

掌握这个框架后，你可以轻松实现复杂的数据交互功能，无论是简单的列表展示，还是复杂的表格编辑、树形层级展示，都能游刃有余.。
