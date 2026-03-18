## Qt 布局进阶：QSplitter 与 QDockWidget 类核心整理

在 Qt 开发中，QSplitter（窗口分割）和 QDockWidget（停靠窗口）是两类提升界面灵活性的关键控件，分别适配 “可调整分割区域” 和 “可停靠附属窗口” 的场景。本文基于零声教育相关课程内容，提炼两类控件的核心特性、功能及使用逻辑，方便开发时快速选型。

## 一、QSplitter 类：可拖动分割的窗口布局

QSplitter 是用于分割界面的容器控件，核心优势是支持用户手动调整子区域大小，比固定布局更具交互性。

<img src="D:\DownLoad\文件信息 (1).png" alt="文件信息 (1)" style="zoom: 25%;" />

### 1. 核心基础信息

- **类继承关系**：派生于 QFrame（与 QStackedWidget 同源），支持设置边框样式；
- **核心功能**：**按水平或垂直方向排列多个控件 / 布局，控件间添加可拖动分割线，拖动时子控件实时适配大小**；
- **依赖配置**：需包含头文件 `#include <QSplitter>`，依赖 `QT += widgets` 模块。
- **不需要显式调用添加函数，只要在创建控件时把 `QSplitter` 设为父对象，控件就会自动成为拆分器的子项**

### 2. 关键特性

- 支持两种分割方向：**水平（`Qt::Horizontal`）、垂直（`Qt::Vertical`）**；
- 可通过 API 设置子控件初始大小比例，支持嵌套使用（如水平分割中嵌套垂直分割）；
- 分割线拖动无需手动编写尺寸变化逻辑，Qt 自动处理子控件适配。

### 3. 适用场景

- 多区域联动展示界面（如代码编辑器：左侧文件树 + 中间编辑区 + 右侧控制台）；
- 用户需自定义空间分配的场景（如数据可视化工具：左侧参数面板 + 右侧图表区）；
- 替代固定比例布局，提升界面交互灵活性。

### 代码案例

**代码`mainwindow.cpp`**

```cpp
#include "mainwindow.h"

#include <QSplitter>
#include <QTextEdit>


MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{
    // 1：拆分窗口（分割窗口、分裂窗口布局）
    //最外层水平拆分器（Qt::Horizontal 表示水平方向拆分，父对象为0即无父）
    QSplitter *spMainWindow=new QSplitter(Qt::Horizontal,0);
    QTextEdit *txteditmain=new QTextEdit("左边主窗口",spMainWindow);

    // 2：拆分右边部分窗口
    //中间垂直拆分器（Qt::Vertical 垂直拆分，父对象是 spMainWindow，作为第二个子项）   
    QSplitter *spRight=new QSplitter(Qt::Vertical,spMainWindow);
    QTextEdit *txteditup=new QTextEdit("右边上部分窗口",spRight);  // 中间上侧文本框
    QTextEdit *txteditdown=new QTextEdit("右边下部分窗口",spRight);  // 中间下侧文本框

    // 3：
    //嵌套在 spRight 里的垂直拆分器,作为spRight的子项
    QSplitter *sptest=new QSplitter(Qt::Vertical,spRight);  
    QTextEdit *txtedittest=new QTextEdit("Qt",sptest);

    //4.
    //最右侧水平拆分器
    QSplitter *sptestend=new QSplitter(Qt::Horizontal,spMainWindow);
    QTextEdit *txtedittestend=new QTextEdit("Qt",sptestend);

    spMainWindow->setWindowTitle("Splitter类拆分窗口测试");
    spMainWindow->show();
}
MainWindow::~MainWindow()
{}
```

<img src="C:\Users\张超然\AppData\Roaming\Typora\typora-user-images\image-20260126220401999.png" alt="image-20260126220401999" style="zoom: 50%;" />





## 二、QDockWidget 类：可停靠的悬浮窗口

QDockWidget 是独立的附属窗口控件，核心作用是实现 “**主窗口 + 灵活调整的辅助窗口**” 结构，必须依附 QMainWindow 使用。

### 1. 核心基础信息

- **核心功能**：作为独立窗口可停靠在 QMainWindow 指定区域（上、下、左、右），也可脱离主窗口悬浮显示；
- **依赖配置**：需包含头文件 `#include <QDockWidget>`，依赖 `QT += widgets` 模块；
- **使用前提**：仅支持 QMainWindow 作为父容器，无法用于 QWidget 或 QDialog。

### 2. 关键特性

- 可停靠性：支持停靠在 QMainWindow 四个边缘区域，支持切换停靠位置；
- 独立交互：自带标题栏，支持拖动、关闭操作，可通过 API 限制功能（如禁止关闭、禁止悬浮）；
- 多窗口管理：QMainWindow 可同时管理多个 QDockWidget，支持调整停靠顺序。

### 3. 适用场景

- 软件附属功能面板（如绘图工具：左侧颜色选择器、右侧属性设置面板）；
- 可隐藏 / 显示的辅助内容（如浏览器：右侧书签栏、底部下载栏）；
- 主窗口内容复杂，需拆分独立模块的场景（如 IDE：右侧调试面板、底部终端面板）。

### 代码案例

**代码`mainwindow.cpp`**

