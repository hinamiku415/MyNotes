## Qt QFontDialog 字体对话框实战：从基础到应用全解析

在 Qt 框架的界面开发中，字体选择是许多应用（如文本编辑器、报表工具、个性化界面）的核心功能之一。Qt 提供的`QFontDialog`组件，让开发者无需手动搭建复杂界面，就能快速实现跨平台的字体选择交互。

## 一、QFontDialog 基础

`QFontDialog`是 Qt Widgets 模块下的标准对话框类，继承自`QDialog`，专门用于提供**可视化的字体选择界面**。它的核心价值在于：

- **跨平台一致性**：在 Windows、macOS、Linux 等系统上自动适配原生风格，无需额外适配代码；

- **易用性**：通过静态方法直接调用，无需手动创建对话框实例，几行代码即可集成；

- **完整功能**：支持字体族（如 “方正舒体”“等线”）、字号（如 20、26）、样式（粗体、斜体）、效果（下划线、删除线）的选择，还提供实时预览功能；

- **灵活交互**：可通过返回值判断用户操作（确认 / 取消），并轻松将选择的字体应用到文本控件（如`QLineEdit`、`QLabel`）。

### 核心依赖与头文件

使用`QFontDialog`需满足以下依赖，这是后续代码运行的基础：

- 头文件：`#include <QFontDialog>`

- 项目配置（.pro 文件）：`QT += widgets`（因`QFontDialog`属于 Widgets 模块）

## 二、实战案例

下面将以 “点击按钮调用字体对话框，将选择的字体应用到输入框” 为例，完整解析代码实现逻辑。案例包含 3 个核心文件（`dialog.h`、`dialog.cpp`、`main.cpp`），覆盖类定义、界面布局、信号与槽绑定、字体应用全流程。

### 头文件（dialog.h）：类结构定义

首先定义对话框类`Dialog`，包含界面控件（按钮、输入框）、布局管理器和槽函数的声明：

```cpp
#ifndef DIALOG_H
#define DIALOG_H
#include <QDialog>
#include <QPushButton>   // 按钮控件
#include <QLineEdit>     // 单行输入框
#include <QFontDialog>   // 字体对话框
#include <QGridLayout>   // 网格布局

class Dialog : public QDialog
{
    Q_OBJECT  // 必须添加，支持信号与槽机制

public:
    Dialog(QWidget *parent = nullptr);
    ~Dialog();

private:
    // 布局管理器：控制控件位置和大小
    QGridLayout *glayout;
    // 功能控件：触发字体选择的按钮、显示字体效果的输入框
    QPushButton *fontbutton;
    QLineEdit *fontlineedit;

private slots:
    // 槽函数：点击按钮后调用，处理字体选择逻辑
    void dispFontFunc();
};

#endif // DIALOG_H
```

### 源文件（dialog.cpp）：核心逻辑实现

这是案例的核心，包含界面初始化、信号与槽绑定、字体对话框调用和字体应用逻辑：

```cpp
#include "dialog.h"

// 构造函数：初始化界面和逻辑
Dialog::Dialog(QWidget *parent)
    : QDialog(parent)
{
    // 1. 设置对话框标题
    setWindowTitle("字体对话框测试");

    // 2. 初始化布局管理器（网格布局）
    glayout = new QGridLayout(this);  // this表示布局作用于当前对话框

    // 3. 初始化控件
    fontbutton = new QPushButton("调用字体对话框");  // 按钮文本
    fontlineedit = new QLineEdit;                    // 输入框
    fontlineedit->setText("111");               // 输入框默认文本（用于预览字体）

    // 4. 将控件添加到网格布局：参数（控件，行号，列号）
    glayout->addWidget(fontbutton, 0, 0);  // 按钮放在第0行第0列
    glayout->addWidget(fontlineedit, 0, 1); // 输入框放在第0行第1列

    // 5. 信号与槽绑定：按钮点击（信号）→ 调用dispFontFunc（槽函数）
    connect(fontbutton, SIGNAL(clicked()), this, SLOT(dispFontFunc()));
}

Dialog::~Dialog()
{}

// 槽函数：处理字体选择逻辑
void Dialog::dispFontFunc()
{
    // 1. 调用QFontDialog静态方法getFont，显示字体对话框
    // 参数1：&isbool - 接收用户操作结果（true=确认，false=取消）
    // 参数2：默认字体（此处未指定，使用系统默认）
    // 参数3：父窗口（此处为this，确保对话框居中显示在父窗口上）
    bool isbool;
    QFont font = QFontDialog::getFont(&isbool, this);

    // 2. 判断用户是否点击“确认”按钮
    if (isbool)
    {
        // 3. 将选择的字体应用到输入框
        fontlineedit->setFont(font);
    }
}
```

