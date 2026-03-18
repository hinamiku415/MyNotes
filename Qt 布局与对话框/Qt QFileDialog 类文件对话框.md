## Qt QFileDialog 类文件对话框实战笔记

在 Qt 开发中，**文件对话框是实现文件选择功能的常用组件。QFileDialog 类可以让用户便捷地遍历系统文件，选择单个或多个文件、目录**。这篇笔记就来梳理 QFileDialog 类的基础使用要点。

## 一、QFileDialog 类基础信息

1. **类的作用**

   QFileDialog 类提供文件对话框组件，支持用户选择文件或者目录。

2. **头文件与模块**

   - 头文件：`#include <QFileDialog>`
   - qmake 配置：`QT += widgets`

3. **类的继承关系**

   该类继承自 QDialog，与 QColorDialog、QFontDialog、QInputDialog 等同属 Qt 的对话框组件类。<img src="D:\DownLoad\去除图片水印.png" alt="去除图片水印" style="zoom: 25%;" />

## 二、实例代码

以选择文件为例，程序运行后可实现以下功能：

定义用于存储文件名的变量：`QString fileName`

1. 调用 QFileDialog 的相关静态方法打开对话框，获取用户选择的文件路径
2. 通过文件路径读取文件信息，计算并展示文件大小

#### 头文件：

```cpp
#ifndef QFILEDIALOGTEST_H
#define QFILEDIALOGTEST_H

#include <QDialog>

#include <QLabel>
#include <QLineEdit>
#include <QPushButton>
#include <QHBoxLayout> // 水平布局
#include <QVBoxLayout> // 垂直布局

#include <QFileDialog>

class QFileDialogTest : public QDialog
{
    Q_OBJECT

public:
    QFileDialogTest(QWidget *parent = nullptr);
    ~QFileDialogTest();

private:
    QLabel *FileNameLabel;
    QLineEdit *FileNameLineEdit;

    QPushButton *FileButton;

    QLabel *FileSizeLabel;
    QLineEdit *FileSizeLabelLineEdit;

    QPushButton *GetFileInfoButton;

private slots:
    void GetFileInfoFunc(); // 用于打开文件
    void GetFileSizeFunc(); // 用于获取文件大小

};
#endif // QFILEDIALOGTEST_H

```

#### 代码部分：

```cpp
#include "qfiledialogtest.h"

QFileDialogTest::QFileDialogTest(QWidget *parent)
    : QDialog(parent)
{
    // 1:创建控件
    FileNameLabel=new QLabel("文件名称：");
    FileNameLineEdit=new QLineEdit;
    FileButton=new QPushButton("选择...");

    FileSizeLabel=new QLabel("文件大小：");
    FileSizeLabelLineEdit=new QLineEdit;

    GetFileInfoButton=new QPushButton("获取文件大小信息");


    // 2:排列布局
    QGridLayout *glayout=new QGridLayout;
    glayout->addWidget(FileNameLabel,0,0);
    glayout->addWidget(FileNameLineEdit,0,1);
    glayout->addWidget(FileButton,0,2);

    glayout->addWidget(FileSizeLabel,1,0);
    glayout->addWidget(FileSizeLabelLineEdit,1,1,1,2);

    //创建水平布局：用于放置"获取文件大小信息"按钮
    QHBoxLayout *hlayout=new QHBoxLayout;
    hlayout->addWidget(GetFileInfoButton);


    //创建垂直布局（作为主布局），并设置给当前对话框
    QVBoxLayout *vlayout=new QVBoxLayout(this);
    vlayout->addLayout(glayout);
    vlayout->addLayout(hlayout);

    // 信号槽函数连接
    connect(FileButton,SIGNAL(clicked()),this,SLOT(GetFileInfoFunc()));
    connect(GetFileInfoButton,SIGNAL(clicked()),this,SLOT(GetFileSizeFunc()));
}

QFileDialogTest::~QFileDialogTest()
{}

//槽函数：打开文件对话框，获取选中文件的路径并显示
void QFileDialogTest::GetFileInfoFunc()
{
    // QFileDialog::getOpenFileName：打开"打开文件"对话框，返回选中的文件路径
    // 参数说明：
    // "打开" → 对话框标题
    // "/" → 默认打开的目录（根目录，可改为如"D:/"）
    // "Files(*)" → 文件过滤器（所有文件，可改为如"文本文件(*.txt);;图片文件(*.jpg *.png)"）
    QString strFileName = QFileDialog::getOpenFileName(this, "打开", "/", "Files(*)");
    
    // 将选中的文件路径设置到FileNameLineEdit中显示
    FileNameLineEdit->setText(strFileName);
}

// 槽函数：获取选中文件的大小并显示
void QFileDialogTest::GetFileSizeFunc()
{
    // 1. 从FileNameLineEdit中读取用户选中的文件路径
    QString strFileNames = FileNameLineEdit->text();
    
    // 2. 创建QFileInfo对象：用于获取文件的元信息（大小、创建时间、后缀名等）
    QFileInfo fileinfo(strFileNames);
    
    // 3. 获取文件大小：size()返回qint64类型（字节数）
    qint64 FileSize = fileinfo.size();
    
    // 4. 将文件大小（qint64）转换为QString，并设置到FileSizeLabelLineEdit中显示
    FileSizeLabelLineEdit->setText(QString::number(FileSize));
}
```

![image-20260128173446574](C:\Users\张超然\AppData\Roaming\Typora\typora-user-images\image-20260128173446574.png)