```cpp
#include "mainwindow.h"

#include <QTextEdit>
#include <QDockWidget>

MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{

    DockWidgetFunc();

}

MainWindow::~MainWindow()
{
}


void MainWindow::DockWidgetFunc()
{
    setWindowTitle("QDockWidget类停靠窗口测试.");

    QTextEdit *tedit=new QTextEdit(this); // 定义QTextEdit对象作为主窗口
    tedit->setText("国防科技大学，中南大学，湖南大学，湖南师范大学");
    tedit->setAlignment(Qt::AlignCenter);
    setCentralWidget(tedit); //将编辑框控件设置为主窗口的中央窗体

    // 创建停靠窗口1
    QDockWidget *dw1=new QDockWidget("停靠窗口（一）",this);
    // 设置停靠窗口特性：仅可移动（不可关闭、不可浮动）
    dw1->setFeatures(QDockWidget::DockWidgetMovable); // 可移动特性
    // 设置允许停靠的区域：仅左侧/右侧（不能拖到上下侧）
    dw1->setAllowedAreas(Qt::LeftDockWidgetArea|Qt::RightDockWidgetArea);

    QTextEdit *qtedit1=new QTextEdit();
    qtedit1->setText("浙江大学（Zhejiang University），简称“浙大”，位于浙江省杭州市，是中华人民共和国教育部直属的综合性全国重点大学，位列国家“双一流”、 [106]  “211工程”、“985工程”，是九校联盟（C9） [113]  、中国大学校长联谊会、环太平洋大学联盟、世界大学联盟、全球大学校长论坛、全球高校人工智能学术联盟、国际应用科技开发协作网、新工科教育国际联盟、全球能源互联网大学联盟、CDIO工程教育联盟、医学“双一流”建设联盟成员，入选“珠峰计划”、“强基计划”、“2011计划”、“111计划”、卓越工程师教育培养计划、卓越医生教育培养计划、卓越法律人才教育培养计划、卓越农林人才教育培养计划、全国首批深化创新创业教育改革示范高校、学位授权自主审核单位。曾培养出厉绥之、束星北、李政道等杰出校友。");
    dw1->setWidget(qtedit1);  // 停靠窗口必须通过 setWidget() 设置内部内容
    addDockWidget(Qt::RightDockWidgetArea,dw1);


    // 创建停靠窗口2
    QDockWidget *dw2=new QDockWidget("停靠窗口（二）",this);
    // 设置特性：可关闭（Closable）+ 可浮动（Floatable）
    dw2->setFeatures(QDockWidget::DockWidgetClosable|QDockWidget::DockWidgetFloatable); // 关闭 浮动

    QTextEdit *qtedit2=new QTextEdit();
    qtedit2->setText("复旦大学，简称“复旦”，位于直辖市上海，是中华人民共和国教育部直属的全国重点大学，中央直管高校， [145]  由教育部与上海市重点共建， [148]  位列国家“双一流”、“985工程”、“211工程”建设高校， [150]  入选珠峰计划、强基计划、111计划、2011计划、卓越医生教育培养计划、卓越法律人才教育培养计划、国家建设高水平大学公派研究生项目、新工科研究与实践项目、中国政府奖学金来华留学生接收院校、深化创新创业教育改革示范高校、首批学位授权自主审核单位，是环太平洋大学联盟、九校联盟、全球大学高研院联盟、中国大学校长联谊会、东亚研究型大学协会、新工科教育国际联盟、医学“双一流”建设联盟、长三角研究型大学联盟、 长三角高校智库联盟、中俄综合性大学联盟成员，是一所综合性研究型大学。");
    dw2->setWidget(qtedit2);
    // 将 dw2 添加到主窗口右侧（和 dw1 并排/堆叠）
    addDockWidget(Qt::RightDockWidgetArea,dw2);

}
```

<img src="C:\Users\张超然\AppData\Roaming\Typora\typora-user-images\image-20260127010140658.png" alt="image-20260127010140658" style="zoom:50%;" />



## 三、两类控件核心区别

|   特性   |            QSplitter 类            |         QDockWidget 类          |
| :------: | :--------------------------------: | :-----------------------------: |
| 核心作用 | 分割界面为多区域，支持拖动调整大小 | 提供可停靠 / 悬浮的独立附属窗口 |
| 依附对象 |   可独立使用，或依附任意容器控件   |      必须依附 QMainWindow       |
| 交互方式 |      拖动分割线调整子区域大小      | 拖动窗口实现停靠 / 悬浮，可关闭 |
| 界面结构 |      子区域是整体布局的一部分      |     独立窗口，可脱离主窗口      |
| 适用场景 |         多区域内容联动展示         |       主窗口附属功能模块        |

## 四、核心使用总结

1. **QSplitter**：聚焦 “分割 + 可调整大小”，适合多区域内容需要灵活分配空间的场景，嵌套使用可实现复杂分割结构；
2. **QDockWidget**：聚焦 “停靠 + 独立窗口”，必须结合 QMainWindow，适合拆分主窗口附属功能，支持用户按需显示 / 隐藏；
3. **搭配使用**：QMainWindow 中心区域可用 QSplitter 分割，周围停靠 QDockWidget，实现 “主区域灵活分割 + 附属窗口可停靠” 的专业级界面。