## 三、功能扩展：让 QFontDialog 更灵活

基础案例实现了核心功能，但实际开发中可能需要更多定制化需求。结合 Qt 官方文档和实践经验，以下是 3 个常用的扩展方向：

### 1. 设置默认字体

若希望字体对话框打开时，默认选中特定字体（而非系统默认），可重载`getFont`方法的第二个参数（`const QFont &initial`）：

```cpp
// 示例：默认字体为“隶书”、24号、粗体
QFont defaultFont("隶书", 24, QFont::Bold);
bool isbool;
// 传入默认字体，对话框打开时会优先显示该字体
QFont font = QFontDialog::getFont(&isbool, defaultFont, this);
```

### 2. 过滤可选字体

若需限制用户只能选择特定类型的字体（如仅中文字体、仅等宽字体），可通过`QFontDialog`的`setFontFilters`方法设置过滤规则：

```cpp
// 1. 创建QFontDialog实例（而非直接调用静态方法）
QFontDialog dialog(this);
// 2. 设置过滤规则：仅显示中文字体
dialog.setFontFilters(QFontDialog::FontFilter::ChineseSimplifiedFonts);
// 3. 显示对话框并判断用户操作
if (dialog.exec() == QDialog::Accepted)
{
    // 4. 获取选择的字体并应用
    QFont font = dialog.selectedFont();
    fontlineedit->setFont(font);
}
```

常用过滤规则包括：

- `QFontDialog::AllFonts`：显示所有字体（默认）；

- `QFontDialog::NonScalableFonts`：仅显示不可缩放字体；

- `QFontDialog::MonospacedFonts`：仅显示等宽字体（如 Consolas、Courier）；

- `QFontDialog::ChineseSimplifiedFonts`：仅显示简体中文字体。

### 3. 隐藏对话框按钮

若需自定义对话框的确认 / 取消逻辑（如通过菜单触发，而非对话框自带按钮），可隐藏默认的 “OK” 和 “Cancel” 按钮：

```cpp
QFontDialog dialog(this);
// 设置选项：隐藏按钮
dialog.setOption(QFontDialog::Option::NoButtons, true);
// 自定义确认逻辑（示例：5秒后自动确认当前选择）
QTimer::singleShot(5000, [&]() {
    dialog.accept();  // 触发“确认”操作
});
// 显示对话框
if (dialog.exec() == QDialog::Accepted)
{
    QFont font = dialog.selectedFont();
    fontlineedit->setFont(font);
}
```

## 总结：QFontDialog 的核心优势与注意事项

### 核心优势

1. **低开发成本**：静态方法`getFont`可快速集成，无需手动构建界面；
2. **跨平台兼容**：自动适配不同系统的原生风格，提升用户体验；
3. **功能完整**：覆盖字体选择的所有核心需求，支持扩展；

### 注意事项

1. **模态对话框特性**：`QFontDialog`默认是模态对话框（打开时阻塞其他操作），若需非模态，需手动创建实例并调用`show()`（而非`exec()`）；
2. **字体可用性**：不同系统安装的字体可能不同（如 Windows 的 “微软雅黑” 在 Linux 上可能没有），建议提供默认字体 fallback 方案；
