### Qt 布局管理：QLayout 类与 QStackedWidget 类核心整理

在 Qt 开发中，布局管理器（QLayout 相关类）是实现控件有序排列、界面自适应的核心工具，QStackedWidget 则专注于多页面切换场景。

## 一、QLayout 类：基础布局管理体系

QLayout 是所有布局管理器的基类，定义了布局管理的基础接口，Qt 提供多个实用子类，支持多种排列方式，核心优势是自动适配窗口大小变化，无需手动设置控件位置。

### 1. 类继承关系

![image](https://p9-flow-imagex-sign.byteimg.com/tos-cn-i-a9rns2rl98/rc_gen_image/848cd84cb63f4ba9a39de87e69969f69.jpeg~tplv-a9rns2rl98-downsize_watermark_1_6_b.png?lk3s=8e244e95&rcl=202601261945424A5CEBE323B03E162D57&rrcfp=827586d3&x-expires=2084787964&x-signature=idSBnQFXHHxvjCCA1b1%2FEYOYMLI%3D)

```json
QLayout（基类）
├─ QGridLayout（网格布局）
├─ QBoxLayout（盒子布局，抽象类）
│  ├─ QHBoxLayout（水平布局）
│  └─ QVBoxLayout（垂直布局）
└─ QStackedLayout（堆栈布局）
```

- 所有布局类均依赖 `QT += widgets` 模块（需在 pro 文件中配置）；
- QBoxLayout 为抽象类，无法直接实例化，需通过其子类 QHBoxLayout、QVBoxLayout 使用。

### 2. 核心布局类详情

|   布局类名称   |                           核心描述                           |           头文件            |  依赖模块配置   |
| :------------: | :----------------------------------------------------------: | :-------------------------: | :-------------: |
|    QLayout     | 布局基类，定义添加控件、设置边距 / 间距等基础接口，不可直接使用 |    `#include <QLayout>`     | `QT += widgets` |
|   QBoxLayout   | 抽象基类，**提供水平 / 垂直排列控件的基础逻辑**，支持控件拉伸比例设置 |   `#include <QBoxLayout>`   | `QT += widgets` |
|  QHBoxLayout   | **水平布局子类**，沿水平方向（左右）排列子控件，支持对齐方式、间距设置 |  `#include <QHBoxLayout>`   | `QT += widgets` |
|  QVBoxLayout   | **垂直布局子类**，沿垂直方向（上下）排列子控件，功能与 QHBoxLayout 对称 |  `#include <QVBoxLayout>`   | `QT += widgets` |
|  QGridLayout   | 网格布局，将界面划分为多行多列网格，控件可占据单个或多个网格单元 |  `#include <QGridLayout>`   | `QT += widgets` |
| QStackedLayout | 堆栈布局，多个控件堆叠排列，同一时间仅显示一个控件，支持切换功能 | `#include <QStackedLayout>` | `QT += widgets` |

### 案例代码：

**头文件：`dialog.h`**

```cpp
#ifndef DIALOG_H
#define DIALOG_H

#include <QDialog>

#include <QLabel>
#include <QLineEdit>
#include <QComboBox>
#include <QTextEdit>
#include <QGridLayout>
#include <QPushButton>

class Dialog : public QDialog
{
    Q_OBJECT

public:
    Dialog(QWidget *parent = nullptr);
    ~Dialog();

private:
    // 1:左边 网格布局、表格布局
    QGridLayout *lLayout;


    QLabel *UserNumber;
    QLineEdit *UserNumberLineEdit;

    QLabel *UserName;
    QLineEdit *UserNameLineEdit;

    QLabel *UserSex;
    QComboBox *UserSexCombobox;

    QLabel *UserDepart;
    QTextEdit *UserDepartTextEdit;

    QLabel *UserAge;
    QLineEdit *UserAgeLineEdit;


    // 2:右边 水平布局
    QHBoxLayout *toprightlayout;
    QVBoxLayout *rightlayout;

    QLabel *MyselfInfo;
    QTextEdit *MyselfInfoTextEdit;

    // 3:右边底部
    QPushButton *okbutton,*cancelbutton;
    QHBoxLayout *buttomLayout;
};
#endif // DIALOG_H
```

**代码：`dialog.cpp`**

```cpp
#include "dialog.h"

Dialog::Dialog(QWidget *parent)
    : QDialog(parent)
{
    setWindowTitle("员工信息");

    // 左边控件
    UserNumber=new QLabel("员工编号：");
    UserNumberLineEdit=new QLineEdit;

    UserName=new QLabel("员工姓名：");
    UserNameLineEdit=new QLineEdit;

    UserSex=new QLabel("员工性别：");
    UserSexCombobox=new QComboBox;
    UserSexCombobox->addItem("男");
    UserSexCombobox->addItem("女");

    UserDepart=new QLabel("所在部门：");
    UserDepartTextEdit=new QTextEdit;

    UserAge=new QLabel("员工年龄：");
    UserAgeLineEdit=new QLineEdit;


    // 网格布局
    //把上面设置的信息加入网格
    lLayout=new QGridLayout();

    lLayout->addWidget(UserNumber,0,0); // // 员工编号
    lLayout->addWidget(UserNumberLineEdit,0,1);

    lLayout->addWidget(UserName,1,0);
    lLayout->addWidget(UserNameLineEdit,1,1);

    lLayout->addWidget(UserSex,2,0);
    lLayout->addWidget(UserSexCombobox,2,1);

    lLayout->addWidget(UserDepart,3,0);
    lLayout->addWidget(UserDepartTextEdit,3,1);

    lLayout->addWidget(UserAge,4,0);
    lLayout->addWidget(UserAgeLineEdit,4,1);

    lLayout->setColumnStretch(0,1);
    lLayout->setColumnStretch(1,3);


    // 右边上部分
    toprightlayout=new QHBoxLayout();
    toprightlayout->setSpacing(25);

    MyselfInfo=new QLabel("个人简历:");
    MyselfInfoTextEdit=new QTextEdit;


    rightlayout=new QVBoxLayout();
    rightlayout->addLayout(toprightlayout);
    rightlayout->addWidget(MyselfInfo);
    rightlayout->addWidget(MyselfInfoTextEdit);


    // 右边下部分
    okbutton=new QPushButton("确认");
    cancelbutton=new QPushButton("退出");

    buttomLayout=new QHBoxLayout();
    buttomLayout->addStretch();
    buttomLayout->addWidget(okbutton);
    buttomLayout->addWidget(cancelbutton);

    //总布局，主布局
    QGridLayout *mlayout=new QGridLayout(this); 
    mlayout->setMargin(20);
    mlayout->setSpacing(10);
    mlayout->addLayout(lLayout,0,0); // 左边
    mlayout->addLayout(rightlayout,0,1); // 右上
    mlayout->addLayout(buttomLayout,1,0,1,2);

    mlayout->setSizeConstraint(QLayout::SetFixedSize); //设置布局的尺寸约束规则

}

Dialog::~Dialog()
{}
```

<img src="C:\Users\张超然\AppData\Roaming\Typora\typora-user-images\image-20260126195937564.png" alt="image-20260126195937564" style="zoom:67%;" />

------

## 二、QStackedWidget 类：堆栈式窗体控件

QStackedWidget 是容器型控件，提供**堆栈式空间存放多个子控件（页面），同一时间仅显示一个控件，核心用于多页面切换场景。**

### 1. 类继承关系

```json
QWidget（控件基类）
└─ QFrame（带边框控件基类）
   └─ QStackedWidget（堆栈窗体）
```

- 继承自 QFrame，支持设置边框样式；
- 依赖 `QT += widgets` 模块，需包含对应头文件。

### 2. 核心详情

|     类名称     |                           核心描述                           |           头文件            |  依赖模块配置   |
| :------------: | :----------------------------------------------------------: | :-------------------------: | :-------------: |
| QStackedWidget | 堆栈式容器控件，可存放多个子控件（页面），同一时间仅显示一个，支持页面切换 | `#include <QStackedWidget>` | `QT += widgets` |
|     QFrame     |        QStackedWidget 的父类，为子类提供边框显示功能         |     `#include <QFrame>`     | `QT += widgets` |

### 3. 核心特性

- 页面管理：支持添加、插入、移除子控件（页面）；
- 页面切换：可通过索引或控件对象直接切换显示的页面；
- 信号支持：提供 `currentChanged(int index)` 信号，切换页面时触发，便于联动更新界面状态。

### 案例代码：

一个可以实现页面切换的小实例

**头文件：`dialog.h`**

```cpp
#ifndef STACKEDDLG_H
#define STACKEDDLG_H

#include <QDialog>

#include <QListWidget>
#include <QStackedWidget>
#include <QLabel>


class stackedDlg : public QDialog
{
    Q_OBJECT

public:
    stackedDlg(QWidget *parent = nullptr);
    ~stackedDlg();


public:
    QStackedWidget *statcks;
    QListWidget *qlist; // 列表框控件创建

    QLabel *lab1,*lab2,*lab3,*lab4,*lab5;

};
#endif // STACKEDDLG_H
```

**代码：`dialog.cpp`**

```cpp
#include "stackeddlg.h"

#include <QHBoxLayout>


stackedDlg::stackedDlg(QWidget *parent)
    : QDialog(parent)
{

    setWindowTitle("堆栈窗体测试");

    qlist=new QListWidget(this);
    qlist->insertItem(0,"Linux1");
    qlist->insertItem(1,"Linux2");
    qlist->insertItem(2,"Linux3");
    qlist->insertItem(3,"Linux4");
    qlist->insertItem(4,"Linux5");

    lab1=new QLabel("Linux1 Qt");
    lab2=new QLabel("Linux2 Qt");
    lab3=new QLabel("Linux3 Qt");
    lab4=new QLabel("Linux4 Qt");
    lab5=new QLabel("Linux5 Qt");

    statcks=new QStackedWidget(this);  // 实例化堆栈窗体
    statcks->addWidget(lab1);
    statcks->addWidget(lab2);
    statcks->addWidget(lab3);
    statcks->addWidget(lab4);
    statcks->addWidget(lab5);


    QHBoxLayout *mlayout=new QHBoxLayout(this);

    mlayout->setMargin(10);
    mlayout->setSpacing(20);

    mlayout->addWidget(qlist);
    mlayout->addWidget(statcks,0,Qt::AlignCenter);
    mlayout->setStretchFactor(qlist,1);
    mlayout->setStretchFactor(statcks,5);
     
    connect(qlist, &QListWidget::currentRowChanged, this, [=](int row) {
        statcks->setCurrentIndex(row); // 切换堆栈窗体的当前页面索引，与列表选中行索引一致
    });     
}
stackedDlg::~stackedDlg()
{}
```

<img src="C:\Users\张超然\AppData\Roaming\Typora\typora-user-images\image-20260126213258301.png" alt="image-20260126213258301" style="zoom: 67%;" />

------

## 三、关键补充

1. 布局类使用逻辑：实例化布局 → 添加控件 / 嵌套布局 → 设置布局属性（边距、间距等）→ 绑定到窗口 / 容器控件；
2. QStackedWidget 与 QStackedLayout 区别：
   - QStackedWidget 是容器控件，自带窗口特性，支持边框设置；
   - QStackedLayout 是布局管理器，需依附于其他控件使用，仅负责页面排列，无边框功能；
3. 搭配使用：通常在 QStackedWidget 的子页面中，嵌套 QHBoxLayout、QVBoxLayout 等布局，实现页面内控件自适应排列。